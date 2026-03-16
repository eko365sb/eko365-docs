# EKO365 Internal Docs

Documentazione tecnica interna pubblicata via GitHub Pages e integrata nella intranet SharePoint.

## Struttura

```
eko365-docs/
├── index.html                        # Hub di navigazione (landing page)
├── css/
│   └── eko365.css                    # Design system condiviso (colori, font, componenti)
├── reporting-platform/
│   └── index.html                    # SmartREPORTING Architecture
├── _templates/
│   └── page-template.html            # Template per nuove pagine
└── README.md
```

## Aggiungere una nuova pagina

1. Creare una cartella con nome kebab-case: `mkdir nuova-pagina`
2. Copiare il template: `cp _templates/page-template.html nuova-pagina/index.html`
3. Modificare titolo, contenuto e footer
4. Aggiungere la card nel hub (`index.html` root) copiando il blocco HTML commentato
5. Push su `main` — GitHub Pages pubblica automaticamente

## Pubblicazione

- **GitHub Pages**: Settings → Pages → Source: `main` branch, root `/`
- **URL**: `https://eko365sb.github.io/eko365-docs/`
- **SharePoint**: Embed web part con URL GitHub Pages

## Design System (css/eko365.css)

Colori corporate, tipografia Century Gothic, componenti riutilizzabili:
flow diagram, tech table, KPI cards, module grid, schedule rows, S3 tree, tenant chips, output tags, tech badges.

## Convenzioni

- Stili inline solo per override puntuali; tutto il resto va in `eko365.css`
- Ogni pagina linka `../css/eko365.css` (path relativo)
- Il link "Hub Documentazione" in header riporta sempre alla root
- Footer con "Riservato, uso interno" su tutte le pagine
