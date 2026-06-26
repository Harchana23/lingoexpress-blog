# 📝 LingoExpress — Blog Content Engine

**🔗 Live site:** https://lingoexpress.com.sg

> **An 897-post SEO blog pipeline for [LingoExpress](https://lingoexpress.com.sg)** (a Singapore
> certified document-translation company) — every post individually LLM-written, grounded in real
> document anatomies, and held to a **measurable, automated quality gate**. Ships alongside the
> LingoExpress GoHighLevel / MCP instance config.

![Python](https://img.shields.io/badge/Python-3776AB)
![HTML](https://img.shields.io/badge/HTML-10MB%20corpus-E34F26)
![Posts](https://img.shields.io/badge/posts-897-success)
![QA](https://img.shields.io/badge/QA-automated%20gate-blueviolet)
![GoHighLevel](https://img.shields.io/badge/GoHighLevel-MCP-FF6A00)
![Visibility](https://img.shields.io/badge/visibility-Internal-lightgrey)

---

## TL;DR

**The problem.** Three earlier content batches (50 reviews + 100 service posts + 747 topic guides)
were near-**duplicate** — 0.91–0.95 pairwise similarity — over-branded, and riddled with internal
links pointing at posts that weren't published yet (instant 404s). Unusable for SEO.

**The solution.** Merge all three into **one 897-post set**, interleave by content type for variety,
and **regenerate every post from scratch** — one LLM call per title against a machine-readable spec,
*not* a string template — so each post is uniquely written and **grounded in a real document's anatomy**
(no hallucinated fields). A Python **QA gate** then enforces every rule with hard numeric thresholds
before a post is allowed to be marked "Done."

**Why this approach**

| | Templated content farm | This engine |
|---|---|---|
| Uniqueness | Shared boilerplate | Per-title generation, similarity-capped |
| Trust | Invented details | Grounded in real document anatomies |
| Quality | Spot-checked | **Automated gate** (links, pricing, structure, uniqueness) |
| SEO safety | Over-linked, 404s | 0 internal links, 2–3 brand links, authority links only when named |
| Repeatability | Manual | Resumable batch picker + idempotent index rebuild |

**Status:** ✅ all **897 generated, QA-clean, and tracked** (Google Sheet, every row *Done HTML*).
Whole-set sweep: **0** internal links · **0** pricing figures · hero + footer + SEO on every post ·
inter-post uniqueness healthy. **Next phase: publishing to GHL / Blogger.**

---

## What's inside

Two things for LingoExpress, in one repo:

1. **The SEO blog content pipeline** — 897 standalone HTML posts + the Python tooling that builds and QAs them *(the bulk of the repo)*.
2. **The LingoExpress GHL / MCP instance** — credentials, briefs, and conventions for driving the GoHighLevel sub-account (a sibling to the main `MCP-GHL` project).

> Heavy binaries (the 250-image library, PDFs) and all secrets (`.env`, `.mcp.json`) are **gitignored** —
> images live on disk + the GHL CDN, not in git.

---

## The content pipeline

**Every post** is a self-contained, colorful HTML document (the approved **"v3" template**):

- Navy H1 → **hero image** → H2s in **rotating accent colours** (each emoji-led, ≥4 distinct, no repeat on consecutive H2s) → **colored callout boxes** (*In short* / *Real example* / *Key rule* / *Watch out*) → verbatim navy contact footer.
- **~1000–1300 words**, second-person voice ("you/your"); the company is always third-person ("LingoExpress…").
- **Grounded in real document anatomies** — Chinese *Medical Certificate of Birth*, Malaysian *Form A*, the *hukou*, SPM grade legends, driving-licence class codes, JPN marriage/divorce records, etc. No hallucinated fields, agencies, or requirements.
- **No pricing** (explain what *drives* cost, point to a free quote); notarisation claims hedged.
- **Zero internal blog links**; 2–3 homepage links; outbound **authority links only when that agency is named** (ICA, MOM, ROM, SAL, LTA, JPN…).
- Unique `SEO_TITLE` (≤60) + `META_DESCRIPTION` (≤155) + a theme-matched hero served from the GHL CDN.

### Three rule layers, one standard

| Layer | File | Audience |
|---|---|---|
| Human do's & don'ts | [`BLOG_RULES.md`](BLOG_RULES.md) | the operator |
| Machine content spec | `blog-posts/combined/_GROUNDING.md` | the generator |
| Automated enforcement | `blog-posts/combined/_qa.py` | the gate |

---

## The QA gate (the standout)

A post is only **DONE** when **every** check passes — and they're quantified, not vibes:

- internal blog links **= 0**
- pricing figures **= 0**
- hero image + navy footer + SEO/meta comments **present**
- rotating H2 accents **≥ 4 distinct** · **3–4** colored callouts
- word count **~1000–1300** (hard floor 850)
- `"we/our"` only inside the quoted certification statement
- **uniqueness:** within-batch `SequenceMatcher < 0.30`, cross-set **4-gram Jaccard < 0.45**

That last line is the heart of it: the original batches sat at **0.91–0.95** similarity; the gate
mathematically refuses to let near-duplicates ship.

---

## Tooling

```bash
# Rebuild the 897-row master index from the 3 batch sources (idempotent)
python blog-posts/combined/build_combined_index.py

# Pick the next N posts that still have no HTML (resumable; writes _active_batch.json)
python blog-posts/combined/_next_batch.py 10

# QA specific posts by g_index — or, with no args, the whole set
python blog-posts/combined/_qa.py 11 12 13
```

Generation is **LLM-per-title** — each post written individually against `_GROUNDING.md` + the matching
reference sample, deliberately **not** a string template, to keep every post unique.

---

## How it flows

```
  3 legacy batches            combined master index            per-title generation
  (50 + 100 + 747)  ──────►   build_combined_index.py   ──────►  one LLM call / title,
   near-duplicate             (g_index·title·slug·theme·          grounded in _GROUNDING.md
                               image_cdn_url, interleaved)        + a real document anatomy
                                       │                                   │
                                       ▼                                   ▼
                            _next_batch.py (resumable)            897 standalone .html posts
                                       │                                   │
                                       ▼                                   ▼
                            _qa.py gate (links·pricing·       Google Sheet tracker (Done HTML)
                            structure·uniqueness)  ──────►     → next: publish to GHL / Blogger
```

---

## LingoExpress GHL / MCP instance

The repo is also the **LingoExpress** instance of the MCP-GHL setup — its own credentials and content,
but **sharing the built `ghl-extended` MCP server** from the main project (no duplicate ~411 MB install;
each folder runs the same Node binary in its own isolated process/env).

| Tool prefix | Server | Tools |
|---|---|---|
| `mcp__ghl__*` | Official HighLevel MCP (HTTP) | ~35 stable tools |
| `mcp__ghl-extended__*` | Community fork (stdio Node) | 520+ tools (workflows, invoices, voice AI, blogs…) |

Both target the LingoExpress sub-account only — no cross-account leakage. GHL gotchas are logged in
`docs/ghl-mcp-quirks.md` (read before any MCP-driven work). Secrets live only in this folder's gitignored
`.env` / `.mcp.json`.

---

## Repository structure

```
LingoExpress-Blog/
├── README.md
├── BLOG_RULES.md                       # human-facing do's & don'ts
├── blog-posts/
│   ├── combined/                       # ⭐ the canonical 897-post set
│   │   ├── html/                       #   897 standalone posts (NNN_slug.html)
│   │   ├── combined_index.csv / .json  #   master index
│   │   ├── _GROUNDING.md               #   machine content spec
│   │   ├── reference_samples/          #   5 approved v3 template posts (markup only)
│   │   ├── _source_archive/            #   original batch records (live GHL post IDs)
│   │   ├── build_combined_index.py     #   rebuild index
│   │   ├── _next_batch.py              #   resumable picker
│   │   └── _qa.py                      #   automated QA gate
│   └── "All batches combined image/"   #   250-image library (binaries gitignored; CSV tracked)
├── briefs/  docs/  pages/              # GHL funnel / website work
├── .env.example  .mcp.json.example     # config templates (real secrets gitignored)
└── .gitignore
```

---

## Lessons baked into the rules

- Near-duplicate batches (0.91–0.95 similarity) → **every post uniquely written and grounded**, never templated text.
- Early drafts over-branded (20+ mentions, 11–20 links) and read as spam → **brand rhythm: 2–3 links**.
- Internal links hit unpublished posts and the live URL is `/post/{slug}` not `/blog/` → **drop internal links, use authority links**.
- A scraping artifact leaked into 12 titles → **clean titles before they reach a slug or the sheet**.
- House rule: real translation examples in hand beat invented detail → **ground, don't guess**.

---

## Status & limitations

- Content generation is **complete and QA-clean**; the remaining phase is **publishing** to GHL / Blogger.
- Image binaries and secrets are gitignored — regenerate/re-supply locally.
- The GHL instance depends on the main `MCP-GHL` project's built server; rebuilding or deleting it affects this folder.

---

## Ownership

Internal **CyberG7 / LingoExpress** project — built and maintained by [@CyberG7-org](https://github.com/CyberG7-org).
All rights reserved. Not open for external contributions; issues and questions welcome.


## 📸 Screenshots

![Screenshot 1](assets/shot-01.png)

