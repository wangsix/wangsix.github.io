# Website Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild wangsix.github.io as a modern, bilingual (EN / 繁體中文), dark-mode-aware Jekyll site that keeps the logo, cream palette and black-label headings, with content driven by YAML data files.

**Architecture:** One Jekyll layout, one stylesheet, content in `_data/*.yml`. Language switching is a `data-lang` attribute on `<html>` toggled by ~10 lines of JS and enforced by CSS `display:none` on the inactive `lang` elements. Dark mode is CSS custom properties under `prefers-color-scheme`. No plugins beyond GitHub Pages defaults.

**Tech Stack:** Jekyll (GitHub Pages), Liquid, vanilla CSS with custom properties, self-hosted woff2 fonts (Inter, Newsreader), potracer (Python) once, offline, to trace the logo to SVG.

**Spec:** `docs/superpowers/specs/2026-09-19-website-redesign-design.md`

## Global Constraints

- GitHub Pages default Jekyll; no plugins beyond the defaults. `_config.yml` must not list plugins.
- Palette: light background `#f6f3e6`; dark background `#1d1b17`, dark text `#e8e2d2`; hover accent `#F15A29`; muted serif `#b0ab99`.
- Fonts: Inter 400/500/600 and Newsreader 400 italic, self-hosted from `assets/fonts/`, no third-party font requests. CJK on system fonts.
- Content language: English default; 繁體中文 via toggle. Never Simplified Chinese.
- No email address anywhere on the site. Links: LinkedIn `https://www.linkedin.com/in/cheng-i-wang/`, Scholar `https://scholar.google.com/citations?user=SmuY2sMAAAAJ`.
- Photos are never deleted. Old URL `/publications.html` stays valid.
- No guessed URLs in publication data.
- Commit messages in English, ending with `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`. Never push.

## Toolchain

- `JEKYLL="$HOME/.gem/ruby/2.6.0/bin/jekyll"` (installed with `gem install --user-install jekyll`). All build steps below use `$JEKYLL build` from the worktree root. Output goes to `_site/` (gitignored).
- Preview: `python3 -m http.server 4000 -d _site` via `.claude/launch.json`, opened with the built-in browser.
- Tracing venv: `/private/tmp/claude-504/-Users-cheng-iwang-Repos-wangsix-github-io--claude-worktrees-personal-website-redesign-71a6ce/a6bf81b2-13b1-43c9-80d3-0e6627b0a198/scratchpad/trace-venv` (has `potracer`, `pillow`, `numpy`).

## File Structure

| Path | Responsibility |
|---|---|
| `_config.yml` | site metadata, excludes |
| `Gemfile` | local preview with a modern Ruby (`github-pages`) |
| `_layouts/default.html` | whole page chrome: head, header/nav/lang toggle, footer, JS |
| `_includes/logo.svg` | traced logo mark, `fill="currentColor"` (Task 7) |
| `_data/nav.yml` | nav entries, bilingual labels, `hidden` flag |
| `_data/bio.yml` | name, headline, bio paragraphs, EN + ZH |
| `_data/updates.yml` | dated updates, EN + ZH, optional url |
| `_data/publications.yml` | papers/theses/reports |
| `_data/patents.yml` | two patents |
| `index.html` | home: hero, about, updates |
| `publications.html` | publications by year + patents |
| `projects.html`, `music.html` | `published: false` drafts |
| `css/site.css` | all styling |
| `assets/fonts/*.woff2` | already present |
| `README.md` | maintenance instructions |

---

### Task 1: Config, Gemfile, dead-asset removal, build baseline

**Files:**
- Modify: `_config.yml`
- Create: `Gemfile`
- Modify: `.gitignore`
- Delete: `css/style.css`, `css/stylesheets/` (3 files), `images/` (4 files), `img/{x,x3,x4,xh,xh4,next,prev,back,back2,down,download,newwindow,newwindow2,newwindow3}.png`, `img/newwindow.psd`, `img/bg.png`, `img/bg.jpg`

**Interfaces:**
- Produces: `site.title`, `site.description`, `site.url` used by the layout; `_site/` build output used by every later verification step.

- [ ] **Step 1: Confirm the build baseline fails for the right reason**

Run: `$JEKYLL build 2>&1 | tail -5`
Expected: builds (old site) or errors mentioning missing gems. Either way, note the output; this is the "before".

- [ ] **Step 2: Write `_config.yml`**

```yaml
title: Cheng-i Wang
description: Cheng-i Wang (汪正一) — Deep Learning Engineer at AudioShake. PhD in Computer Music, UC San Diego. Audio source separation, music information retrieval, bassist.
url: https://wangsix.github.io
baseurl: ""
lang: en
timezone: America/Los_Angeles

exclude:
  - docs
  - README.md
  - Gemfile
  - Gemfile.lock
  - vendor
  - .claude
  - node_modules
```

- [ ] **Step 3: Write `Gemfile`**

```ruby
source "https://rubygems.org"
# Local preview with a modern Ruby: `bundle install && bundle exec jekyll serve`.
# GitHub Pages builds the site itself; this file is not used in production.
gem "github-pages", group: :jekyll_plugins
```

- [ ] **Step 4: Update `.gitignore`**

```
_site/
.sass-cache/
.jekyll-metadata
.jekyll-cache/
Gemfile.lock
vendor/
```

- [ ] **Step 5: Delete dead assets**

```bash
git rm -q css/style.css css/stylesheets/base.css css/stylesheets/skeleton.css css/stylesheets/layout.css \
  images/favicon.ico images/apple-touch-icon.png images/apple-touch-icon-114x114.png images/apple-touch-icon-72x72.png \
  img/x.png img/x3.png img/x4.png img/xh.png img/xh4.png img/next.png img/prev.png img/back.png img/back2.png \
  img/down.png img/download.png img/newwindow.png img/newwindow2.png img/newwindow3.png img/newwindow.psd \
  img/bg.png img/bg.jpg
ls img
```
Expected `ls img`: `banner.jpg banner2.jpg banner3.jpg banner4.jpg bannerEFX.jpg bannerLDA.jpg bannerMO.jpg bannerPilot.jpg chengiwangwebsite.png favsix.png favsix114.png favsix72.png logo.png logo_hover.png p11.jpg p9.jpg`

