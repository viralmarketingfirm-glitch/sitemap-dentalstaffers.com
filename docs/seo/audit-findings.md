# SEO audit findings — sitemap-dentalstaffers.com

Date: 2026-05-15
Branch: `seo-audit-fixes`
Auditor: Claude (per `seo-audit-prompt.md`)

## Recon summary (100 words)

This repo is a **utility repo**, not the main site codebase. It hosts the public sitemap (`dentalstaffers-sitemap.xml`) and two Google Search Console verification files. The main DentalStaffers.com site lives in a separate Wix Vibe + Astro codebase (not in this repo). Therefore the vast majority of the master SEO audit prompt — Phase 0 stack detection, schema audit, internal linking, component-level meta cleanup, FAQ schema, Phase 1 schema builders, Phase 2 competitor SERP audit, Phase 4 content expansion — is **N/A for this repo**. Only the sitemap and verification files are in scope here.

## Repo architecture

| Item | Detail |
|---|---|
| Repo | `viralmarketingfirm-glitch/sitemap-dentalstaffers.com` |
| Files | `dentalstaffers-sitemap.xml`, `index.html`, `google31fe6ecb89ecb8ac.html` |
| Hosting | Inferred GitHub Pages (verification files indicate a static host) |
| Sitemap source | **Hand-maintained** (not generated from the main site's route config) |
| Main-site repo | Separate; an Astro + React Wix Vibe project — not in this repo |

## Audit matrix

Cross-reference the routes the main site actually serves (per `Router.tsx` in the Wix Vibe project) against the entries in this sitemap, BEFORE this PR.

| Route | Router.tsx | Sitemap (before) | Sitemap (after this PR) |
|---|---|---|---|
| `/` | ✓ | ✓ | ✓ |
| `/employers` | ✓ | ✓ | ✓ |
| `/about` | ✓ | ✓ | ✓ |
| `/new-graduates` | ✓ | ✓ | ✓ |
| `/for-applicants` | ✓ | ✓ | ✓ |
| `/for-employers` | ✓ | ✓ | ✓ |
| `/dental-staffing-new-york` | ✓ | ✓ | ✓ |
| `/dental-staffing-new-jersey` | ✓ | ✓ | ✓ |
| `/dental-staffing-rockland-county` | ✓ | ✓ | ✓ |
| `/dental-staffing-bergen-county` | ✓ | ✓ | ✓ |
| `/dental-staffing-manhattan` | ✓ | ✓ | ✓ |
| `/dental-staffing-westchester-county` | ✓ | ✓ | ✓ |
| `/dental-staffing-middlesex-county` | ✓ | ✓ | ✓ |
| `/dental-staffing-nassau-county` | ✓ | ✓ | ✓ |
| **`/dental-staffing-bronx-county`** | ✓ | **✗ MISSING** | ✓ added |
| **`/nj/essex-county-dental-staffing`** | ✓ | **✗ MISSING** | ✓ added |
| `/sitemap` | ✓ | ✓ (see HIGH-1 below) | ✓ (unchanged) |
| `/our-services` | ✓ | ✓ | ✓ |
| `/screening-process` | ✓ | ✓ | ✓ |
| `/staffing/dental-hygienists` | ✓ | ✓ | ✓ |
| `/staffing/dental-assistants` | ✓ | ✓ | ✓ |
| `/staffing/front-office` | ✓ | ✓ | ✓ |
| `/staffing/office-managers` | ✓ | ✓ | ✓ |
| `/staffing/dentists` | ✓ | ✓ | ✓ |
| `/staffing/registered-certified-dental-assistants` | ✓ | ✓ | ✓ |
| **Total** | **25** | **23** | **25** |

## Findings

### CRITICAL

**CRIT-1 — Two indexable routes missing from sitemap.** `OBVIOUS-FIX`
- `/dental-staffing-bronx-county` and `/nj/essex-county-dental-staffing` exist in the main site's `Router.tsx` but were absent from the sitemap.
- **Fixed in this PR.** Both added with `<priority>0.7</priority>` and `<changefreq>monthly</changefreq>` to match the pattern of the other county/state landing pages.

### HIGH

**HIGH-1 — `/sitemap` is in the sitemap but is a user-facing HTML index, not a search target.** `NEEDS-CONFIRMATION`
- The HTML sitemap page is a user navigation aid. It shouldn't compete for ranking against actual service pages. In the SOP-01 SEO config produced for the main site, this route is marked `robots: "noindex, follow"`.
- A noindex URL in a sitemap is a contradictory signal to crawlers — Google will pull it from indexing anyway, but the sitemap is the place to list URLs you want indexed.
- **Not removed in this PR.** Removing entries from a sitemap is a "changing canonical / removing rules" class change the audit prompt says to surface, not act on. Recommend removing in a follow-up PR if you agree.

**HIGH-2 — Sitemap is hand-maintained, not generated.** `NEEDS-CONFIRMATION`
- The drift bug above (CRIT-1) is the direct consequence: someone shipped new routes in the main Astro project without updating this separate sitemap repo. The audit prompt explicitly flags this as a smell ("hand-maintained sitemaps often drift from route config").
- **Recommended fix (not in this PR):** Generate the sitemap from the main site at build time. The main site already has `src/pages/sitemap.xml.ts` — that file is the right place to centralize. This repo could then be retired, OR a small CI job in the main repo could push the generated XML here on each build.
- This is out of scope for this repo on its own.

### MEDIUM

**MED-1 — No `<lastmod>` on existing entries.** `NEEDS-CONFIRMATION`
- AI search engines (ChatGPT Search, Perplexity, Gemini, Google AI Overviews) weight freshness signals. `<lastmod>` is a key one.
- Adding today's date to all 23 pre-existing entries would be misleading (they didn't all change today). The honest fix is per-route lastmod tracking, which requires the sitemap to be generated rather than hand-maintained (see HIGH-2).
- **Partial fix in this PR:** The 2 newly-added entries (`bronx-county`, `essex-county-dental-staffing`) carry `<lastmod>2026-05-15</lastmod>` because that's a date we can honestly stand behind.
- **Deferred:** Backfill on other entries until automated generation is in place.

