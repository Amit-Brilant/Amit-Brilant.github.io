# Amit Brilant portfolio

Personal site built with [Hugo Blox](https://hugoblox.com) (the open source
`academic-cv` starter), themed blue. Live at https://amit-brilant.github.io.

## Running it locally

```bash
npm install
npm run dev
```

Then open http://localhost:1313. The site rebuilds as you edit.
First time on a new machine: `brew install hugo go`.

## Where the content lives

| What | File |
|---|---|
| Name, role, bio, links, short education, photo | `data/authors/me.yaml`, `assets/media/authors/me.png` |
| Experience and full education | `data/authors/cv.yaml` |
| Section order, project cards, skills | `content/_index.md` |
| Project pages | `content/projects/*/index.md` |
| Home page styles | `layouts/_partials/hooks/head-end/home-sections.html` |
| Site name, colours, nav | `config/_default/params.yaml`, `menus.yaml` |
| CV download | `static/uploads/Amit-Brilant-CV.pdf` |

## Deploying

Every push to `main` runs `.github/workflows/pages.yml`, which builds the site
with Hugo and publishes it to GitHub Pages.