- [ ] **Step 6: Build**

Run: `$JEKYLL build 2>&1 | tail -3 && ls _site`
Expected: "done in N seconds"; `_site` contains `index.html`, `publications.html`, `img/`, `assets/`, no `docs/`, no `Gemfile`.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "Reset site config and remove Skeleton CSS and dead assets

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 2: Data files

**Files:**
- Create: `_data/nav.yml`, `_data/bio.yml`, `_data/updates.yml`, `_data/publications.yml`, `_data/patents.yml`

**Interfaces:**
- Produces (consumed by Tasks 3–5):
  - `site.data.nav[]`: `label_en`, `label_zh`, `url`, `hidden` (bool, optional)
  - `site.data.bio`: `name`, `name_zh`, `nickname_zh`, `headline.en|zh`, `paragraphs.en[]|zh[]`
  - `site.data.updates[]`: `date` (YYYY-MM), `en`, `zh`, `url` (optional)
  - `site.data.publications[]`: `year` (int), `authors`, `title`, `venue`, `url` (optional), `note` (optional)
  - `site.data.patents[]`: `title`, `number`, `status`, `date`, `inventors`, `url`

- [ ] **Step 1: Write `_data/nav.yml`**

```yaml
- label_en: Home
  label_zh: 首頁
  url: /
- label_en: Publications
  label_zh: 論文
  url: /publications.html
- label_en: Projects
  label_zh: 專案
  url: /projects.html
  hidden: true
- label_en: Music
  label_zh: 音樂
  url: /music.html
  hidden: true
```

- [ ] **Step 2: Write `_data/bio.yml`**

```yaml
name: Cheng-i Wang
name_zh: 汪正一
nickname_zh: 汪六
headline:
  en: Deep Learning Engineer at AudioShake
  zh: AudioShake 深度學習工程師
paragraphs:
  en:
    - >-
      Cheng-i Wang (nicknamed “Wang Six”) is a Deep Learning Engineer at
      AudioShake, where he develops audio source separation technology and
      brings deep learning models into production. Before joining AudioShake,
      he spent four years as an Audio Software Engineer at Smule, building
      data-driven solutions for karaoke at consumer scale and architecting
      large-scale audio processing pipelines and ML infrastructure.
    - >-
      His training combines engineering and music: a PhD in Computer Music
      from UC San Diego, an MM in Music Technology from NYU, and a BBA in
      Finance from National Taiwan University. He was a recipient of the
      Taiwan Ministry of Education Government Scholarship for Overseas Study
      (Applied Music) in 2009.
    - >-
      Before moving abroad for graduate studies, he was an active working
      musician in Taiwan: bassist and bandleader of Apay, bassist for live
      concerts by Sandee Chan and Waa Wei, and composer, arranger and
      performing music director for Tainaner Ensemble’s stage production
      Castrated Chicken.
  zh:
    - >-
      汪正一（汪六），AudioShake 深度學習工程師，做 AI 音訊分離。加入 AudioShake
      之前，於 Smule 擔任音訊軟體工程師四年，負責卡拉 OK 應用的音訊處理與機器學習系統。
    - >-
      UCSD 電腦音樂博士、NYU 音樂科技碩士、台大財金學士，2009
      年教育部公費留學獎學金（應用音樂）。
    - >-
      出國前是職業 bass 手——阿霈樂團（Apay）團長、陳珊妮與魏如萱演唱會樂手、臺南人劇團《閹雞》配樂兼台南場指揮。
```

- [ ] **Step 3: Write `_data/updates.yml`** (dates marked approx. are to be corrected by the owner)

```yaml
# Newest first. date is YYYY-MM. url is optional.
- date: 2026-06
  en: Spoke at Taiwan Multimedia Tech meetup #12 in Taipei on “What LLMs Left Untouched in Audio”, a look at the structural split inside audio AI.
  zh: 於台北 Taiwan Multimedia Tech #12 分享「LLM 沒吃掉的那塊 Audio」，談 audio AI 的結構性分裂。
- date: 2026-04   # approx.
  en: AudioShake launched Dialogue RT, dialogue isolation at 11 ms end-to-end for live broadcast. I worked on getting the low-latency separation to ship.
  zh: AudioShake 發表 Dialogue RT，11 毫秒端到端的即時人聲分離，用於直播。我負責低延遲分離的部分。
- date: 2026-01   # approx.
  en: Gave a tech talk at KKCompany (KKBOX) in Taipei on AI source separation and how it differs from generative models.
  zh: 於台北 KKCompany（KKBOX）分享 AI 音訊分離技術，以及它與生成式模型的差異。
- date: 2023-03
  en: Joined AudioShake as a Deep Learning Engineer.
  zh: 加入 AudioShake 擔任深度學習工程師。
- date: 2022-05
  en: Co-authored “ToNet: Tone-Octave Network for Singing Melody Extraction from Polyphonic Music” at ICASSP 2022.
  zh: 合著論文 “ToNet: Tone-Octave Network for Singing Melody Extraction from Polyphonic Music” 發表於 ICASSP 2022。
- date: 2020-05
  en: Co-authored “Deep Autotuner” at ICASSP 2020 and the TISMIR survey on audio-based music structure analysis.
  zh: 合著 “Deep Autotuner”（ICASSP 2020）與 TISMIR 音樂結構分析回顧論文。
- date: 2019-10
  en: US patent 10,446,123, “Intuitive music visualization using efficient structural segmentation”, issued (with Adobe Research).
  zh: 美國專利 10,446,123「Intuitive music visualization using efficient structural segmentation」核准（與 Adobe Research 合作）。
- date: 2018-04
  en: Joined Smule as an Audio Software Engineer, and presented “Singing Style Investigation by Residual Siamese Convolutional Neural Networks” at ICASSP 2018 in Calgary.
  zh: 加入 Smule 擔任音訊軟體工程師，並於 ICASSP 2018（卡加利）發表 “Singing Style Investigation by Residual Siamese Convolutional Neural Networks”。
  url: http://acsweb.ucsd.edu/~chw160/pdf/icassp_2018_singing_style_Wang_Tzanetakis.pdf
- date: 2018-03
  en: Graduated from UC San Diego with a PhD in Music (Computer Music).
  zh: 取得 UC San Diego 音樂博士（電腦音樂）學位。
```

