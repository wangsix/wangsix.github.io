# Personal website redesign — design spec

Date: 2026-09-19
Repo: wangsix.github.io (GitHub Pages, Jekyll)
Status: approved in chat, pending spec review

## Goal

Revive the personal site of Cheng-i Wang (汪正一, "Wang Six"): modern layout
and functionality, current content (AudioShake, publications through 2025,
patents), while keeping the existing artistic identity: the brushstroke
logo, the cream palette, and the black label-style section headings.

## Decisions (from discussion)

| Topic | Decision |
|---|---|
| Stack | Jekyll on GitHub Pages, default plugins only. Chosen because future interactive demos or self-contained web apps can be dropped in as a folder with its own `index.html` and are copied verbatim. |
| Keep | Logo, cream background `#f6f3e6`, warm grey text, black label headings. |
| Drop | Left sidebar layout, Skeleton CSS framework, nav-arrow icons. |
| Photos | Stage photo as a muted hero strip on Home. A thin, consistently filtered ribbon on other pages. The four project banners are kept for the future projects page. Photos are not deleted. |
| New photos | A portrait slot on Home that renders only if `img/portrait.jpg` exists. Owner adds it later. |
| Pages now | Home (bio, updates, links), Publications (papers by year, patents). |
| Pages later | Projects, Music/Performance: created as `published: false` drafts, absent from nav. Content decided later. |
| Language | EN / 繁體中文 toggle for bio, headings, and update text. Publications stay English. |
| Bio | Medium version from the speaker-materials doc, both languages. |
| Updates | Seeded from LinkedIn activity and CV; owner corrects dates. |
| Contact | LinkedIn and Google Scholar links only. No email on the site. |
| Fonts | Self-hosted Latin woff2: Inter (400/500/600) for text, Newsreader italic for accents. CJK falls back to system fonts (PingFang TC, Microsoft JhengHei, Noto Sans TC). No third-party font requests. |
| Dark mode | Follows `prefers-color-scheme`. Warm charcoal background, cream text, logo inverted. No manual toggle. |

## File layout

```
_config.yml                 title, description, url, lang, exclude list
_layouts/default.html       single layout: head, header (logo, nav, lang toggle), main, footer
_includes/ribbon.html       photo strip; takes image path + height variant
_data/bio.yml               name, name_zh, nickname, headline{en,zh}, paragraphs{en[],zh[]}
_data/updates.yml           list of {date, en, zh?, url?}
_data/publications.yml      list of {year, authors, title, venue, url?, note?}
_data/patents.yml           list of {title, number, status, date, inventors, url}
_data/nav.yml               list of {label_en, label_zh, url, hidden?}
index.html                  home
publications.html           publications + patents
projects.html               published: false
music.html                  published: false
css/site.css                all styles (replaces css/style.css and css/stylesheets/)
assets/fonts/*.woff2        Inter latin 400/500/600, Newsreader latin 400 italic
img/                        logo (PNG, plus SVG if tracing succeeds), favicons, photos
Gemfile                     github-pages gem, local preview only
README.md                   how to add a publication/update, how to add a demo folder
docs/superpowers/specs/     this spec (excluded from the build)
```

Removed: `css/style.css`, `css/stylesheets/{base,skeleton,layout}.css`,
`img/{x,x3,x4,xh,xh4,next,prev,back,back2,down,download,newwindow*,bg}.*`,
`img/newwindow.psd`, `images/` (duplicate favicons; the root `favicon.ico`
and `img/favsix*.png` stay).

## Layout and visual design

- Single column, max width 720px, 16px side gutters on phones.
- Header: logo (links home) left; nav links and the EN/中文 toggle right.
  On narrow screens the header wraps to two rows; no hamburger menu.
- Section headings keep the signature treatment: small uppercase white text
  on a black label (`.label`). The old beige Times-italic secondary text
  becomes `.muted-serif` using Newsreader italic in `#b0ab99`.
