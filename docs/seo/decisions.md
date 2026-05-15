# SEO decisions log — sitemap-dentalstaffers.com

Date: 2026-05-15
Branch: `seo-audit-fixes`

This file records what was fixed in this PR, what was deferred, and the reasoning. Companion to `audit-findings.md`.

## Fixed

| ID | Change | Why | Confidence |
|---|---|---|---|
| CRIT-1 | Added `/dental-staffing-bronx-county` and `/nj/essex-county-dental-staffing` to sitemap with `priority=0.7`, `changefreq=monthly`, `lastmod=2026-05-15` | Both are live routes in the main site's Router.tsx but were absent from the sitemap. Pattern (priority/changefreq) matches the other county/state landing pages. `lastmod` reflects the actual date these were added to the sitemap. | High — verified against Router.tsx route list, no judgment call required. |
| Pass-2 | Normalized all 25 sitemap URLs from `https://dentalstaffers.com/...` → `https://www.dentalstaffers.com/...` | Production redirects apex → www. The "real" canonical hostname is www. Production's auto-generated sitemap uses www. This sitemap was the only crawl-relevant artifact using apex. | High — verified by curl on `https://dentalstaffers.com/`, `https://dentalstaffers.com/about`, and `https://dentalstaffers.com/sitemap.xml` (all 301 to www). |

## Deferred (need user confirmation before action)

| ID | Why deferred | Suggested action |
|---|---|---|
| HIGH-1 | Removing `/sitemap` from the sitemap.xml is a removal-of-existing-rules change. The audit prompt instructs surfacing this rather than acting unilaterally. | If you agree, open a one-line follow-up PR removing that `<url>` block. |
| HIGH-2 | Replacing the hand-maintained sitemap with build-time generation requires changes in the **other** repo (the Astro project) — out of scope for this repo. | Add `src/pages/sitemap.xml.ts` generation logic in the main site repo. Either retire this repo, or wire CI to push the generated XML here on each main-site deploy. |
| MED-1 | Adding `<lastmod>` to all 23 pre-existing entries with today's date would falsely signal a sitewide content refresh. | Only do this honestly via automated generation (see HIGH-2), keyed off git history per route. |
| MED-2 | No verified per-page imagery in the main codebase yet — every page falls back to the brand logo. Per GROUND TRUTH rule, no fabrication. | Add proper 1200×630 OG images per landing page on the main site first; then revisit. |

## Not actioned, by design

- **Did not merge this PR.** Per audit prompt rule 2: "Do NOT merge any PR yourself." Awaiting review.
- **Did not push to `main`.** Per rule 3: branch is `seo-audit-fixes`.
- **Did not touch `index.html` or the Google verification file.** They serve their purpose; cosmetic-only edits would risk breaking GSC verification.
- **Did not stack multiple PRs.** Only one coherent change here — adding routes + adjacent audit docs. No reason to split.
- **Did not add analytics, dependencies, or visual redesign.** Per rule 8.

## Honest calibration

This PR is a small, targeted fix on a small, single-purpose repo. Pass-2 audit changed the picture: SOP-01 is already deployed to production, so the title/description bug is closed. The new top-priority codebase bug is **canonical hostname pointing to a 301-redirector** (apex instead of www) — that lives in the main-site repo, not here.

Estimated impact of *this* PR alone, calibrated honestly: **1/10**. The sitemap in this repo is orphaned (production's robots.txt points elsewhere), so fixes here may never be crawled. The aligned-with-production hostname is more about "if anyone ever does pull this XML, it doesn't conflict with production canonicals" than about driving rankings.

**The high-leverage next moves are all in the main-site repo:** fix the canonical-hostname bug (CRIT-2), fix the broken `/logo.png` URL in schema (CRIT-3), add per-page schema variation, add real per-page BreadcrumbList. Roughly 6/10 → 8/10 of available technical-SEO impact lives there.
