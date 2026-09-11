# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page marketing site for Ascension Software (email retention agency), deployed to GitHub Pages at https://ascensionsoftware.co. No build step, no framework, no dependencies, no tests — three hand-written files (`index.html`, `styles.css`, `script.js`) plus a standalone `privacy-policy.html` and image assets at the repo root.

## Commands

```bash
python3 -m http.server 8000   # serve locally, then open http://localhost:8000
```

There is no build, lint, or test tooling. Deployment is a push to `main`: `.github/workflows/static.yml` uploads the whole repo root as the Pages artifact. Note that `.github/workflows/index.yml` is a byte-identical duplicate of `static.yml` — both fire on the same push and contend for the `pages` concurrency group, so edit/remove them together.

## Conventions that matter here

- **Cache busting is manual.** `index.html` loads `styles.css?v=2`. Bump that query string whenever you change CSS, or returning visitors keep the stale file. (`script.js` is loaded unversioned with `defer`.)
- **Section numbering.** Both `styles.css` and `script.js` are organized into large banner-commented sections (`/* ===== 5. FAQ ACCORDION ===== */`). Keep new code inside the matching section in both files rather than appending to the end. The numbering in `script.js` has gaps (8, 9, 11 are gone) — don't renumber.
- **IIFE modules.** Every behavior in `script.js` is a self-invoking `(function initX() { ... })()` that starts with a `getElementById`/`querySelector` guard and bails early if the element is absent. Two exceptions run from the `INIT` block at the bottom (`initScrollReveal`, `initStatsBannerComet`) because they are also callable elsewhere. Follow the guard pattern — it is what keeps `script.js` harmless on pages that lack a given section.
- **Design tokens.** Global palette/typography/radii live in `:root` at the top of `styles.css`. The hero dashboard has its own isolated token set (`--hd-*`) scoped to its container; don't mix the two.
- **`privacy-policy.html` is deliberately self-contained** — its own inline `<style>`, no `styles.css` or `script.js`. Changes to the global stylesheet do not reach it, so brand changes must be applied in both places.

## Content that lives in JavaScript, not HTML

- **FAQ.** The ten Q&A pairs are a `faqs` array inside `initFAQ` in `script.js`; `#faq-list` in `index.html` is an empty container. The same function also generates and injects the `FAQPage` and `Organization` JSON-LD into `<head>`, so editing FAQ copy silently changes the structured data — keep the org details (email, URL, description) there in sync with the `<meta>` tags in `index.html`.
- **Hero dashboard numbers.** KPI counters, the revenue chart, and the donut data are hardcoded in `initHeroDashboard`.
- **Hero headline rotation.** The cycling phrases are in `initHeroCycle`.

## Scroll-driven animation

`initStackingCards` drives the `#features` section by reading `getBoundingClientRect().top` on every scroll event and writing `transform`/`filter`/`zIndex` inline. It allocates `PX_PER_CARD` (600px) of scroll per card transition, so the section's sticky height in `styles.css` must stay consistent with the card count times that constant. It returns early below 768px, where the cards fall back to normal flow via CSS.

## Dead code

Several CSS sections have no corresponding markup: `CURSOR + TRACER`, `FLOW VISUAL`, `CLIENT LOGO BAR`, `FLOATING STATS`, `INTRO SCREEN — Gmail Dark Mock`. `script.js` section 4 (flow visual canvas) targets markup that no longer exists and no-ops. Treat these as removed features, not as styling you can rely on.

## Third-party embeds

Calendly (inline widget in `#book` plus direct booking links), YouTube testimonial iframes in `#results`, Google Fonts, and a `leadsy.ai` visitor-identification tag that is the first element in `<head>`. All are hardcoded — there is no config file.