- [ ] **Step 4: Write `_data/publications.yml`**

```yaml
# One entry per paper. year is an integer; entries within a year keep this order.
# url only when known for certain; note is free text shown after the venue.
- year: 2025
  authors: I. Shapiro, R. Huang, Z. Novack, C. Wang, H.-W. Dong, T. Berg-Kirkpatrick, et al.
  title: Synthesizing Composite Hierarchical Structure from Symbolic Music Corpora
  venue: arXiv:2502.15849
  url: https://arxiv.org/abs/2502.15849
- year: 2023
  authors: O. Cífka, C. Dimitriou, C. Wang, H. Schreiber, L. Miner, F.-R. Stöter
  title: "Jam-ALT: A Formatting-Aware Lyrics Transcription Benchmark"
  venue: arXiv:2311.13987
  url: https://arxiv.org/abs/2311.13987
- year: 2022
  authors: K. Chen, S. Yu, C. Wang, W. Li, T. Berg-Kirkpatrick, S. Dubnov
  title: "ToNet: Tone-Octave Network for Singing Melody Extraction from Polyphonic Music"
  venue: Proceedings of IEEE ICASSP 2022
- year: 2021
  authors: S. Dubnov, K. Huang, C. Wang
  title: Towards Cross-Cultural Analysis Using Music Information Dynamics
  venue: arXiv:2111.12588
  url: https://arxiv.org/abs/2111.12588
- year: 2020
  authors: O. Nieto, G. J. Mysore, C. Wang, J. B. L. Smith, J. Schlüter, T. Grill, B. McFee
  title: "Audio-Based Music Structure Analysis: Current Trends, Open Challenges, and Applications"
  venue: Transactions of the International Society for Music Information Retrieval, 3(1), 2020
- year: 2020
  authors: K. Chen, C. Wang, T. Berg-Kirkpatrick, S. Dubnov
  title: "Music SketchNet: Controllable Music Generation via Factorized Representations of Pitch and Rhythm"
  venue: Proceedings of ISMIR 2020
  url: https://arxiv.org/abs/2008.01291
- year: 2020
  authors: S. Wager, G. Tzanetakis, C. Wang, M. Kim
  title: "Deep Autotuner: A Pitch Correcting Network for Singing Performances"
  venue: Proceedings of IEEE ICASSP 2020
- year: 2019
  authors: S. Wager, G. Tzanetakis, S. Sullivan, C. Wang, J. Shimmin, M. Kim, P. Cook
  title: "Intonation: A Dataset of Quality Vocal Performances Refined by Spectral Clustering on Pitch Congruence"
  venue: Proceedings of IEEE ICASSP 2019
- year: 2019
  authors: S. Wager, G. Tzanetakis, C. Wang, L. Guo, A. Sivaraman, M. Kim
  title: "Deep Autotuner: A Data-Driven Approach to Natural-Sounding Pitch Correction for Singing Voice in Karaoke Performances"
  venue: arXiv:1902.00956
  url: https://arxiv.org/abs/1902.00956
- year: 2018
  authors: C. Wang, G. Tzanetakis
  title: Singing Style Investigation by Residual Siamese Convolutional Neural Networks
  venue: Proceedings of IEEE ICASSP 2018
  url: http://acsweb.ucsd.edu/~chw160/pdf/icassp_2018_singing_style_Wang_Tzanetakis.pdf
- year: 2018
  authors: C. Wang
  title: Analysis and Generation of Music Signals with the Variable Markov Oracle
  venue: PhD dissertation, University of California, San Diego
- year: 2017
  authors: C. Wang, G. J. Mysore, S. Dubnov
  title: Re-Visiting the Music Segmentation Problem with Crowdsourcing
  venue: Proceedings of ISMIR 2017
  url: https://ismir2017.smcnus.org/wp-content/uploads/2017/10/102_Paper.pdf
- year: 2017
  authors: C. Wang, S. Dubnov
  title: Context-Aware Hidden Markov Models of Jazz Music with Variable Markov Oracle
  venue: 5th International Workshop on Musical Metacreation (MUME 2017), at ICCC 2017
  url: https://www.researchgate.net/profile/Shlomo_Dubnov/publication/320268052_Context-Aware_Hidden_Markov_Models_of_Jazz_Music_with_Variable_Markov_Oracle/links/59d8f28daca272e60966c983/Context-Aware-Hidden-Markov-Models-of-Jazz-Music-with-Variable-Markov-Oracle.pdf
- year: 2016
  authors: C. Wang, J. Hsu, S. Dubnov
  title: "Machine Improvisation with Variable Markov Oracle: Toward Guided and Structured Improvisation"
  venue: Computers in Entertainment, 14(3), 2016
- year: 2016
  authors: C. Wang, G. J. Mysore
  title: Structural Segmentation with the Variable Markov Oracle and Boundary Adjustment
  venue: Proceedings of IEEE ICASSP 2016
  url: https://ccrma.stanford.edu/~gautham/Site/Publications_files/segmentation-icassp_2016.pdf
- year: 2015
  authors: C. Wang, S. Dubnov
  title: "The Variable Markov Oracle: Algorithms for Human Gesture Applications"
  venue: IEEE MultiMedia, 22(4), 2015
  url: https://doi.org/10.1109/MMUL.2015.76
- year: 2015
  authors: C. Wang, J. Hsu, S. Dubnov
  title: "Music Pattern Discovery with Variable Markov Oracle: A Unified Approach to Symbolic and Audio Representations"
  venue: Proceedings of ISMIR 2015
  url: http://ismir2015.uma.es/articles/78_Paper.pdf
- year: 2015
  authors: T. Dubnov, C. Wang
  title: Free-body Gesture Tracking and Augmented Reality Improvisation for Floor and Aerial Dance
  venue: arXiv:1509.04751
  url: https://arxiv.org/abs/1509.04751
- year: 2015
  authors: C. Wang, S. Dubnov
  title: "Pattern Discovery from Audio Recordings by Variable Markov Oracle: A Music Information Dynamics Approach"
  venue: Proceedings of IEEE ICASSP 2015
  url: https://doi.org/10.1109/ICASSP.2015.7178056
- year: 2014
  authors: C. Wang, S. Dubnov
  title: "Variable Markov Oracle: A Novel Sequential Data Points Clustering Algorithm with Application to 3D Gesture Query-Matching"
  venue: Proceedings of IEEE International Symposium on Multimedia (ISM 2014)
  url: https://doi.org/10.1109/ISM.2014.39
- year: 2014
  authors: C. Wang, T. Smyth, Z. C. Lipton
  title: Estimation of Saxophone Control Parameters by Convex Optimization
  venue: Proceedings of the 9th Conference on Interdisciplinary Musicology (CIM14)
  url: http://acsweb.ucsd.edu/~chw160/pdf/cim14_sax_convex_Wang_Smyth_Lipton.pdf
- year: 2014
  authors: C. Wang, S. Dubnov
  title: Guided Music Synthesis with Variable Markov Oracle
  venue: 3rd International Workshop on Musical Metacreation (MUME 2014)
  url: http://acsweb.ucsd.edu/~chw160/pdf/mume2014_vmo_Wang_Dubnov.pdf
- year: 2014
  authors: T. Smyth, C. Wang
  title: Toward Real-Time Estimation of Tonehole Configuration
  venue: Proceedings of the Joint ICMC–SMC 2014 Conference
  url: http://acsweb.ucsd.edu/~chw160/pdf/smc14_saxophone_Smyth_Wang.pdf
- year: 2013
  authors: C. Wang
  title: "Quantifying Pianist Style: An Investigation of Performer Space and Expressive Gestures from Audio Recordings"
  venue: Master’s thesis, New York University
  url: http://chengiwang.com/pdf/Cheng-i_Wang_master_thesis.pdf
- year: 2011
  authors: C. Wang
  title: A Report on Information Theoretic Approaches with Melodic Boundaries Detection
  venue: Technical report
```