- Photo strips: `object-fit: cover`, `filter: grayscale(.6) sepia(.2)`, so
  every photo reads as one warm set. Hero on Home ~200px tall, ribbons
  ~120px. Ribbons are cropped thinner partly to hide that the source files
  are only 700px wide.
- Links: underline on hover only, accent colour `#F15A29` (the old hover
  orange) for hover state.
- Footer: LinkedIn and Google Scholar links, © line.
- Dark mode via CSS custom properties: background `#1d1b17`, text
  `#e8e2d2`, labels invert (cream on black stays; black label becomes
  cream label with dark text), photos get `brightness(.8)`, logo
  `filter: invert(1)`.

## Language toggle

- `<html lang="en" data-lang="en">`. A tiny inline script in `<head>` reads
  `localStorage.lang` and sets both attributes before first paint.
- Translatable elements carry `lang="en"` or `lang="zh-Hant"`. CSS hides
  the non-active one: `[data-lang=en] [lang=zh-Hant], [data-lang=zh] [lang=en] { display:none }`.
- The toggle button flips the attribute and stores the choice. Works
  without JS in the sense that English is always rendered.
- Publications page content is English-only; its heading and nav label
  are bilingual.

## Content

### Bio (`_data/bio.yml`)
Medium bio from the speaker doc, EN and 中文. Headline: "Deep Learning
Engineer, AudioShake". Name shown as "Cheng-i Wang" with "汪正一 (汪六)"
beside it in the 中文 view and as a small secondary line in EN.

### Updates (`_data/updates.yml`), seeded, dates to be corrected by owner
- 2026-06 Talk "What LLMs Left Untouched in Audio" at Taiwan Multimedia Tech #12, Taipei.
- 2026-04 AudioShake launches Dialogue RT (11 ms real-time dialogue isolation); worked on the low-latency separation. (approx.)
- 2026-01 Tech talk on AI source separation at KKCompany, Taipei. (approx.)
- 2023-03 Joined AudioShake as Deep Learning Engineer.
- 2022-05 "ToNet" at ICASSP 2022 (co-author).
- 2020-05 "Deep Autotuner" at ICASSP 2020; TISMIR survey on music structure analysis.
- 2019-10 Patent US 10,446,123 issued.
- 2018-04 Joined Smule as Audio Software Engineer; ICASSP 2018 singing-style paper.
- 2018-03 Ph.D. in Music (Computer Music), UC San Diego.

### Publications (`_data/publications.yml`)
All papers, theses, and reports from the Google Scholar profile,
deduplicated (patents excluded here; Scholar's duplicate "Deriving
Representative Structure" entry dropped in favour of the arXiv paper).
Links: URLs already on the current site are kept; arXiv entries link to
arxiv.org by ID; DOIs only where known. No guessed URLs.

### Patents (`_data/patents.yml`)
- US 10,446,123, "Intuitive music visualization using efficient structural segmentation", issued 2019-10-15, with Gautham Mysore.
- US 2022/0222294 A1, "Densification in Music Search and Recommendation", filed 2021-02-22, with S. Sullivan, D. A. Steinwedel, G. Tzanetakis.

## Demos convention
A folder such as `demos/<name>/` containing its own `index.html` and assets
is published at `/demos/<name>/` untouched. Files with YAML front matter
are processed by Jekyll; files without it are copied as-is. Documented in
README.

## Out of scope
- Projects and Music page content and design.
- Manual dark-mode toggle.
- Analytics, comments, RSS.
- Retina re-shoots of the existing photos.

## Verification
- `jekyll build` locally (Docker `jekyll/jekyll` image, or a local Ruby
  with the github-pages gem) must succeed with no warnings about the data files.
- Built-in browser checks: Home and Publications at desktop and 375px
  width; light and dark; EN and 中文; nav to unpublished pages 404s; hero
  and ribbon render; portrait slot absent until `img/portrait.jpg` exists.
- HTML validity spot-check of the generated pages.
