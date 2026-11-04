# Editorial Carousel Storyboard

A single-file web app for designing Instagram portrait carousels (4:5), with PNG export and JSON project backups.

**Live site:** [carousel.kingkind.co.uk](https://carousel.kingkind.co.uk)

## Run it locally

Open [`carousel-editor/carousel.html`](carousel-editor/carousel.html) in a modern browser. There is no build step or server. CDN assets (Tailwind, Lucide, Google Fonts) need a network connection the first time the page loads.

## Repository layout

| Path | Purpose |
| --- | --- |
| `carousel-editor/carousel.html` | The editor (all markup, styles, and script) |
| `carousel-editor/README.md` | Editor documentation: slide data, functions, and behaviour |
| `index.html`, `carousel-editor/index.html` | Redirects to the editor |
| `CNAME` | Custom domain for GitHub Pages |
| `.github/workflows/pages.yml` | Deploys the site to GitHub Pages on push to `main` |

For how the editor works, see the [editor README](carousel-editor/README.md).