- [ ] **Step 5: Write `_data/patents.yml`**

```yaml
- title: Intuitive music visualization using efficient structural segmentation
  number: US 10,446,123
  status: issued
  date: 2019-10-15
  inventors: C. Wang, G. J. Mysore
  url: https://patents.google.com/patent/US10446123B2/en
- title: Densification in Music Search and Recommendation
  number: US 2022/0222294 A1
  status: application filed
  date: 2021-02-22
  inventors: C. Wang, S. Sullivan, D. A. Steinwedel, G. Tzanetakis
  url: https://patents.google.com/patent/US20220222294A1/en
```

- [ ] **Step 6: Validate the YAML parses and counts are right**

Run:
```bash
ruby -ryaml -e 'd=Dir["_data/*.yml"].map{|f|[f,YAML.load_file(f)]}.to_h; d.each{|f,v| puts "#{f}: #{v.size}"}; abort "bad year" unless d["_data/publications.yml"].all?{|p| p["year"].is_a?(Integer)}'
```
Expected: `_data/bio.yml: 5`, `_data/nav.yml: 4`, `_data/patents.yml: 2`, `_data/publications.yml: 25`, `_data/updates.yml: 9`, no abort.

- [ ] **Step 7: Check the publication links are alive** (report only; do not delete entries)

Run:
```bash
ruby -ryaml -e 'YAML.load_file("_data/publications.yml").each{|p| puts p["url"] if p["url"]}' | while read u; do printf "%s  %s\n" "$(curl -sS -o /dev/null -L --max-time 15 -w '%{http_code}' "$u" 2>/dev/null || echo ERR)" "$u"; done
```
Expected: mostly 200. Record any non-200 URLs for the final report to the owner (acsweb.ucsd.edu and chengiwang.com links are the likely casualties). Leave them in place.

- [ ] **Step 8: Commit**

```bash
git add _data
git commit -m "Add site content as YAML data files

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 3: Layout and stylesheet

**Files:**
- Rewrite: `_layouts/default.html`
- Create: `css/site.css`

**Interfaces:**
- Consumes: `site.data.nav`, `site.title`, `site.description`, `page.title`, `page.description`, `page.url`.
- Produces for pages: CSS classes `.photo`, `.photo--hero`, `.photo--ribbon`, `.label`, `.muted-serif`, `.about`, `.portrait`, `.headline`, `.updates`, `.pubs`, `.pub-title`, `.pub-authors`, `.pub-venue`, `.pub-note`; the convention that any element with `lang="en"` or `lang="zh-Hant"` is shown only in that language.

- [ ] **Step 1: Write `_layouts/default.html`**

```html
<!DOCTYPE html>
<html lang="en" data-lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% if page.title %}{{ page.title }} · {% endif %}{{ site.title }}</title>
  <meta name="description" content="{{ page.description | default: site.description }}">
  <meta name="color-scheme" content="light dark">
  <meta name="theme-color" content="#f6f3e6" media="(prefers-color-scheme: light)">
  <meta name="theme-color" content="#1d1b17" media="(prefers-color-scheme: dark)">
  <link rel="canonical" href="{{ page.url | absolute_url }}">
  <meta property="og:type" content="website">
  <meta property="og:title" content="{% if page.title %}{{ page.title }} · {% endif %}{{ site.title }}">
  <meta property="og:description" content="{{ page.description | default: site.description }}">
  <meta property="og:image" content="{{ '/img/banner.jpg' | absolute_url }}">
  <meta property="og:url" content="{{ page.url | absolute_url }}">
  <link rel="icon" href="{{ '/favicon.ico' | relative_url }}">
  <link rel="apple-touch-icon" href="{{ '/img/favsix114.png' | relative_url }}">
  <link rel="stylesheet" href="{{ '/css/site.css' | relative_url }}">
  <script>
    (function () {
      try {
        var l = localStorage.getItem('lang');
        if (l === 'zh' || l === 'en') {
          document.documentElement.setAttribute('data-lang', l);
          document.documentElement.lang = l === 'zh' ? 'zh-Hant' : 'en';
        }
      } catch (e) {}
    })();
  </script>
