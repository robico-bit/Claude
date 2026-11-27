# swatch.com XML sitemap audit: recheck after fixes

**Checked:** 2026-11-27, after the sitemap changes made the same day. It's compared with the first audit of 2026-11-26. I used the same method: live fetch of the index, all 12 child sitemaps and robots.txt, a stratified sample of 150 URLs (plus the known loop URL) checked for status code (25 random per URL group, same groups as before), and canonical/hreflang checks on 6 pages.

## Summary

**Much better.** The sitemap went from 38,227 to **24,699 URLs (−35%)**, and almost all the noise is gone from the 11 regional sitemaps. **Every sampled URL from those files returned 200.** The new `meae` sitemap adds en-ae and en-za, which closes a gap from the first audit.

What's left is almost all in **one file, `sitemap_0.xml` (US/Canada, en-us/en-ca/fr-ca)**, which wasn't cleaned. All 69 non-200 URLs in today's sample came from that file. I estimate **about 600–700 of its 2,321 URLs** still redirect or 404, which is **roughly 3% of the whole sitemap, down from about 25%**.

Two follow-ups: `lastmod` has been removed from the regional files, and the hreflang problems on the pages haven't changed (they're outside the sitemap).

## Before vs after

| Check | 26 Nov | 27 Nov | Status |
|---|---|---|---|
| Total URLs | 38,227 | 24,699 | Improved |
| Child sitemaps | 11 | 12 (new `meae`) | Improved |
| Legacy `/swNNN/` shortcuts | 4,274 | 279 (all US/CA) | Mostly fixed |
| Test / QA pages (404) | 116 | 0 | Fixed |
| URLs with "flikflak" in path | 3,311 | 192 (all US/CA) | Mostly fixed |
| FlikFlak product codes | 2,560 | 165 (all US/CA) | Mostly fixed |
| Old moonshine-gold URLs | 52 | 0 | Fixed |
| `plp-*` legacy pages | 480 | 33 (all US/CA) | Mostly fixed |
| Campaign / seasonal URLs | 1,020 | 228 | Mostly fixed |
| Duplicate `swatchHome.html` | 92 | 6 (all US/CA) | Mostly fixed |
| Malformed ja-jp self-redirect URL | in sitemap | removed | Fixed in sitemap (URL still loops) |
| en-ae / en-za sitemaps | none | 502 URLs each | Fixed |
| Sampled non-200 (stratified sample) | 118 of 175 | 69 of 150, **all from sitemap_0** | Regional files clean |
| URLs with `lastmod` | 100% | 9% (sitemap_0 only) | Needs follow-up |
| hreflang on pages (hu-hu, es-en) | inconsistent | unchanged | Open |
| URLs blocked by robots.txt | 0 | 0 | OK |

## What's left to do

### 1. Clean `sitemap_0.xml` (US/Canada) the same way (high)
This standard Salesforce file wasn't regenerated. It still has 2,321 URLs with the same legacy patterns, and every non-200 in today's sample came from it:
- **279 `/swNNN/` and `/seNNN/` shortcuts.** 21 of 25 sampled returned 301, many to robots-blocked filter URLs (e.g. `/en-us/sw102/` → `/en-us/watches/?prefn1=catalogLine&prefv1=Lady`). The `/seNNN/` ones return 200.
- **FlikFlak pages and products (~350).** Older seasonal pages (`flikflak-halloween/`, `flikflak-mothersday/`) 301 to `/all-kids-watches/` (21 of 25 sampled). Collection pages and FBNP products **302 through the session redirect to flikflak.com** (5 of 25 folders sampled, plus all 29 FlikFlak product URLs in a partial full check).
- **Old campaign/legacy `.html` pages:** 24 of 27 sampled weren't 200.
  - `queens-jubilee.html`, `stores-bioceramic-moonshine.html` and `fathers-day.html` → **307**
  - `timeiswhatyoumakeofit.html` and `accessories.html` → 301 to `swatchHome.html`
  - `best-sellers.html` → 301 to a `/search?cgid=` URL
  - `canvasShop.html` → **404**
- 6 `swatchHome.html` homepage duplicates (still no canonical tag).

**Fix:** apply the same exclusion rules used for the regional files to the US/CA site, or switch it to the same custom generator.

### 2. Bring back an accurate `lastmod` (medium)
The regional files now contain only `<loc>`: 22,378 URLs (91%) have no `lastmod`, `changefreq` or `priority`. Dropping the unreliable bulk dates was reasonable, and `changefreq`/`priority` aren't missed (Google ignores them). But Google does use an **accurate** `lastmod` to decide what to re-crawl, which matters for price, stock and new launches.
**Fix:** add `lastmod` back using the real last-modified date of the product or content (not the generation date).

### 3. hreflang on the pages (unchanged, medium–high)
This is outside the sitemap, so it's expected to be unchanged:
- **hu-hu** is still missing from the alternates on product and homepage templates in other locales (e.g. en-us, es-en). The hu-hu page lists itself, so the tags aren't reciprocal.
- **es-en** (626 URLs in the sitemap) is still not named as an alternate on any page.
- Good news: en-ae and en-za are now in both the hreflang tags and the sitemap.

**Fix:** generate hreflang from the same locale list as the sitemap.

### 4. Smaller follow-ups (low)
- The malformed ja-jp URL (`/ja-jp/https%3A//www.swatch.com/ja-jp/home/swatchHome.html`) is out of the sitemap, but it still 301s to itself. Make it return 404/410 and find the link that created it.
- Product canonicals are still relative. Absolute URLs are safer.
- No image entries yet.
- The 816 Swatch Pay variant products (`-5300`) are still listed as separate indexable pages. Check that this is intended.
- The new regional files use single quotes in the XML declaration and are minified onto one line. That's valid, and fine.

## Method notes and limits
- Today's status figures come from the same stratified sample design (25 per URL group) as the first audit, so the two are directly comparable. A full check of every URL wasn't possible: the site answered slowly (about 2–4 seconds per request), so I stopped the full run after 29 URLs. All 29 were FlikFlak product URLs that 302 to flikflak.com.
- The ~600–700 estimate for `sitemap_0.xml` combines the pattern counts with the sampled failure rates per group.
- Requests came from a US-located IP, without JavaScript rendering.
