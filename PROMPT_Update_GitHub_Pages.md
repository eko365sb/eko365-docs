# Prompt — Aggiornamento repo `eko365-docs` su GitHub Pages

> Copia e incolla questo prompt in una sessione Claude Code con accesso a `gh` CLI autenticato.

---

## Contesto

Il repo `eko365sb/eko365-docs` è già pubblicato su GitHub Pages.
I file sorgente aggiornati si trovano nella cartella locale `eko365-docs/` del workspace.

L'URL pubblico è: `https://eko365sb.github.io/eko365-docs/`

## Task

### 1. Clona il repo (se non già presente)

```bash
# Se la cartella del repo locale non esiste, clona:
gh repo clone eko365sb/eko365-docs /tmp/eko365-docs-repo

# Se esiste già, pull per allinearsi:
cd /tmp/eko365-docs-repo && git pull origin main
```

### 2. Sincronizza i file aggiornati

Copia i file dalla cartella sorgente `eko365-docs/` del workspace al repo clonato, sovrascrivendo i file esistenti. I file da sincronizzare sono:

- `index.html` (hub landing page)
- `css/eko365.css` (design system)
- `reporting-platform/index.html` (pagina SmartREPORTING — il file aggiornato)
- `_templates/page-template.html` (template)
- `README.md`

**NON** copiare i file `PROMPT_*.md` nel repo — sono istruzioni operative che restano solo nel workspace.

Comando suggerito:

```bash
SRC="<percorso-workspace>/eko365-docs"
DEST="/tmp/eko365-docs-repo"

cp "$SRC/index.html" "$DEST/"
cp "$SRC/css/eko365.css" "$DEST/css/"
cp "$SRC/reporting-platform/index.html" "$DEST/reporting-platform/"
cp "$SRC/_templates/page-template.html" "$DEST/_templates/"
cp "$SRC/README.md" "$DEST/"
```

### 3. Verifica le differenze

```bash
cd /tmp/eko365-docs-repo
git diff --stat
```

Mostra un riepilogo dei file modificati prima di procedere. Se non ci sono differenze, il repo è già aggiornato — interrompi qui.

### 4. Commit e push

```bash
git add -A
git commit -m "Update SmartREPORTING architecture — audit corrections (scheduling, S3 tree, quarter-end detection)"
git push origin main
```

**Nota:** adatta il messaggio di commit in base alle modifiche effettive. Mantieni il formato conciso e descrittivo.

### 5. Verifica pubblicazione

GitHub Pages ripubblica automaticamente in ~30 secondi dopo il push. Verifica:

```bash
# Controlla stato deployment
gh api repos/eko365sb/eko365-docs/pages --jq '.status'

# Deve restituire "built"
# Se restituisce "building", attendi 30 secondi e riprova
```

### 6. Output finale

Stampa:
- Lista file aggiornati (da `git diff --stat` del punto 3)
- URL per verifica: `https://eko365sb.github.io/eko365-docs/reporting-platform/`
- Conferma che il deployment è completato

---

## Note

- Ogni push su `main` triggera automaticamente il rebuild di GitHub Pages (~30s)
- Non serve toccare le impostazioni Pages — sono già configurate dal setup iniziale
- SharePoint non richiede aggiornamenti: il web part Embed punta all'URL GitHub Pages che non cambia
- Per aggiornamenti futuri, riutilizza questo stesso prompt modificando solo il messaggio di commit al punto 4