</head>
<body>
  <header class="site-header">
    <a class="brand" href="{{ '/' | relative_url }}">
      <img class="brand-mark" src="{{ '/img/chengiwangwebsite.png' | relative_url }}" alt="" width="44" height="42">
      <span class="brand-name"><span lang="en">Cheng-i Wang</span><span lang="zh-Hant">汪正一</span></span>
    </a>
    <nav class="site-nav" aria-label="Site">
      {%- for item in site.data.nav -%}
      {%- unless item.hidden -%}
      <a href="{{ item.url | relative_url }}"{% if page.url == item.url %} aria-current="page"{% endif %}><span lang="en">{{ item.label_en }}</span><span lang="zh-Hant">{{ item.label_zh }}</span></a>
      {%- endunless -%}
      {%- endfor -%}
      <button class="lang-toggle" type="button" aria-label="Switch language / 切換語言"><span lang="en">中文</span><span lang="zh-Hant">EN</span></button>
    </nav>
  </header>

  <main class="site-main">
    {{ content }}
  </main>

  <footer class="site-footer">
    <a href="https://www.linkedin.com/in/cheng-i-wang/" rel="me">LinkedIn</a>
    <a href="https://scholar.google.com/citations?user=SmuY2sMAAAAJ" rel="me">Google Scholar</a>
    <span class="site-footer__copy">© {{ site.time | date: '%Y' }} Cheng-i Wang</span>
  </footer>

  <script>
    document.querySelector('.lang-toggle').addEventListener('click', function () {
      var h = document.documentElement;
      var l = h.getAttribute('data-lang') === 'zh' ? 'en' : 'zh';
      h.setAttribute('data-lang', l);
      h.lang = l === 'zh' ? 'zh-Hant' : 'en';
      try { localStorage.setItem('lang', l); } catch (e) {}
    });
  </script>
</body>
</html>
```

- [ ] **Step 2: Write `css/site.css`**

```css
/* ---------- Fonts (self-hosted, Latin only; CJK falls back to system) ---------- */
@font-face { font-family: "Inter"; font-style: normal; font-weight: 400; font-display: swap;
  src: url("../assets/fonts/inter-latin-400-normal.woff2") format("woff2"); }
@font-face { font-family: "Inter"; font-style: normal; font-weight: 500; font-display: swap;
  src: url("../assets/fonts/inter-latin-500-normal.woff2") format("woff2"); }
@font-face { font-family: "Inter"; font-style: normal; font-weight: 600; font-display: swap;
  src: url("../assets/fonts/inter-latin-600-normal.woff2") format("woff2"); }
@font-face { font-family: "Newsreader"; font-style: italic; font-weight: 400; font-display: swap;
  src: url("../assets/fonts/newsreader-latin-400-italic.woff2") format("woff2"); }

/* ---------- Tokens ---------- */
:root {
  --bg: #f6f3e6;
  --text: #5a5650;
  --text-strong: #181818;
  --muted: #8a857a;
  --serif-muted: #b0ab99;
  --rule: #e2dfd4;
  --link: #3f6a8f;
  --accent: #F15A29;
  --label-bg: #181818;
  --label-fg: #f6f3e6;
  --photo-filter: grayscale(.6) sepia(.2) contrast(.95);
  --logo-filter: none;
  --sans: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang TC", "Microsoft JhengHei", "Noto Sans TC", "Helvetica Neue", Arial, sans-serif;
  --serif-italic: "Newsreader", Georgia, "Times New Roman", "PingFang TC", "Noto Serif TC", serif;
}
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1d1b17;
    --text: #cfc8b8;
    --text-strong: #f1ecdd;
    --muted: #8f8878;
    --serif-muted: #9a937f;
    --rule: #33302a;
    --link: #9dbbd6;
    --label-bg: #f1ecdd;
    --label-fg: #1d1b17;
    --photo-filter: grayscale(.6) sepia(.2) brightness(.75);
    --logo-filter: invert(1) brightness(.92);
  }
}

/* ---------- Language switching ---------- */
[data-lang="en"] [lang="zh-Hant"] { display: none !important; }
[data-lang="zh"] [lang="en"] { display: none !important; }

/* ---------- Base ---------- */
* { box-sizing: border-box; }
html { -webkit-text-size-adjust: 100%; }
body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: var(--sans);
  font-size: 1rem;
  line-height: 1.65;
}
a { color: var(--link); text-decoration: none; }
a:hover, a:focus-visible { color: var(--accent); text-decoration: underline; text-underline-offset: .15em; }
h1, h2, h3 { color: var(--text-strong); font-weight: 600; line-height: 1.25; }
h1 { font-size: 1.75rem; margin: 1.25rem 0 .25rem; letter-spacing: -.01em; }
h2 { font-size: 1rem; margin: 2.75rem 0 1rem; }
p { margin: 0 0 1rem; }
img { max-width: 100%; height: auto; display: block; }
button { font: inherit; }

.site-header, .site-main, .site-footer {
  width: 100%;
  max-width: 760px;
  margin: 0 auto;
  padding-left: 20px;
  padding-right: 20px;
}
@media (max-width: 480px) {
  .site-header, .site-main, .site-footer { padding-left: 16px; padding-right: 16px; }
}

/* ---------- Header ---------- */
.site-header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: .75rem 1.5rem;
  padding-top: 1.5rem;
  padding-bottom: 1.25rem;
  border-bottom: 1px solid var(--rule);
}
.brand { display: inline-flex; align-items: center; gap: .75rem; color: var(--text-strong); }
.brand:hover { color: var(--text-strong); text-decoration: none; }
.brand-mark { width: 44px; height: auto; filter: var(--logo-filter); image-rendering: auto; }
.brand-name { font-weight: 600; letter-spacing: .12em; text-transform: uppercase; font-size: .85rem; }
.brand-name [lang="zh-Hant"] { letter-spacing: .3em; font-size: 1rem; }
.site-nav { display: flex; align-items: center; gap: 1.25rem; font-size: .85rem; letter-spacing: .1em; text-transform: uppercase; }
.site-nav a { color: var(--muted); }
.site-nav a[aria-current="page"] { color: var(--text-strong); }
.site-nav a:hover { color: var(--accent); text-decoration: none; }
.lang-toggle {
  background: none; border: 1px solid var(--rule); border-radius: 999px;
  color: var(--muted); cursor: pointer; padding: .15rem .7rem; letter-spacing: .05em; text-transform: none;
}
.lang-toggle:hover { color: var(--accent); border-color: var(--accent); }

