# Quarterly Report Generation — Cross-Pipeline Reference

How each EKO365 reporting pipeline handles automatic quarterly report generation.

## Schedule Overview

| Pipeline | Repo | Cron | Trigger Day | Quarterly Months |
|----------|------|------|-------------|-----------------|
| SmartBackup (AFI.ai) | smartbackup-reports | `0 6 2 * *` | 2nd | 03, 06, 09, 12 |
| Veeam Backup | smartbackup-reports | `0 10 * * *` | 2nd (auto-detect) | 03, 06, 09, 12 |
| SmartPATCH | smartpatch-reports | `0 7 * * *` | 1st (auto-detect) | 03, 06, 09, 12 |
| M365 Graph | m365-graph-pipeline | `0 14 3 * *` | 3rd | 03, 06, 09, 12 |
| Desk365 | desk365-reports | `0 6 3 * *` | 3rd | 03, 06, 09, 12 |

## Quarter-End Detection Patterns

All pipelines detect quarter-end by checking if the processed month (previous month)
ends in 03, 06, 09, or 12. The **period expansion** mechanism differs:

### Pattern A: Explicit Date-Range Expansion

**Used by:** SmartBackup, Veeam, Desk365

The prepare step expands the `PERIOD` parameter from a single month to the full quarter.

```
Input:  PERIOD = "2026-06:2026-06"  (auto-calculated previous month)
Detect: END_MM = "06" -> quarter-end
Output: PERIOD = "2026-04:2026-06"  (expanded to full Q2)
```

Analysis and report scripts receive the expanded date range directly.
Manual `period` input is never overridden.

| Pipeline | Period Format | Expansion Logic |
|----------|--------------|-----------------|
| SmartBackup | `YYYY-MM:YYYY-MM` | `case` on MONTH: 03->01, 06->04, 09->07, 12->10 |
| Veeam | `YYYY-MM:YYYY-MM` | Same `case` pattern (added 2026-03-23) |
| Desk365 | `--from YYYY-MM-DD --to YYYY-MM-DD` | FROM expanded to quarter start date |

### Pattern B: Implicit Expansion via S3 Data Loading

**Used by:** SmartPATCH

No period parameter is expanded. Instead, the report generation step downloads
**all** historical KPI files from S3 for each organization. Trend depth depends
on how much data exists on S3.

```
1st of July: scope="all" (quarter-end detected)
Report script: downloads s3://{ORG}/kpi/action1/*.json (all dates)
Trend chart: uses all available data points (e.g. Jan-Jun if daily collection active)
```

### Pattern C: Quarter Label

**Used by:** M365 Graph

The prepare step sets `period_client` to a human-readable label (e.g. "Q1 2026")
instead of a date range. Report scripts parse this label to determine the data window.
Security and audit modules use a 90-day API lookback, so a single collection in March
already contains full Q1 data.

```
3rd of April: MONTH = "2026-03", MM = "03" -> quarter-end
period_ops   = "2026-03"     (for ops report, always monthly)
period_client = "Q1 2026"    (for client DOCX + PPTX)
report_type  = "all"         (ops + client)
```

## Data Dependency for Quarterly Reports

For a complete quarterly report, each pipeline needs data from all 3 months.
Here is how data accumulates:

| Pipeline | Data Source | Monthly Accumulation | Q Report Reads From |
|----------|-----------|---------------------|-------------------|
| SmartBackup | AFI.ai API (35-day window) | Fetch on 2nd -> S3 snapshot | S3: all 3 monthly snapshots |
| Veeam | Graph API (email inbox) | Daily fetch -> S3 daily files | S3: all daily files in range |
| SmartPATCH | Action1 API (real-time) | Daily collection -> S3 raw+KPI | S3: all KPI files (any date) |
| M365 Graph | Graph API (90-day lookback) | Weekly/monthly -> S3 per-module | S3: latest collection (has 90d history) |
| Desk365 | Desk365 API (full history) | Fresh extraction on report day | API: full quarter in one call |

## Key Differences and Gotchas

1. **SmartBackup January gap**: The AFI.ai API has a 35-day rolling window for
   `task_statistics`. If the monthly fetch on the 2nd fails or is skipped,
   that month's task performance data is lost. Other endpoints (resources,
   storage, policies) remain available indefinitely.

2. **Veeam daily gaps**: If the daily cron fails for multiple days, those emails
   may still be in the mailbox but won't be fetched until the next successful run
   (which fetches last 3 days only). Backfill requires `--since` parameter.

3. **SmartPATCH trend depth**: Since there is no explicit period parameter,
   reports always show ALL available history. A new organization added mid-quarter
   will show a shorter trend than established ones.

4. **M365 Graph label parsing**: Report scripts must understand "Q1 2026" format.
   If a manual `period` input is provided, it overrides both `period_ops` and
   `period_client` with the same value.

5. **Desk365 full extraction**: Unlike other pipelines, Desk365 re-extracts ALL
   tickets in the quarter range from the API on report day. No dependency on
   previously stored data — always fresh.

## Manual Dispatch for Quarterly Reports

If a quarterly report was missed or needs regeneration:

```bash
# SmartBackup (requires snapshots on S3 for all 3 months)
gh workflow run smartbackup-pipeline.yml -f period="2026-01:2026-03" -f phase="report"

# Veeam (requires daily files on S3 for the full quarter)
gh workflow run veeam-pipeline.yml -f period="2026-01:2026-03" -f phase="all"

# SmartPATCH (uses all KPI data on S3 regardless of period)
gh workflow run smartpatch-on-demand.yml -f report_scope="quarterly"

# M365 Graph (uses collected data on S3)
gh workflow run graph-report.yml -f report_type="all" -f period="Q1 2026"

# Desk365 (re-extracts from API)
gh workflow run desk365-pipeline.yml -f from_date="2026-01-01" -f to_date="2026-03-31"
```

## Dry-Run (No Email)

All workflows accept a `send_email` boolean input (default: `true`).
To run a full pipeline without sending email to recipients:

```bash
# Any pipeline — add -f send_email=false
gh workflow run smartbackup-pipeline.yml -f phase="all" -f send_email=false
gh workflow run veeam-pipeline.yml -f phase="all" -f send_email=false
gh workflow run smartpatch-scheduled.yml -f phase="monthly" -f send_email=false
gh workflow run graph-report.yml -f report_type="all" -f send_email=false
gh workflow run desk365-pipeline.yml -f phase="all" -f send_email=false
```

This is useful for:
- Testing workflows after code changes
- Regenerating reports on S3 without notifying recipients
- QA validation before a scheduled run
