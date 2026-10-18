# swatch.com: SEO review from search results only

**Date:** 2026-10-18
**Scope:** Limited review. This session's network policy blocks `swatch.com`, so **no page on the site was fetched**. Everything below comes from web search results: which swatch.com URLs show up, for which queries, and with which titles and snippets. The search tool returns results as seen from the US.

Read each finding as **a hypothesis to check**, not a confirmed defect. The "How to verify" column says what to look at once the site can be crawled (or in Google Search Console).

---

## Summary

| # | Finding | Severity (if confirmed) | Confidence |
|---|---------|------------------------|-----------|
| 1 | Pages for other countries show up for US searches instead of `/en-us/` pages | High | Medium |
| 2 | The same legal page is indexed under several URLs | Medium | High |
| 3 | Locale codes in URLs are inconsistent and some are not valid hreflang values | Medium | High |
| 4 | Snippets for product pages are mostly menu and country-selector text | Medium | Medium |
| 5 | swatch.com is missing from the top 10 for "MoonSwatch", its own flagship product | High | Medium |
| 6 | swatch.com is missing for "Swatch store near me"; spam pages hold those results | High | Medium |
| 7 | The site publishes `llms.txt` (good) | — | High |

---

## Findings

### 1. Pages for other countries show up for US searches
**Observed:** US searches for products returned country pages from elsewhere instead of the US store:
- "Swatch Big Bold Bioceramic watch" returned `/en-ie/`, `/da-dk/`, `/en-ca/`, `/es-en/` and `/sv-se/` product pages. No `/en-us/` page appeared.
- "swatch.com en-us" returned only `/en-cz/` product pages.
- "Swatch watches official site" returned `/en-ie/` product pages.

**Why it matters:** A US shopper who lands on the Irish or Czech store sees the wrong currency, the wrong shipping options and possibly a different catalogue. That hurts conversion, and it suggests Google isn't matching each locale to its market.

**Likely causes:** Missing or incomplete hreflang tags. Return links that don't match (page A points to page B, but B doesn't point back). Each locale's canonical tag pointing somewhere other than itself. Invalid language/region codes (see #3).

**How to verify:** Check the hreflang tags on one product (for example SB03N102) in every locale. Each locale should list all the others plus `x-default`, and each should name itself as canonical. In Search Console, look for hreflang errors under International Targeting, if the report is still available, or run a crawler's hreflang report.

### 2. The same legal page is indexed under several URLs
**Observed:** One search returned all of these as separate results:
- `https://www.swatch.com/en-us/legal/swatch-conditions-of-sale.html`
- `https://www.swatch.com/en-us/swatch-conditions-of-sale.html`
- `https://www.swatch.com/swatch-conditions-of-sale.html` (no locale prefix)
- `https://www.swatch.com/flikflak-conditions-of-sale.html` (Flik Flak brand, no locale prefix)

**Why it matters:** Ranking signals get split across copies of the same page. Google also spends crawl time on duplicates, and that cost grows across dozens of locales.

**Fix:** Pick one URL pattern (for example `/{locale}/legal/...`). Permanently redirect (301) the old paths to it, and make sure the kept URL names itself as canonical. Pages with no locale prefix should redirect based on the visitor's country or point to the `x-default` version.

### 3. Locale codes in URLs are inconsistent
**Observed:**
- Capitals in some URLs and not others: `/en-EN/press/collection` and `/en-en/explore-collections.html`.
- `en-en` / `en-EN` is not a real language-region pair, because `EN` is not a country code. If hreflang uses the same value, Google ignores it.
- `/es-en/` appears to mean "English for Spain", but it reads like "Spanish for England".
- A third-party directory links to `/en_gb` (underscore). This may be an old format.

**Why it matters:** Invalid hreflang values are silently ignored, which feeds #1. Paths that differ only by letter case can be indexed as duplicates.

**Fix:** In the hreflang tags, use `en` or `x-default` for the international site, and `en-ES`, `en-GB` and so on for regional English. URL paths can stay as they are, but they should be lowercase. Permanently redirect (301) any URL with capitals to its lowercase version, and permanently redirect old `_` paths too.

### 4. Snippets for product pages are mostly menus and country selectors
**Observed:** For the `/en-cz/` product pages, the text the search engine pulled was "mostly navigation menus and country selectors". Several swatch.com results also came back with the bare title "www.swatch.com" instead of a page title.

**Why it matters:** If search engines mostly see site-wide boilerplate, product pages look thin and similar to each other. The bare titles may mean the `<title>` or product description loads only through JavaScript, or comes after a large block of navigation in the HTML.