/* ---------- Main / footer ---------- */
.site-main { padding-top: 1.5rem; padding-bottom: 4rem; }
.site-footer {
  display: flex; flex-wrap: wrap; gap: 1.25rem; align-items: center;
  padding-top: 1.25rem; padding-bottom: 2.5rem;
  border-top: 1px solid var(--rule);
  font-size: .8rem; letter-spacing: .05em; color: var(--muted);
}
.site-footer__copy { margin-left: auto; }

/* ---------- Signature pieces ---------- */
.label {
  display: inline-block;
  background: var(--label-bg);
  color: var(--label-fg);
  font-size: .72rem;
  font-weight: 500;
  letter-spacing: .14em;
  text-transform: uppercase;
  padding: .2em .55em .15em;
}
.muted-serif { font-family: var(--serif-italic); font-style: italic; color: var(--serif-muted); font-size: 1.05rem; }
.photo {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 3px;
  filter: var(--photo-filter);
}
.photo--hero { height: 220px; }
.photo--ribbon { height: 120px; }
@media (max-width: 480px) {
  .photo--hero { height: 160px; }
  .photo--ribbon { height: 90px; }
}

/* ---------- Home ---------- */
.about { display: grid; grid-template-columns: 1fr; gap: 0 2rem; margin-top: 1.5rem; }
.about:has(.portrait) { grid-template-columns: 1fr 160px; }
.about .portrait { grid-column: 2; grid-row: 1 / span 3; width: 160px; height: 160px; object-fit: cover; border-radius: 50%; filter: var(--photo-filter); margin-top: 1.5rem; }
@media (max-width: 600px) {
  .about:has(.portrait) { grid-template-columns: 1fr; }
  .about .portrait { grid-column: 1; grid-row: auto; margin-top: 0; }
}
.headline { font-family: var(--serif-italic); font-style: italic; color: var(--serif-muted); font-size: 1.15rem; margin-bottom: 1.25rem; }
.about-name-zh { font-size: 1rem; color: var(--muted); font-weight: 500; letter-spacing: .15em; margin-left: .5rem; }

.updates { list-style: none; margin: 0; padding: 0; }
.updates li { display: grid; grid-template-columns: 5.5rem 1fr; gap: .5rem 1rem; padding: .6rem 0; border-top: 1px solid var(--rule); }
.updates li:last-child { border-bottom: 1px solid var(--rule); }
.updates time { font-family: var(--serif-italic); font-style: italic; color: var(--serif-muted); font-size: .95rem; padding-top: .1rem; }
@media (max-width: 480px) {
  .updates li { grid-template-columns: 1fr; gap: 0; }
}

/* ---------- Publications ---------- */
.pubs { list-style: none; margin: 0; padding: 0; }
.pubs li { padding: .55rem 0; border-top: 1px solid var(--rule); }
.pubs li:last-child { border-bottom: 1px solid var(--rule); }
.pub-title { color: var(--text-strong); font-weight: 500; }
a.pub-title:hover { color: var(--accent); }
.pub-authors { color: var(--text); }
.pub-venue, .pub-note { font-family: var(--serif-italic); font-style: italic; color: var(--serif-muted); }
```

- [ ] **Step 3: Build and check the chrome**

Run:
```bash
$JEKYLL build 2>&1 | tail -2
grep -c 'class="site-header"' _site/index.html
grep -o 'href="[^"]*publications.html"' _site/index.html | head -1
grep -c 'lang="zh-Hant"' _site/index.html
grep -c 'projects.html' _site/index.html
test -f _site/css/site.css && test -f _site/assets/fonts/inter-latin-400-normal.woff2 && echo assets-ok
```
Expected: build done; `1`; `href="/publications.html"`; a number ≥ 3; `0` (hidden nav item absent); `assets-ok`.

- [ ] **Step 4: Commit**

```bash
git add _layouts/default.html css/site.css assets/fonts
git commit -m "Add new layout, stylesheet and self-hosted fonts

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 4: Home page

**Files:**
- Rewrite: `index.html`

**Interfaces:**
- Consumes: `site.data.bio`, `site.data.updates`, classes from Task 3, `site.static_files` for the portrait slot.

- [ ] **Step 1: Write `index.html`**

```html
---
layout: default
permalink: /
description: Cheng-i Wang (汪正一) — Deep Learning Engineer at AudioShake, PhD in Computer Music from UC San Diego, audio source separation and music information retrieval, bassist.
---
{%- assign bio = site.data.bio -%}
{%- assign portrait = site.static_files | where: "path", "/img/portrait.jpg" | first -%}

<img class="photo photo--hero" src="{{ '/img/banner.jpg' | relative_url }}" alt="A band performing on an outdoor stage" width="700" height="229">

<section class="about">
  {%- if portrait %}
  <img class="portrait" src="{{ '/img/portrait.jpg' | relative_url }}" alt="Portrait of Cheng-i Wang">
  {%- endif %}
  <h1>
    <span lang="en">{{ bio.name }} <span class="about-name-zh">{{ bio.name_zh }}</span></span>
    <span lang="zh-Hant">{{ bio.name_zh }}（{{ bio.nickname_zh }}）<span class="about-name-zh">{{ bio.name }}</span></span>
  </h1>
  <p class="headline"><span lang="en">{{ bio.headline.en }}</span><span lang="zh-Hant">{{ bio.headline.zh }}</span></p>
  <div lang="en">
    {%- for p in bio.paragraphs.en %}
    <p>{{ p }}</p>
    {%- endfor %}
  </div>
  <div lang="zh-Hant">
    {%- for p in bio.paragraphs.zh %}
    <p>{{ p }}</p>
    {%- endfor %}
  </div>
</section>

<section>
  <h2><span class="label"><span lang="en">Updates</span><span lang="zh-Hant">近況</span></span></h2>
  <ul class="updates">
    {%- for u in site.data.updates %}
    <li>
      <time datetime="{{ u.date }}">{{ u.date }}</time>
      <span>
        {%- if u.url %}<a href="{{ u.url }}"><span lang="en">{{ u.en }}</span><span lang="zh-Hant">{{ u.zh }}</span></a>
        {%- else %}<span lang="en">{{ u.en }}</span><span lang="zh-Hant">{{ u.zh }}</span>{% endif %}
      </span>
    </li>
    {%- endfor %}
  </ul>
</section>
```

