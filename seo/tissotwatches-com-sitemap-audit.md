# tissotwatches.com XML sitemap audit

**Checked:** 2026-11-26. I fetched robots.txt and the sitemap index of each of the 41 locales live, then parsed all 188 child sitemaps from the 4 regional indexes that exist. I checked status codes on about 400 sampled URLs and canonical/hreflang on product, category and article pages.

## Summary

The sitemap setup is **duplicated and partly broken**. Each region publishes two overlapping sitemap sets: Salesforce's standard `sitemap_N.xml` and a custom set (product, product-images, category, content, favicon). Together they hold 75,652 entries for only 30,945 unique URLs. The standard set still lists **about 3,650 discontinued products that return 404/410**. The custom content set lists **magazine and legal pages that return 500 server errors**. Six live locales have **no sitemap at all**. The only sitemap declared in robots.txt **redirects**. On the pages, the hreflang tags use **invalid codes and broken x-default markup**.

Product pages that appear in both sets are healthy: 40 of 40 sampled returned 200, are self-canonical, and are `index, follow`.

## Structure

| Item | Result |
|---|---|
| robots.txt | One line: `Sitemap:https://www.tissotwatches.com/en-en/sitemap_index.xml` |
| Declared index | **302** → `/en-us/sitemap_index.xml` (geo-based; `/en-en/` 302s to `/en-us/` for US visitors, and Googlebot crawls mostly from the US) |
| Regional indexes | Europe (98 children, served identically under 18 EU locales), APAC (68, 13 locales), North America (16: en-us, en-ca, fr-ca), Mexico (6: es-mx) |
| No sitemap | **el-gr, en-za, en-sa, pt-br, pt-pt, tr-tr**. Their `sitemap_index.xml` 302s to en-us, yet the pages are live (200) and in every hreflang cluster |
| Files | 188 child sitemaps, all 200; largest 5,000 URLs / 1.1 MB (within limits) |
| Entries | 75,652 entries → 30,945 unique URLs across 34 locales (~720–1,140 each) |
| Composition | ~23,800 product URLs, ~6,700 other `.html`, ~500 extensionless content URLs |
| hreflang in sitemaps | None |
| Images | 61,400 `image:image` entries (2,622 unique images), all **300 px thumbnails** |
| robots conflicts | No sitemap URL is blocked by robots.txt |

## Issues, by priority

### 1. ~3,650 discontinued products (404/410) in the standard sitemaps (high)
Of 22,058 product URLs in the standard `sitemap_N.xml` files, 18,405 also appear in the custom product sitemaps, and all 40 sampled returned 200. The other **3,653 appear only in the standard set**, and all 40 sampled returned **410 (30) or 404 (10)**. Examples: `/fr-fr/T0064282203801.html` (410), `/zht-tw/T8624101904200.html` (404). The standard file also gives them a fresh `lastmod`, so Google keeps re-crawling dead pages.
**Fix:** drop the standard `sitemap_N.xml` from the index and keep the custom product sitemaps. If it stays, exclude offline and discontinued products from the job.

### 2. Content pages that return 500 (high)
- **Magazine articles:** 2,133 `magazine-news-*` URLs. 20 of 25 sampled from the content-only bucket returned **500**, and 5 of 40 across all magazine URLs. Examples: `/en-gb/magazine-news-tissot-boutique-amsterdam.html` and `/de-ch/magazine-news-tony-parker-2024.html`, which return 500 on repeated, slow requests.
- **Extensionless legal and service URLs** (331, e.g. `/en-us/terms-of-use`, `/en-no/delivery-returns`): 19 of 30 returned **500** and 6 returned **404**.
- **132 raw pipeline URLs** (`/on/demandware.store/Sites-APAC-Site/en_AU/Search-ShowContent?fdid=cookie-notice`) in the APAC content sitemaps: **25 of 25 returned 500**. These URLs should never be in a sitemap.

Repeated 5xx responses make Google crawl the site more slowly and eventually drop the pages.
**Fix:** fix the server error on these content assets first. It looks like a broken template or controller for content-asset pages. Then list only the SEO-friendly URL (`/xx-xx/page.html`) for each content asset, and never `demandware.store` URLs.

### 3. The declared sitemap redirects, and some regions are undeclared (high)
robots.txt declares only `/en-en/sitemap_index.xml`. For US visitors it **302s to the en-us index**, which covers only US/Canada. The EU, APAC and Mexico indexes are **not declared anywhere public**. They may be submitted in Search Console, but other search engines (Bing and others) can't find them.
**Fix:** list every regional index in robots.txt with direct 200 URLs (e.g. `/fr-ch/sitemap_index.xml`, `/en-au/sitemap_index.xml`, `/en-us/sitemap_index.xml`, `/es-mx/sitemap_index.xml`), or publish one global index that is exempt from geo-redirects. Make sure Googlebot is never geo-redirected.