**How to verify:** Look at the raw HTML of a product page (not what the browser shows after scripts run). Check that the unique `<title>`, meta description, H1, product description and price are in that HTML, and that `Product` structured data is there, including `offers`, `brand`, `sku` and `aggregateRating` where applicable. Use Search Console's URL Inspection and its "View crawled page" to see what Googlebot gets.

### 5. swatch.com is missing from the top 10 for "MoonSwatch"
**Observed:** For "MoonSwatch", the top 10 was Wikipedia, Dezeen, Khaleej Times, watchesbysjx, Gear Patrol and Localiiz. There was no swatch.com URL. For "swatch.com en-us MoonSwatch collection", only the general `/en-us/explore-collections.html` page appeared. No MoonSwatch landing page showed up.

**Why it matters:** MoonSwatch is Swatch's most-searched product line. The brand should own the top result for its own name.

**Fix:** Build one strong MoonSwatch landing page per locale that can be crawled, with a clean URL like `/{locale}/moonswatch/` and stable content: the history, every model, availability (in store vs online), and FAQs. Link to it from the main navigation and the homepage. Add `CollectionPage` or `ItemList` structured data, plus `FAQPage` where it fits. Stop seasonal campaign pages from replacing it.

### 6. swatch.com is missing for "Swatch store near me"
**Observed:** All 9 results were spam pages on unrelated domains (`sso-test.faw.cymru`, `wpdocs.vantagecircle.com`, `blog.cityofeverett.com` and others). No official store locator appeared.

**Why it matters:** Searches for a nearby store come from people ready to buy. Swatch's empty slot has been filled by junk, which also carries a phishing risk for the brand.

**Fix:** Make the store locator crawlable, with a separate page for each store or city (for example `/{locale}/stores/{city}/{store}`). Each page needs the name, address, phone and opening hours in the HTML, plus `Store` / `LocalBusiness` structured data. Link the city and store pages together. Keep Google Business Profiles in sync with those pages. Consider reporting the spam domains.

### 7. The site publishes `llms.txt` (good)
**Observed:** `https://www.swatch.com/llms.txt` is indexed. It describes the brand (founded 1983 in Biel/Bienne, part of Swatch Group) and gives press and customer-care contacts. It is dated March 2026.

**Note:** Keep it current. Consider linking it to the main collections, including MoonSwatch, so AI assistants have authoritative pages to cite.

---

## Not covered (needs site access)

robots.txt, XML sitemaps, redirect chains, status codes, Core Web Vitals and page weight, JavaScript rendering, internal linking depth, faceted-navigation crawl traps, image alt text and formats, a structured-data audit, and mobile rendering.

When `swatch.com` and `www.swatch.com` are allowed in the environment's network settings, a full technical audit can run against the live site.

## Sources (search results used)
- https://www.swatch.com/llms.txt
- https://www.swatch.com/fr-fr/swatch-legal-notice.html
- https://www.swatch.com/en-ie/big-bold-bioceramic-arctic-sb03n102/SB03N102.html
- https://www.swatch.com/da-dk/big-bold-bioceramic-arctic-sb03n102/SB03N102.html
- https://www.swatch.com/en-ca/big-bold-bioceramic-arctic-sb03n102/SB03N102.html
- https://www.swatch.com/es-en/big-bold-bioceramic-arctic-sb03n102/SB03N102.html
- https://www.swatch.com/sv-se/big-bold-bioceramic-basalt-sb03b110/SB03B110.html
- https://www.swatch.com/en-ie/moonswatch-dark-blue-turquoise-rubber-acso33015m/ACSO33015M.html
- https://www.swatch.com/en-cz/blue-shore-gw217/GW217.html
- https://www.swatch.com/en-us/legal/swatch-conditions-of-sale.html
- https://www.swatch.com/en-us/swatch-conditions-of-sale.html
- https://www.swatch.com/swatch-conditions-of-sale.html
- https://www.swatch.com/flikflak-conditions-of-sale.html
- https://www.swatch.com/en-EN/press/collection
- https://www.swatch.com/en-en/explore-collections.html
- https://www.swatch.com/en-us/explore-collections.html
- https://en.wikipedia.org/wiki/MoonSwatch
- https://www.gearpatrol.com/watches/moonswatch-mission-to-earthphase-moonshine-gold
- https://sso-test.faw.cymru/tag/swatch/
- https://blog.cityofeverett.com/?p=1019138