- [ ] **Step 2: Build and check**

Run:
```bash
$JEKYLL build 2>&1 | tail -1
grep -c '<li>' _site/index.html
grep -c 'class="portrait"' _site/index.html
grep -o 'AudioShake' _site/index.html | wc -l
grep -c 'chw160' _site/index.html
```
Expected: `9` list items; `0` portrait (file absent); AudioShake count ≥ 4; `0` (old email gone).

- [ ] **Step 3: Prove the portrait slot works, then remove the probe**

Run:
```bash
cp img/p9.jpg img/portrait.jpg && $JEKYLL build >/dev/null 2>&1 && grep -c 'class="portrait"' _site/index.html; rm img/portrait.jpg
```
Expected: `1`. (`img/portrait.jpg` is removed again; `git status` must not show it.)

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Rebuild home page from bio and updates data

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 5: Publications page

**Files:**
- Rewrite: `publications.html`

**Interfaces:**
- Consumes: `site.data.publications`, `site.data.patents`, classes from Task 3.

- [ ] **Step 1: Write `publications.html`**

```html
---
layout: default
title: Publications
permalink: /publications.html
description: Publications and patents of Cheng-i Wang — music information retrieval, machine improvisation, audio source separation.
---
<img class="photo photo--ribbon" src="{{ '/img/banner4.jpg' | relative_url }}" alt="" width="700" height="229">

<h1><span lang="en">Publications</span><span lang="zh-Hant">論文與專利</span></h1>
<p class="muted-serif"><a href="https://scholar.google.com/citations?user=SmuY2sMAAAAJ">Google Scholar profile</a></p>

{%- assign groups = site.data.publications | group_by: "year" | sort: "name" | reverse %}
{%- for g in groups %}
<h2><span class="label">{{ g.name }}</span></h2>
<ol class="pubs">
  {%- for p in g.items %}
  <li>
    <span class="pub-authors">{{ p.authors }}.</span>
    {% if p.url %}<a class="pub-title" href="{{ p.url }}">{{ p.title }}</a>{% else %}<span class="pub-title">{{ p.title }}</span>{% endif %}.
    <span class="pub-venue">{{ p.venue }}.</span>
    {%- if p.note %} <span class="pub-note">{{ p.note }}</span>{% endif %}
  </li>
  {%- endfor %}
</ol>
{%- endfor %}

<h2><span class="label"><span lang="en">Patents</span><span lang="zh-Hant">專利</span></span></h2>
<ol class="pubs">
  {%- for p in site.data.patents %}
  <li>
    <span class="pub-authors">{{ p.inventors }}.</span>
    <a class="pub-title" href="{{ p.url }}">{{ p.title }}</a>.
    <span class="pub-venue">{{ p.number }}, {{ p.status }} {{ p.date }}.</span>
  </li>
  {%- endfor %}
</ol>
```

- [ ] **Step 2: Build and check ordering and counts**

Run:
```bash
$JEKYLL build 2>&1 | tail -1
grep -o '<span class="label">[0-9]\{4\}</span>' _site/publications.html | tr -d '<>/a-z="' | tr '\n' ' '; echo
grep -c '<li>' _site/publications.html
grep -c 'patents.google.com' _site/publications.html
```
Expected: `2025 2023 2022 2021 2020 2019 2018 2017 2016 2015 2014 2013 2011`; `27` (25 papers + 2 patents); `2`.

- [ ] **Step 3: Commit**

```bash
git add publications.html
git commit -m "Rebuild publications page from data, add patents

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 6: Draft pages and README

**Files:**
- Rewrite: `projects.html` (unpublished)
- Create: `music.html` (unpublished)
- Create: `README.md`

- [ ] **Step 1: Write `projects.html`**

```html
---
layout: default
title: Projects
permalink: /projects.html
published: false
---
<img class="photo photo--ribbon" src="{{ '/img/banner2.jpg' | relative_url }}" alt="" width="700" height="229">
<h1><span lang="en">Projects</span><span lang="zh-Hant">專案</span></h1>
<p class="muted-serif">Content to be decided. Candidate items: vmo (Python Variable Markov Oracle library), Dialogue RT, Guided Synthesis with Markov Oracle, Topic Modeling of Audio Signals with LDA, Pd externals / audio effects, automatic crossfade point finding.</p>
```

- [ ] **Step 2: Write `music.html`**

```html
---
layout: default
title: Music
permalink: /music.html
published: false
---
<img class="photo photo--ribbon" src="{{ '/img/banner3.jpg' | relative_url }}" alt="" width="700" height="229">
<h1><span lang="en">Music</span><span lang="zh-Hant">音樂</span></h1>
<p class="muted-serif">Content to be decided. Candidate items: bassist for Sandee Chan and Waa Wei, Apay, recording engineer on Sodagreen’s Incomparable Beauty, Tainaner Ensemble’s Castrated Chicken, Kinematik Dance Theater projection design.</p>
```

- [ ] **Step 3: Write `README.md`**

````markdown
# wangsix.github.io

Personal site of Cheng-i Wang. Jekyll on GitHub Pages, no plugins.

## Editing content

Everything is in `_data/`:

- `updates.yml` — home page news. Newest first. Fields: `date` (`YYYY-MM`), `en`, `zh`, optional `url`.
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
````

- [ ] **Step 4: Build and check the drafts are excluded**

Run: `$JEKYLL build 2>&1 | tail -1; ls _site/projects.html _site/music.html _site/README.md 2>&1 | grep -c 'No such'`
Expected: `3`.

- [ ] **Step 5: Commit**

```bash
git add projects.html music.html README.md
git commit -m "Add unpublished Projects and Music drafts and README

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 7: Logo as SVG

