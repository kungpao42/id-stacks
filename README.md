# id-stacks

Generator and playbook for local-SEO **@ID stacks**: three linked pages that describe one local business, plus
schema on the client's own site that points at them.

```
client homepage schema   "@id": ID page URL
  -> S3 ID page           https://bigstack.s3.amazonaws.com/<slug>/id.html
       -> iframes GCS page   https://storage.googleapis.com/g-bigstacks/<Business-Name>.html
            -> iframes BatchGeo map   https://batchgeo.com/map/<id>
```

Reverse-engineered from the Priority 1 Plumbing stack
(https://slstacks.s3.amazonaws.com/priority1plumbing/id.html, money site priority1plumbing.com).

## Layout

```
build.mjs                     the generator
clients/_template/            copy this to start a new client
clients/<slug>/config.json    everything about one client
clients/<slug>/assets/        logo + hero image (source files)
clients/<slug>/homepage-schema.html   JSON-LD for the client's own homepage
dist/<slug>/                  upload to s3://bigstack/<slug>/
dist/<slug>-batchgeo/         BatchGeo paste data + the GCS wrapper page
dist/<slug>-citations-prompt.md   Gemini prompt for the citations Google Sheet
prompts/citations-sheet.md    template for that prompt ({{placeholders}} filled from config)
```

`dist/` is committed on purpose: it is the record of what is live.

## Build

```
npm install
node build.mjs <slug>
```

Writes `dist/<slug>/` (`id.html`, `sitemap.html`, `privacy-policy.html`, `img/logo.png`,
`img/<slug>-<imageDate>.jpg`) and `dist/<slug>-batchgeo/` (`batchgeo-paste-this.txt`, `batchgeo-title.txt`,
`batchgeo-description.txt`, `<Business-Name>.html`).

Set `imageDate` once, at the first build. The hero filename is built from it, so changing it renames a file
that is already live on S3.

## Infrastructure

| Piece | Where | Who uploads |
|---|---|---|
| ID page + images | S3 bucket `bigstack`, folder per client, public-read via bucket policy | HTML: Claude via the AWS MCP connector. Images: Chris by hand (see gotchas) |
| GCS wrapper | GCS bucket `g-bigstacks`, public | Chris (no GCS credentials in Claude's sandbox) |
| BatchGeo map | batchgeo.com, account cjwright@gmail.com, privacy **Public** | Claude in Chris's Brave after he logs in |

## Workflow for a new client

1. **Facts.** Pull the Google Business Profile (GMB Crush: profile, media, reviews) for NAP, hours, CID,
   category and photo media IDs. Read the money site's homepage and its current schema.
2. **Config.** `cp -r clients/_template clients/<slug>`, fill `config.json`, drop `logo` and `hero` into
   `assets/`. Social links in `sameAs` must match the money site's exactly. Copy is real facts only.
3. **Build** and look at `dist/<slug>/id.html` in a browser.
4. **Upload to S3.** HTML through the connector (PutObject, `ContentType: text/html; charset=utf-8`), then
   HeadObject and check the ETag equals the local md5. Images go up by hand.
5. **BatchGeo.** In batchgeo.com (logged in): load `batchgeo-paste-this.txt` (see gotchas), Set Options
   (Address = Address, Title = Keyword Title, URL = URL), Make Map, Save & Continue with
   `batchgeo-title.txt`, `batchgeo-description.txt`, Public. Note the `batchgeo.com/map/<id>` URL.
6. **Wire up.** Put that URL in `batchgeo.mapUrl`, rebuild. This fills the GCS wrapper and adds the kw5
   iframe to `id.html`. Re-upload `id.html`; Chris uploads the wrapper to `g-bigstacks`.
7. **Client homepage schema.** One LocalBusiness node of the most specific subtype, `"@id"` = ID page URL,
   `"url"` = money site, `image` = S3 hero, NAP matching the GBP exactly, hours, geo, `hasMap`, `sameAs`
   = socials + GBP. The ID page and GCS page do **not** go in `sameAs`. No self-serving `aggregateRating`.
   Save it as `clients/<slug>/homepage-schema.html`. If an SEO plugin emits Organization or Local Business
   schema, turn those off so there is one business node (SEOPress: Social Networks > Knowledge Graph = None;
   PRO > Local Business off; Titles & Metas > Home site name = business name).
8. **Citations sheet.** Paste `dist/<slug>-citations-prompt.md` into Gemini to build a Google Sheet of the
   business's citations and branded profiles (with NAP-match flags, so it doubles as a cleanup list). Per-client
   hints go in `config.citations` (category wording, region, extra sources, a note about a recent move). Publish
   the sheet (File > Share > Publish to web > Embed) and put its `src` in `citations.sheetUrl`; the build then adds a
   "Citations & Profiles" section (brand H2, no keyword) between kw7 and the Google map.
9. **Verify the chain** in a browser: homepage `@id` -> ID page with all images -> GCS page -> map with pins.
10. **Indexing.** Search Console URL-prefix properties for `https://bigstack.s3.amazonaws.com/<slug>/` and
   `https://storage.googleapis.com/g-bigstacks/`, HTML-file verification, then Request indexing for both
   pages (ID page first). Check with `site:` queries (DataForSEO, one task per live request).

## Gotchas

- **AWS MCP connector corrupts binary uploads.** Bytes get re-encoded (images arrive larger and broken).
  Only send text through it. For a small edit to a live page: GetObject, string-replace, PutObject, verify
  the md5 matches the local build.
- **BatchGeo ignores synthetic paste into its grid.** Set the hidden `#sourceData` textarea's value, then
  `jQuery('#sourceData').trigger('focus').trigger('change').trigger('blur')`; the grid re-renders.
- **BatchGeo save needs Chris's login** (the email already has an account). Claude can't type passwords;
  Chris logs in to batchgeo.com in Brave, Claude drives it with Claude in Chrome.
- **The built-in browser pane can be bound to a different computer** than the one Chris is looking at. Use
  Claude in Chrome (Brave) for anything needing his logins.
- **The cloud sandbox can't reach** S3, the client's site or Instagram directly; read them through a browser.
- The hero filename depends on `imageDate` (see Build).

## Rules

- No false structured data. The reference template lists "Best <keyword>" `award` entries the business never
  won; they're left out.
- No invented claims (24/7, "best", years in business you can't source).
- Video blocks and VideoObject schema only when the client has a real YouTube video (`youtubeId`).
- Ask before deleting anything from S3.

## Anatomy of the reference page

**Head**
- `<title>`, `meta title`, `DC.title` = "Business - Category City, State"
- `description`, `keywords` = the 7-keyword list; robots/googlebot/bingbot with `max-image-preview:large`
- canonical = the ID page itself; favicon = local logo
- JSON-LD #1 `LocalBusiness`: name, sameAs (socials), logo/image (local), url (ID page), description,
  telephone, address, `hasMap` (GBP CID link)
- JSON-LD #2 `WebPage`: significantLink/relatedLink/mainEntityOfPage = money site;
  `mainContentOfPage.about` (identifier/additionalType = money site, disambiguatingDescription = keywords);
  keywords/teaches/educationalUse/text = keyword list; citation sameAs socials;
  `hasPart` VideoObject (YouTube, locationCreated hasMap, publisher)
- JSON-LD #3 standalone `VideoObject` with `@id` = money-site#VideoObject

**Body: one keyword per heading, H1 down to H6**

| # | Heading | Keyword slot | Media |
|---|---|---|---|
| 1 | H1 | Business - Category City, State | — |
| 2 | H2 | kw1 (primary) + paragraph and brand-anchor link to money site, inside H3 | YouTube iframe (title = kw1) + **local** hero image (alt = kw1, title = H1 + name) |
| 3 | H2 | kw2 | text only |
| 4 | H4 ×2 | category entity + city entity | Wikipedia iframes |
| 5 | H3 | kw3 | iframe of money-site home |
| 6 | H3 | kw4 | **GBP photo** hotlinked `lh5.googleusercontent.com/p/<mediaId>` (alt = kw4), image left |
| 7 | H4 | kw5 | iframe of a second stack page (Google Cloud Storage) |
| 8 | H5 | kw6 | **GBP photo** (alt = kw6), image left |
| 9 | H6 | kw7 | **GBP photo** (alt = kw7), image right |
| 10 | — | — | Google Maps embed of the business |
| 11 | footer | NAP, City/State/Zip lines, GBP CID link, website (nofollow), money-site service links (nofollow), county map embed, sitemap + privacy links, socials | — |

Images: the logo and hero are hosted with the page; every other photo is the business's own Google Business
Profile photo, hotlinked by media ID, so the page carries the GBP entity's images. Alt text = the
section keyword.

## Deliberate differences from the reference
- No `award` property: the reference lists "Best <keyword>" awards the business never won. That's a false
  claim in structured data, so it's left out.
- Video blocks and VideoObject schema render only when `youtubeId` is set.
- The second-stack iframe (row 7) renders only when `secondStackUrl` is set.
- Copy is written per business from real facts (no "24/7", no invented claims).
- Kept from the reference on purpose: the intro and kw2 paragraphs sit inside `<h3>` (not valid HTML, but
  part of the template's heading weighting), styled back to body size.


## Second stack (GCS + BatchGeo)

The reference frames `storage.googleapis.com/<bucket>/<Business>.html` under its H4 (kw5). That file is a bare
wrapper: one text line, a BatchGeo iframe (BatchGeo's own embed code) and a full-screen link. The map carries
the content: title = kw5 + business name, a long description that opens with a link to the GBP and ends with
the website URL, and 15 rows (the business's real address first, then nearby places/neighborhoods labelled
with keyword variants, each linking to the GBP).

## Clients

| Client | ID page | Map | Built |
|---|---|---|---|
| La Sheida Spa | https://bigstack.s3.amazonaws.com/lasheidaspa/id.html | https://batchgeo.com/map/c5e3f9441a7dcd23182954e4c1d5a2fc | 2026-11-22 (citations sheet added same day) |
