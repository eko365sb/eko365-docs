# Prompt — Setup repo `eko365-docs` su GitHub Pages

> Copia e incolla questo prompt in una sessione Claude Code con accesso a `gh` CLI autenticato.

---

## Contesto

Nella cartella `eko365-docs/` del workspace è presente un sito statico HTML pronto per essere pubblicato su GitHub Pages. La struttura è:

```
eko365-docs/
├── index.html                        # Hub di navigazione (landing page)
├── css/eko365.css                    # Design system condiviso EKO365
├── reporting-platform/index.html     # SmartREPORTING Architecture (prima pagina)
├── _templates/page-template.html     # Template per nuove pagine
└── README.md                         # Istruzioni operative
```

Il sito deve essere pubblicato come repo **public** nell'org `eko365sb` su GitHub, con GitHub Pages abilitato.

## Task

Esegui le seguenti operazioni in sequenza:

### 1. Crea il repo GitHub

```bash
gh repo create eko365sb/eko365-docs --public --description "EKO365 Internal Documentation Hub — GitHub Pages" --clone
```

Se il repo esiste già, clonalo:

```bash
gh repo clone eko365sb/eko365-docs
```

### 2. Copia i file

Copia tutto il contenuto della cartella `eko365-docs/` nel repo appena clonato, mantenendo la struttura delle cartelle. Assicurati che siano presenti:

- `index.html` (root)
- `css/eko365.css`
- `reporting-platform/index.html`
- `_templates/page-template.html`
- `README.md`

### 3. Aggiungi .gitignore

Crea un `.gitignore` con:

```
.DS_Store
*.swp
*~
.env
```

### 4. Aggiungi .nojekyll

Crea un file vuoto `.nojekyll` nella root del repo. Questo è essenziale: dice a GitHub Pages di servire i file HTML statici direttamente senza processarli con Jekyll (che ignorerebbe le cartelle che iniziano con `_`, come `_templates/`).

### 5. Commit e push

```bash
git add -A
git commit -m "Initial commit: EKO365 Internal Docs Hub with SmartREPORTING Architecture"
git push -u origin main
```

### 6. Abilita GitHub Pages

```bash
gh api repos/eko365sb/eko365-docs/pages -X POST -f source.branch=main -f source.path="/" --silent 2>/dev/null || echo "Pages already enabled or requires manual setup"
```

Se il comando API fallisce (succede su alcune org), abilita manualmente:
- Repository Settings → Pages → Source: Deploy from branch → Branch: `main`, folder: `/ (root)` → Save

### 7. Verifica

Attendi 1-2 minuti, poi verifica che il sito sia raggiungibile:

```bash
gh api repos/eko365sb/eko365-docs/pages --jq '.html_url'
```

L'URL sarà: `https://eko365sb.github.io/eko365-docs/`

Verifica anche la pagina specifica:
- Hub: `https://eko365sb.github.io/eko365-docs/`
- SmartREPORTING: `https://eko365sb.github.io/eko365-docs/reporting-platform/`

### 8. Output finale

Stampa un riepilogo con:
- URL del repo GitHub
- URL del sito GitHub Pages
- URL diretto della pagina SmartREPORTING (da usare nell'Embed web part di SharePoint)
- Conferma che `.nojekyll` è presente
- Conferma che GitHub Pages è attivo

---

## Note

- **Non servono dipendenze**: il sito è puro HTML/CSS statico, nessun build step
- **Aggiornamenti futuri**: basta push su `main`, GitHub Pages ri-pubblica automaticamente in ~30 secondi
- **Nuove pagine**: creare cartella, copiare `_templates/page-template.html`, aggiungere card in `index.html`
- **SharePoint**: nell'intranet, aggiungere un web part "Embed" con l'URL GitHub Pages della pagina specifica