**Files:**
- Create: `_includes/logo.svg`
- Modify: `_layouts/default.html` (brand mark)
- Modify: `css/site.css` (`.brand-mark`)

**Interfaces:**
- Produces: `_includes/logo.svg`, an `<svg>` with `viewBox`, `fill="currentColor"`, `aria-hidden="true"`.

- [ ] **Step 1: Trace `img/chengiwangwebsite.png` (the mark alone, 88×85)**

Save as `<scratchpad>/trace_logo.py` (the scratchpad is the parent of the tracing venv) and run it with the tracing venv's python from the worktree root:

```python
import numpy as np
from PIL import Image
import potrace

SRC = "img/chengiwangwebsite.png"
OUT = "_includes/logo.svg"
SCALE = 8

im = Image.open(SRC).convert("RGBA")
bg = Image.new("RGBA", im.size, (255, 255, 255, 255))
bg.alpha_composite(im)
g = bg.convert("L").resize((im.width * SCALE, im.height * SCALE), Image.LANCZOS)
bitmap = np.array(g) < 128           # True = ink

path = potrace.Bitmap(bitmap).trace(turdsize=4, alphamax=1.0, opticurve=True, opttolerance=0.2)

d = []
for curve in path:
    s = curve.start_point
    d.append(f"M{s.x:.1f},{s.y:.1f}")
    for seg in curve.segments:
        if seg.is_corner:
            d.append(f"L{seg.c.x:.1f},{seg.c.y:.1f}L{seg.end_point.x:.1f},{seg.end_point.y:.1f}")
        else:
            d.append(f"C{seg.c1.x:.1f},{seg.c1.y:.1f} {seg.c2.x:.1f},{seg.c2.y:.1f} {seg.end_point.x:.1f},{seg.end_point.y:.1f}")
    d.append("z")

w, h = im.width * SCALE, im.height * SCALE
svg = (f'<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 {w} {h}" '
       f'width="44" height="{round(44*h/w)}" aria-hidden="true" focusable="false">'
       f'<path fill="currentColor" fill-rule="evenodd" d="{"".join(d)}"/></svg>\n')
open(OUT, "w").write(svg)
print(OUT, len(svg), "bytes,", len(path.curves), "curves")
```

Run: `VENV=/private/tmp/claude-504/-Users-cheng-iwang-Repos-wangsix-github-io--claude-worktrees-personal-website-redesign-71a6ce/a6bf81b2-13b1-43c9-80d3-0e6627b0a198/scratchpad/trace-venv; $VENV/bin/python $VENV/../trace_logo.py`
Expected: `_includes/logo.svg <N> bytes, <small number> curves`, N under 20 KB.

- [ ] **Step 2: Eyeball the trace**

Copy `_includes/logo.svg` to `_site/logo-check.svg` after a build and open it in the built-in browser at a large size next to `img/chengiwangwebsite.png`. Accept if the brushstroke silhouette matches with no stray specks. If specks remain, raise `turdsize` to 8 and re-run. If the shape is mangled, stop, keep the PNG, and report.

- [ ] **Step 3: Switch the header to the inline SVG**

In `_layouts/default.html`, replace the `<img class="brand-mark" …>` line with:

```html
      <span class="brand-mark">{% include logo.svg %}</span>
```

In `css/site.css`, replace the `.brand-mark` rule with:

```css
.brand-mark { display: inline-flex; width: 44px; color: var(--text-strong); }
.brand-mark svg { width: 100%; height: auto; }
```
and delete the `--logo-filter` token from both `:root` blocks (no longer used).

- [ ] **Step 4: Build and check**

Run: `$JEKYLL build 2>&1 | tail -1; grep -c '<svg' _site/index.html; grep -c 'logo-filter' _site/css/site.css`
Expected: `1`; `0`.

- [ ] **Step 5: Commit**

```bash
git add _includes/logo.svg _layouts/default.html css/site.css
git commit -m "Trace logo mark to inline SVG

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```

---

### Task 8: Browser verification

**Files:**
- Create: `.claude/launch.json` (not committed; `.claude` is excluded from the build and ignored by git)

- [ ] **Step 1: Write `.claude/launch.json`**

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "site",
      "runtimeExecutable": "python3",
      "runtimeArgs": ["-m", "http.server", "4000", "-d", "_site"],
      "port": 4000
    }
  ]
}
```

- [ ] **Step 2: Build and start the preview**

Run `$JEKYLL build`, then `preview_start` with name `site`.

- [ ] **Step 3: Check each combination and screenshot**

For each: take a screenshot, read the page text, confirm no console errors.
1. Home, desktop, light, EN: header with logo + nav + toggle; hero photo muted; name, headline, three bio paragraphs, nine updates; footer with LinkedIn and Scholar only.
2. Click the toggle: page shows 汪正一（汪六）, Chinese headline, three Chinese paragraphs, Chinese update text, nav shows 首頁 / 論文, toggle reads EN. Reload: still Chinese (localStorage).
3. Publications page (via nav, in 中文): heading 論文與專利, years 2025 → 2011, patents section 專利. Toggle back to EN.
4. `resize_window` preset `mobile`: header wraps cleanly, no horizontal scroll (`document.documentElement.scrollWidth <= innerWidth` via javascript_tool), updates stack date-over-text.
5. `resize_window` colorScheme `dark`: background `#1d1b17`, labels cream with dark text, photos dimmer, logo light. Then back to `light` and preset `desktop`.
6. Navigate to `/projects.html` and `/music.html`: 404 from the Python server.
7. Navigate to `/publications.html` directly: 200 (old URL preserved).

- [ ] **Step 4: Fix anything found, rebuild, re-check, commit fixes**

```bash
git add -A
git commit -m "Fix layout issues found in browser verification

Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>"
```
(Only if fixes were needed.)

- [ ] **Step 5: Final report to owner**

List: dates marked approx. in `updates.yml`; dead publication links from Task 2 Step 7; two Scholar entries deliberately left out ("Learning audio features for singer identification and embedding", 2018, no venue; "Discovery of Repeated Themes and Sections with Variable Markov Oracle", undated); portrait slot instructions; that nothing was pushed.
