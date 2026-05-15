# SEO decisions log — sitemap-dentalstaffers.com

Date: 2026-05-15
Branch: `seo-audit-fixes`

This file records what was fixed in this PR, what was deferred, and the reasoning. Companion to `audit-findings.md`.

## Fixed

| ID | Change | Why | Confidence |
|---|---|---|---|
| CRIT-1 | Added `/dental-staffing-bronx-county` and `/nj/essex-county-dental-staffing` to sitemap with `priority=0.7`, `changefreq=monthly`, `lastmod=2026-05-15` | Both are live routes in the main site's Router.tsx but were absent from the sitemap. Pattern (priority/changefreq) matches the other county/state landing pages. `lastmod` reflects the actual date these were added to the sitemap. | High — verified against Router.tsx route list, no judgment call required. |

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

This PR is a small, targeted fix on a small, single-purpose repo. It plugs a 2-route gap in the sitemap that was a real (but minor) indexing/crawl-budget issue. The larger SEO lever for this business is the main Wix Vibe site — the title/description bug I fixed via SOP-01 against the local zip. Until that fix is deployed to production, every non-home page on dentalstaffers.com is server-rendering an identical `<title>Home | DentalStaffers.com</title>`. That's the single biggest unblock.

Estimated impact of *this* PR alone, calibrated honestly: 1–2/10. Two missing pages get crawled and indexed faster. The other 23 pages were already in the sitemap; this PR doesn't affect them.