**MED-2 — No `xmlns:image` namespace, no image entries.** `OBVIOUS-FIX (deferred)`
- For local-service pages, including `image:image` entries in the sitemap helps with Google Images and AI search visual results.
- Not done in this PR — requires the main site to produce verified per-page hero images first, and per the GROUND TRUTH rule we use only what's already in the codebase.

### LOW

**LOW-1 — Verification stub `index.html` has minimal content.** `OBVIOUS-FIX (not actioned)`
- The `index.html` serves Google verification only. That's fine — but if anyone navigates to the sitemap-repo host root, they see "Google Verification Active" plain text. Cosmetically harmless. Not touching it.

## Findings that are N/A for this repo

The master audit prompt covers a full site codebase. The following sections do not apply here because the main site lives in a separate repo:

- Stack detection beyond "static HTML+XML"
- Per-route SEO metadata audit (lives in the main site)
- `<title>`, meta, canonical, OG, Twitter card rendering (main site)
- Schema.org / JSON-LD audit (main site)
- Single source of truth for business profile (main site)
- Internal linking topology (main site)
- Content audit (titles, descriptions, H1s, image alts) (main site)
- AI-tell content rewrites (main site)
- Schema builders (Phase 1, main site)
- Competitor SERP audit (Phase 2, main site)
- Internal linking fixes (Phase 3, main site)
- New content expansion (Phase 4, main site)

These need to run inside the main DentalStaffers.com codebase (the Wix Vibe Astro project). I previously ran SOP-01 against a local copy of that codebase; the resulting drop-in files (`src/lib/seo-config.ts`, `src/pages/[...slug].astro`, `src/pages/index.astro`) address the most CRITICAL bug there: hardcoded `pageName: 'Home'` resulting in identical `<title>Home | DentalStaffers.com</title>` server-side on every page.

## What changed in this PR

1. `dentalstaffers-sitemap.xml` — added 2 missing routes with `<lastmod>2026-05-15</lastmod>`.
2. `docs/seo/audit-findings.md` — this document.
3. `docs/seo/decisions.md` — decisions log (companion to this file).

## Recommended follow-ups (not in this PR)

1. **HIGH-1 sign-off:** confirm `/sitemap` should be removed from this XML (it's `noindex`).
2. **HIGH-2:** automate sitemap generation from the main Astro project's route config. Retire this manual file once that's live.
3. **SOP-01 deployment:** deploy the three drop-in files from SOP-01 to the main site so titles/descriptions stop being identical across pages. This is the largest single SEO lever still open.
4. **Off-codebase work** (calibrated honestly, per the audit prompt's closing note): backlinks, real review collection on Google Business Profile, content velocity. A solid technical SEO build is a 6-8/10 outcome. A 9-10 needs these complementary efforts that don't live in any repo.
