# wangsix.github.io

Personal site of Cheng-i Wang. Jekyll on GitHub Pages, no plugins.

## Editing content

Everything is in `_data/`:

- `updates.yml` — home page news. Newest first. Fields: `date` (`YYYY-MM`), `en`, `zh`, optional `url`. Quote any text containing `: ` or ` #`.
- `publications.yml` — papers. Fields: `year` (integer), `authors`, `title`, `venue`, optional `url`, optional `note`. Grouped by year automatically, newest year first; within a year, file order.
- `patents.yml` — `title`, `number`, `status`, `date`, `inventors`, `url`.
- `bio.yml` — name, headline and bio paragraphs in `en` and `zh`.
- `nav.yml` — nav links; set `hidden: true` to keep a page out of the nav.

Pages `projects.html` and `music.html` exist but have `published: false`. Remove that line and the `hidden: true` in `nav.yml` to launch them.

## Portrait

Drop a square image at `img/portrait.jpg` (about 800×800). The home page shows it automatically.

## Language

Elements with `lang="en"` or `lang="zh-Hant"` are shown only in that language. The toggle stores the choice in `localStorage`.

## Adding a demo or web app

Create a folder such as `demos/my-demo/` with its own `index.html` and assets. Files without YAML front matter are copied verbatim, so any self-contained static app works at `/demos/my-demo/`. Do not add front matter unless you want the site layout.

## Local preview

With a modern Ruby: `bundle install && bundle exec jekyll serve`, then open http://localhost:4000.
