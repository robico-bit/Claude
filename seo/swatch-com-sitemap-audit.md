# swatch.com XML sitemap audit

**Checked:** 2026-11-26, live fetch of `https://www.swatch.com/sitemap_index.xml`, all 11 child sitemaps, `robots.txt`, and a stratified sample of 182 URLs checked for status code, plus canonical/hreflang on 10 pages.

## Summary

The sitemap is valid and well within size limits, but it is **not clean**. By pattern counts plus the status sample, an estimated **9,000–10,000 of its 38,227 URLs (~25%) don't return 200**: they redirect (301/302/307), redirect off-domain to flikflak.com, or 404. Google treats a sitemap as a list of canonical, indexable URLs. This much noise wastes crawl budget and weakens the sitemap as a signal. On top of that, the hreflang clusters on the pages don't match the locales the sitemap publishes.

## Structure (what's fine)

| Item | Result |
|---|---|
| Index | Valid `<sitemapindex>`, 11 children, all HTTPS on www.swatch.com, all 200 `application/xml` |
| Children | `sitemap_0` (US/CA), `asia_0-1`, `eastafrica_0` (ar-sa only), `eu_0-4`, `int_0` (en-en, es-en), `southam_0` (es-ar, es-mx) |
| Size | 38,227 URLs, no duplicates; largest file 5,000 URLs / 1.2 MB (limit 50,000 / 50 MB) |
| Locales | 46 locale folders, ~770–940 URLs each |
| robots.txt | References the index (plus flikflak.com's); **no sitemap URL is blocked by robots rules** |
| Hosts/schemes | No http, non-www, query-string or staging-host URLs |
| Index `lastmod` | All 11 set to generation time (2026-11-26 15:45 UTC) |

Composition: ~18,400 product pages (557 distinct product codes), ~9,100 other `.html` pages, ~10,700 folder-style URLs, 46 homepages.

## Issues, by priority

### 1. Legacy `/swNNN/` shortcut URLs: ~4,270 URLs, mostly 301 (high)
Examples: `/en-cz/sw102/`, `/da-dk/sw122/`, `/fi-fi/sw607/`. 21 of 25 sampled returned **301**. Many redirect to **faceted URLs that robots.txt blocks**, e.g. `/en-cz/sw102/` → `/en-cz/watches/?prefn1=catalogLine&prefv1=Lady` (`Disallow: /*prefn1=*`). So the sitemap points Google through a redirect to a page it isn't allowed to crawl.
**Fix:** take all `/[a-z]{2}NNN/` shortcut URLs out of the sitemap. Keep the redirects for users, and point them at clean category URLs, not parameter URLs.

### 2. FlikFlak content on swatch.com that 302s to flikflak.com: ~3,000–5,800 URLs (high)
3,311 URLs have `flikflak` in the path, and 2,560 are FlikFlak product codes (FBNP…, ZF…). Many redirect **off-domain with a 302** through a Salesforce session hop:
`/de-de/nature-watches-for-kids/` → `/s/Sites-swarp-EU-Site/dw/shared_session_redirect?url=…flikflak.com…` → `flikflak.com/s/…&token=…` → final page (2+ hops, all temporary).
Older ones (`flikflak-halloween/`, `flikflak-summer-specials/`, `flikflak-plp-pink-watches/`) 301 to `/all-kids-watches/`.
**Fix:** take FlikFlak URLs out of the swatch.com sitemaps, since flikflak.com already has its own sitemap in robots.txt. Make the cross-domain moves **301s straight to the final flikflak.com URL**, without the session-redirect hop for bots.

### 3. Test / QA pages in production sitemap: 116 URLs, all 404 (high, quick win)
`swatchHometest.html`, `…/green-abyss-qa-test.html`, `…/green-abysstest.html`, `swatch-pay/how-it-works-QA-test.html` appear across locales. All 25 sampled returned **404**.
**Fix:** exclude pages whose names contain `test`/`QA` from sitemap generation, or better, from production publishing.

### 4. Malformed ja-jp URL with an infinite redirect loop (high, quick win)
`https://www.swatch.com/ja-jp/https%3A//www.swatch.com/ja-jp/home/swatchHome.html` (and its `swatchHometest` twin) is a full URL pasted into a path. It **301s to itself** forever.
**Fix:** take it out of the sitemap, find the CMS page or link with the bad URL field, and make the URL return 404/410.

### 5. Retired campaign and legacy pages that redirect (medium)
Campaign and legacy pages still in the sitemap redirect, often to the homepage or to unrelated targets:
- `swatch-x-supriya-lele.html` → **301 to the root `/`** (soft-404 risk)
- `timeiswhatyoumakeofit.html`, `accessories.html` → 301 to `swatchHome.html`
- `queens-jubilee.html` → **307** (temporary) to a product; `stores-bioceramic-moonshine.html` → 307
- `plp-*.html` (480 URLs) → 302 to `/watches/...` categories; `colorful-selection.html` → 302
- `bioceramic-moonswatch/moonshine-gold-so33m102-11x.html` (52) → 301 to new path
- `megastore-paris.html` (it-ch) → 404
- A French contest page (`jeu-concours-fr.html`) is published in every locale, including pl-pl

About 1,200 URLs have a `lastmod` before 2024, the oldest from 2020-09. Those are the first to review.
**Fix:** list only URLs that return 200. Change permanent moves from 302/307 to 301. Retire pages with no real replacement as 410 instead of redirecting them to the homepage.

### 6. hreflang (page HTML) doesn't match the sitemap's locale set (medium–high)
The sitemaps carry **no `xhtml:link` hreflang** (0 annotations), so the page tags are the only source. Those tags don't match what the sitemap publishes:
- **hu-hu is missing from the cluster.** en-us, es-en and en-en product pages list 46 alternates without hu-hu. The hu-hu page lists itself, so the tags are **not reciprocal**, and Google may ignore the hu-hu alternate. (The fr-fr category page does include hu-hu, so this looks product-template specific.)
- **es-en (937 URLs) is not in any cluster.** It's published and self-canonical, but no page names it as an alternate. Its locale code is also odd: Spanish content for an "en" region.
- **en-ae and en-za** are live (200) and in every hreflang cluster, but **have no sitemap URLs at all**.
- `x-default` → en-en (international), which is reasonable.

**Fix:** generate hreflang from one locale list shared by the templates and the sitemap. Add hu-hu to product-page clusters, decide on es-en (add it with a valid code, or noindex it and drop it from the sitemap), and add en-ae/en-za to the sitemaps. Optionally put hreflang in the sitemap instead of the HTML to lighten pages.

### 7. Duplicate homepages (medium)
Each locale lists both `/xx-xx/` and `/xx-xx/swatchHome.html` (92 URLs). Both return 200. `/en-us/` has a canonical, but `swatchHome.html` has **no canonical tag**.
**Fix:** list only `/xx-xx/` and canonicalise or 301 `swatchHome.html` to it. Do the same for `flikflakHome.html`, which is a redirect target too.

### 8. `lastmod` and hint fields (low)
18,882 URLs (49%) have a November 2026 `lastmod`, and ~3,600 have October. Many look bulk-touched, not real content changes. Google ignores `lastmod` once it proves unreliable, so set it only on real content or price/stock changes. `changefreq`/`priority` are ignored by Google; there's also one `priority 0.0` block (26 URLs) worth looking at.

### 9. Minor
- Product canonicals are **relative** (`href="/en-us/golden-tac-gb274/GB274.html"`). This is valid, but absolute URLs are safer.
- No image sitemap entries. Adding `image:image` for product shots would help Google Images for a visual product range.
- The "Swatch Pay" variants (`-5300` codes, 782 URLs) are separate indexable products. Check that they're meant to compete with the base watch pages.

## Suggested sitemap generation rules (Salesforce B2C / Demandware)
1. Include only pages that return 200, are self-canonical, and are indexable.
2. Exclude: `/[a-z]{2}\d{3}/` shortcuts, FlikFlak pages/products (they belong to flikflak.com), names containing `test`/`QA`, `swatchHome.html`/`flikflakHome.html`, and any URL with `%3A//` in the path.
3. Drive locales and hreflang from one config, and add en-ae/en-za.
4. Recheck monthly: crawl every sitemap URL and alert when the share of non-200s goes over 1%.

## Method notes and limits
- Status codes came from 182 URLs: 25 random per pattern bucket, plus hand-picked checks. Pattern counts cover all 38,227 URLs. The ~25% non-200 figure is an **estimate** from the two combined, not a full crawl.
- The flikflak.com end of the redirect chains wasn't fetched (outside this environment's network allowlist).
- Pages were fetched without JavaScript, so tags injected client-side weren't seen.
