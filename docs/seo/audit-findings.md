# SEO audit findings — sitemap-dentalstaffers.com

Date: 2026-05-15 (pass 1: file-level; pass 2: live HTML)
Branch: `seo-audit-fixes`
Auditor: Claude (per `seo-audit-prompt.md`)

## Pass-2 update — live HTML audit against production

After the first pass, I scanned the live production site (`https://www.dentalstaffers.com/`) and discovered three things that change the picture:

1. **SOP-01 IS already deployed.** Every non-home page now renders a unique server-side `<title>` and `<meta description>` keyed by route. The `<title>Home | DentalStaffers.com</title>` bug is closed in production. Verified on `/`, `/about`, `/staffing/dental-hygienists`. Titles/keywords match the `seo-config.ts` I produced from SOP-01 exactly.
2. **CRITICAL — Canonical-hostname mismatch with production.** Production canonical is `www.` (apex 301-redirects to www). Every page's `<link rel="canonical">`, `og:url`, `twitter:url`, and the JSON-LD `@id` / `url` in Organization/LocalBusiness schema all point to `https://dentalstaffers.com/...` (apex). They redirect. Canonical URLs SHOULD point to the actual canonical, not to a redirector. This is the highest-priority codebase fix on the main site right now. — fix is **in the main-site repo, not this one** (lives in `src/lib/seo-config.ts` constant `HOST` and `src/components/Head.tsx`'s `siteUrl` + the hardcoded JSON-LD).
3. **CRITICAL — Logo URL in JSON-LD schema 404s.** Organization schema references `https://dentalstaffers.com/logo.png`. That URL redirects to `https://www.dentalstaffers.com/logo.png` which **returns 404**. So the Organization schema's `logo` field is broken — Rich Results validators will flag this. — fix is **in `src/components/Head.tsx`** in the main-site repo (replace with the wixstatic.com brand image already used for `og:image`).
4. **This repo's sitemap is orphaned.** Production's `robots.txt` declares the sitemap at `https://www.dentalstaffers.com/sitemap.xml` (Wix's auto-generated one), NOT at `https://viralmarketingfirm-glitch.github.io/sitemap-dentalstaffers.com/dentalstaffers-sitemap.xml`. No search engine reaches this repo's sitemap unless someone manually submitted it to GSC.
5. **Wix's auto-generated production sitemap already contains all 25 routes.** Including the two routes my pass-1 PR added here (`bronx-county`, `essex-county-dental-staffing`). So the pass-1 fix was correct in principle but had no real-world crawl impact, because Google was getting the routes from Wix's sitemap, not this repo's. Confirmed by `curl https://www.dentalstaffers.com/sitemap.xml`.

**Net effect on this PR:** the most useful change I can make in *this* repo is to normalize the sitemap hostnames here to `www.` so they match production. Then I've documented findings 2 and 3, which need a follow-up PR against the main-site repo (not this one) to actually move the needle.

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
- Real-world crawl impact: low — production's auto-generated sitemap at `www.dentalstaffers.com/sitemap.xml` already contained both routes. This repo's sitemap is orphaned (see CRIT-4).

**CRIT-2 — Sitewide canonical hostname points to a redirector.** `OBVIOUS-FIX` (but in main-site repo, not this one)
- Live HTML on every page declares `<link rel="canonical" href="https://dentalstaffers.com/...">` (apex). Production redirects apex → www with a 301. So canonicals point at a URL that doesn't serve content, only redirects.
- Same hostname bug affects `og:url`, `twitter:url`, and JSON-LD `@id` / `url` fields in Organization, LocalBusiness, Service, BreadcrumbList schema.
- Lives in main-site repo:
  - `src/lib/seo-config.ts` constant `HOST = "https://dentalstaffers.com"` — change to `"https://www.dentalstaffers.com"`.
  - `src/components/Head.tsx` constant `siteUrl = "https://dentalstaffers.com"` and the inlined JSON-LD strings.
- **Not fixable from this repo.** Sample verified on 5 routes; bug is sitewide.

**CRIT-3 — Organization-schema logo URL 404s.** `OBVIOUS-FIX` (but in main-site repo)
- JSON-LD in `<head>` on every page contains: `"logo": { "@type": "ImageObject", "url": "https://dentalstaffers.com/logo.png" }`.
- `dentalstaffers.com/logo.png` → 301 → `www.dentalstaffers.com/logo.png` → **404**.
- Google Rich Results / Schema validators will flag this as broken. Affects every page sitewide.
- Fix in `src/components/Head.tsx`: either upload an actual logo at `/logo.png`, or change the URL to the wixstatic.com brand image that's already used everywhere for `og:image` and the favicon.

**CRIT-4 — This repo's sitemap is orphaned from production crawl path.** `NEEDS-CONFIRMATION`
- Production `robots.txt` declares `Sitemap: https://www.dentalstaffers.com/sitemap.xml` (Wix's auto-generated). It does NOT reference this repo's hosted file at `viralmarketingfirm-glitch.github.io/sitemap-dentalstaffers.com/dentalstaffers-sitemap.xml`.
- Practical implication: Google never sees this repo's sitemap unless someone manually submitted it via GSC. Fixes made here (CRIT-1, this PR's hostname normalization) won't reach search engines via the crawl path.
- **Recommend retiring this repo** OR submitting its sitemap to GSC explicitly as a supplementary source AND keeping it in lock-step with production. The first option is cleaner.

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

1. `dentalstaffers-sitemap.xml`:
   - Added the 2 missing routes (`/dental-staffing-bronx-county`, `/nj/essex-county-dental-staffing`) with `<lastmod>2026-05-15</lastmod>`.
   - **Normalized all 25 hostnames from `https://dentalstaffers.com/` → `https://www.dentalstaffers.com/`** to match production canonical hostname (which is the actual non-redirecting URL).
2. `docs/seo/audit-findings.md` — this document (pass-1 + pass-2 live-HTML audit).
3. `docs/seo/decisions.md` — decisions log (companion to this file).

## Recommended follow-ups (not in this PR)

**In the main-site repo (priority order):**

1. **CRIT-2: Fix canonical hostname sitewide.** Single-line change in `src/lib/seo-config.ts` (`HOST` constant) plus matching change in `src/components/Head.tsx` (`siteUrl` constant + the inlined JSON-LD `@id` URLs). Single largest open SEO bug.
2. **CRIT-3: Fix the broken `/logo.png` reference in Organization schema.** Either upload a logo at that path, or change the schema to use the wixstatic.com brand image already referenced for `og:image`. Until fixed, Rich Results validators flag every page as having broken Organization schema.
3. **MED — Add per-page schema variation.** Currently every page ships the same JSON-LD block. Service pages should ship `Service` schema specific to that service. Location pages should ship `Place` / extended `LocalBusiness` with `areaServed`. Article-type pages should ship `Article`.
4. **MED — Add `BreadcrumbList` schema per page.** Currently only Home is in the breadcrumb item list, on every page. Nested pages should declare their hub → child crumb path with absolute URLs.
5. **LOW — `og:image` is a favicon-sized brand PNG, not a 1200×630 social card.** Causes poor link-preview rendering on social/iMessage. Requires creating an actual social card image; not actionable without design input.

**In this repo:**

6. **HIGH-1 sign-off:** confirm `/sitemap` should be removed from this XML (it's `noindex` in `seo-config.ts`).
7. **CRIT-4 sign-off:** decide whether to retire this repo entirely. Production's robots.txt points at Wix's auto-generated sitemap, not here.
8. **HIGH-2:** automate sitemap generation from the main Astro project's route config. (Skip if retiring this repo per #7.)

**Off-codebase (calibrated honestly):** backlinks, real review collection on Google Business Profile, content velocity. A solid technical SEO build is a 6-8/10 outcome. A 9-10 needs these complementary efforts that don't live in any repo.