### 4. Six live locales without a sitemap (medium–high)
**el-gr, en-za, en-sa, pt-br, pt-pt, tr-tr** serve live pages (e.g. `/el-gr/men.html` returns 200) and are in every hreflang cluster. Their sitemap index redirects to en-us, so none of their URLs are submitted.
**Fix:** generate sitemaps for these sites (they appear to run on a separate setup), or confirm they are deliberately left out.

### 5. Duplicate listing across sitemap sets (medium)
25,630 URLs appear in more than one file: products in the standard + product + product-images sitemaps, and categories and content in both standard and custom files. The two sets also disagree on `lastmod`: 20,353 custom entries have **no lastmod**. The rest are mostly bulk-dated November 2026.
**Fix:** use one sitemap per URL type, put image data in the product sitemap itself, and set `lastmod` from real content changes.

### 6. hreflang errors on the pages (high for international SEO)
The sitemaps carry no hreflang, so the page tags are the only signal, and they have errors:
- **Broken x-default:** `<link rel="x-default" href="…/en-en/…" hreflang="en-en"/>` is invalid. It must be `rel="alternate" hreflang="x-default"`. As written, Google sees no x-default.
- **Invalid language codes:** `zht-hk`, `zht-mo` and `zht-tw` aren't valid ISO 639-1 codes (use `zh-HK`/`zh-Hant-HK`, `zh-MO`, `zh-TW`), so the Traditional Chinese pages drop out of the clusters. `en-en` isn't a valid region either.
- **`hreflang="null"`** on the homepages for en-sa.
- **ru-ru alternate points to the en-au URL** (`/en-au/men.html` tagged `ru-ru`), while the real `/ru-ru/` paths 301 to `/en-en`. The APAC index also contains 5 `ru_RU` sitemaps that list **en-au URLs only** (474 entries), a locale fallback leak.
- The **ar-sa** alternate (`/ar-sa/men.html`) returns **404**.
- Product pages list 41 alternates and category pages 43, so the clusters aren't consistent.

**Fix:** build hreflang from one locale table with valid codes. Use `rel="alternate" hreflang="x-default"` and drop ru-ru/ar-sa until they exist. The same table should drive the sitemaps.

### 7. Favicon "sitemaps" (low, quick win)
36 files (`sitemap-custom-sitemap_xx_XX-favicon.xml`) each list one URL with **no scheme**, e.g. `www.tissotwatches.com/on/demandware.static/…/favicon.ico`. That's invalid in a sitemap and not a page.
**Fix:** remove these files from the indexes.

### 8. Image sitemap quality (low–medium)
Every image URL is a 300×345 thumbnail (`?sm=cut&sw=300&sh=345,gravity=center`). The `sh=345,gravity=center` parameter also looks malformed (a comma instead of `&`). 180 images have no caption.
**Fix:** point `image:loc` at a large rendition (≥1200 px) with a clean URL so product shots can rank in Google Images.

### 9. Minor
- 35 URLs with uppercase or legacy names, e.g. `/en-us/Garantie.html` (404) and `…-Jungfrau.html`.
- `changefreq`/`priority` are ignored by Google, and the two sets contradict each other anyway (products `daily 0.5` vs `yearly 0.3`).
- `Disallow: /*start` and `/*search*` are broad prefix wildcards. They block any path containing those words, e.g. a future `/start-here.html`. They're harmless today, but worth tightening.

## Suggested target setup (Salesforce B2C)
1. One sitemap job per site, outputting only indexable URLs that return 200, are self-canonical and are online.
2. Remove the standard `sitemap_N.xml` (or filter it), the favicon files and the ru_RU fallback files.
3. No `demandware.store` URLs; content assets only by their SEO URL.
4. Declare every regional index in robots.txt, with 200 URLs and no geo-redirect for bots.
5. Add sitemaps for el-gr, en-za, en-sa, pt-br, pt-pt and tr-tr.
6. Fix the hreflang codes and the x-default markup.
7. Monitor monthly: crawl every sitemap URL and alert when the share of non-200s goes over 1%.

## Method notes and limits
- Status codes come from stratified random samples (25–40 per URL bucket, ~400 requests in total). Counts like "3,653 dead products" are the full size of a bucket in which every sampled URL failed, not a full crawl.
- Requests came from a US-located IP. Geo-redirect behaviour from other countries wasn't tested, and I didn't send a spoofed Googlebot user agent.
- The 500s reproduced on slow, repeated requests, but they could be specific to non-browser clients. Check them in Search Console's crawl stats or with the URL Inspection tool.
