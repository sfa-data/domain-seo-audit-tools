# site-seo-audit

Store page: https://apify.com/sfa-data/site-seo-audit

**Free broken link checker, redirect and HTTP status code checker, sitemap checker and bulk Lighthouse test for
up to 100 websites in one run.** Enter a website, click Start, get one table with every problem and the page it
is on. The actor is free; you only pay Apify's compute time in your own account.

- **Broken link checker:** broken internal links (404, 410, unknown host, redirect loops) with up to five pages that link to each one.
- **Redirect checker:** HTTP status and the full redirect chain of every page.
- **Sitemap and robots.txt checker:** missing or invalid sitemaps, sitemap URLs that redirect, break, are noindex or canonicalised elsewhere.
- **Meta tag checker data:** title, title and meta description length, H1 count, canonical, noindex and `lang` per page.
- **Google Lighthouse scores** (the lab test behind PageSpeed Insights): performance, accessibility, best practices, SEO.

**Quick start:** the input is prefilled with `https://example.com`, a one-page test site, so the first run is short.
Then enter your own website and click **Start**; open the **Overview** table (or **Page SEO**) and export as
CSV, Excel or JSON. Excerpt from a real run (2026-10-09, python.org and example.com, 10 pages and 10 links each):

| type | url | verdict | what it shows |
|---|---|---|---|
| `page` | `https://python.org/` | `redirect` | 301 to `https://www.python.org/`, title 21 characters, 5 H1 headings |
| `link` | `https://www.python.org/download/alternatives` | `redirect` | 301 to the URL with a trailing slash, linked from 10 pages |
| `lighthouse` | `https://example.com/` | | performance 100, accessibility 100, best practices 100, SEO 80 |
| `summary` | `python.org` | | 16 ok, 4 redirect, 0 broken; finding `SITEMAP_MISSING` |

Google Lighthouse (Apache-2.0) runs inside the actor; you do not need a Google API key.

**For AI agents:** input is `{"startUrls": ["https://example.com"]}` (1–100 websites per run). Every output row
has a `type`: `page`, `link`, `lighthouse`, `summary` or `error`. Read the `summary` row of each website first
(`findings`, `counts`). The actor is free; you only pay Apify's compute time in your own account. Lighthouse is
optional (`lighthousePagesPerSite`, default 1; 0 turns it off) and needs more compute time than the plain checks.

## What does Site Audit do?

Per website, in this order:

1. **robots.txt**: fetched and obeyed for every request. If it cannot be read (unknown domain, timeout, 5xx or
   429), nothing else on the website is requested; the summary shows `ROBOTS_UNREACHABLE` and no page rows.
2. **Start page**: checked first. If it has no definite answer (for example timeout, bot protection, 401/403
   or robots.txt disallow), the website ends here with `start_unreachable`. If it redirects to another host,
   the website is not audited (`offsite_redirect`); enter the final host instead.
3. **Sitemaps**: the sitemaps named in robots.txt, or `/sitemap.xml` and `/sitemap_index.xml`. Sitemap
   index files and gzip are supported. Without a sitemap the actor crawls links from the HTML, up to three
   clicks deep.
4. **Pages**: each page gets its HTTP status, a verdict, the full redirect chain (up to 10 hops), and SEO
   fields: title, title length, meta description length, canonical, noindex (meta tag or `X-Robots-Tag`),
   number of H1 headings, `lang`, whether it is in the sitemap, and the number of outgoing links.
5. **Internal links**: link targets on the website that were not already checked as pages, with up to five
   pages that link to each one.
6. **Lighthouse**: performance, accessibility, best practices and SEO scores (0–100), lab metrics (FCP,
   LCP, TBT, CLS, Speed Index, TTFB), and the ten heaviest failed audits.
7. **Summary**: counts per verdict and the findings below.

### Verdicts

| Verdict | Meaning |
|---|---|
| `ok` | 2xx without redirect |
| `redirect` | at least one redirect, ending in 2xx (or a redirect to another host, which is not followed) |
| `broken` | any other non-2xx status except 401/403/429 and 5xx (for example 404, 410), unknown domain, redirect loop or more than 10 redirects |
| `server_error` | 5xx after one retry |
| `restricted` | the server refused us: `bot_protection`, `rate_limited`, `auth_required`, `forbidden` (see `error`) |
| `unchecked` | no definite answer, for example `robots_disallowed`, `timeout` or `budget_exhausted` (see `error`) |

### Findings (in `summary.findings`, each with a count)

`ROBOTS_MISSING`, `ROBOTS_UNREACHABLE`, `SITEMAP_MISSING`, `SITEMAP_ERROR`, `SITEMAP_URL_NOT_OK`,
`SITEMAP_URL_REDIRECTS`, `SITEMAP_URL_NOINDEX`, `SITEMAP_URL_CANONICAL_ELSEWHERE`, `BROKEN_INTERNAL_LINKS`,
`LIMIT_REACHED` (a run limit or the run time stopped the check) and `BUDGET_EXHAUSTED` (the
website's time budget ran out; the count says how many known URLs and Lighthouse targets were left out; if the budget
stopped sitemap reading, it is a lower bound, at least 1, as unread files have no known URL count). Findings
cover sitemaps, robots.txt and broken links only; the page SEO fields are raw data for your own rules, the
actor does not grade them.

If one of your own limits (`maxPagesPerSite`, `maxLinksPerSite`) cut the check short, `truncatedBy` names it;
that is not a finding. Three safety caps can appear there too: `pageLinkCap` (a page had more than 2,000 links;
only the first 2,000 unique ones are read, and at most 20,000 distinct hrefs are looked at), `siteLinkCap` (more
than 50,000 link targets on one website) and `sitemapFileCap` (sitemap files left unread: at most 50 files per
website, and an index two levels below the first one is not followed). URLs longer than
8,192 characters are out of scope.

A 404 or 410 for `/sitemap.xml`, a redirect to another host or an HTML page in its place counts as no sitemap
(`SITEMAP_MISSING`), not as `SITEMAP_ERROR`. If the time budget ended sitemap reading or robots.txt did not
allow the default locations, the actor does not claim the sitemap is missing. Plain-text sitemaps (one URL per line) are not supported yet.
A sitemap with a tag or text longer than 64 KiB counts as invalid XML (`SITEMAP_ERROR`).

## Who is it for?

- **SEO agencies and freelancers** who audit client websites and need broken links, redirect chains and
  sitemap problems in one table they can export.
- **Website owners and web developers** who want to check a site after a relaunch, a migration or a CMS
  update.
- **Teams that monitor many sites**, for example a group of country websites, with a scheduled run per week.
- **AI agents** that need a cheap, structured technical check of a website with clear, documented fields.

## How to use it

1. Enter one or more websites in **Websites**, for example `https://example.com` or just `example.com`.
2. Optional: change how many pages and links to check per website, and how many pages Lighthouse should
   measure (0 turns Lighthouse off). Set a run timeout in the run options if you want a hard limit.
3. Click **Start**. Open the **Overview** table, sort by `verdict` or `type`, or export the results as CSV,
   JSON, Excel or HTML. The **Page SEO** table lists title and meta description length, H1 count, canonical,
   noindex and language per page.

## Input

| Field | Default | Meaning |
|---|---|---|
| `startUrls` | — | Start URLs or host names, one per website, 1–100 per run after clean-up. `https://` is added when missing, duplicates of the same host are dropped. `example.com` and `www.example.com` count as two websites, so enter only one. Invalid entries get an `error` row. |
| `maxPagesPerSite` | 200 | Pages per website (1–5,000), from the sitemap or, without one, by crawling. |
| `checkLinks` | `true` | Also check internal link targets that were not already checked as pages. |
| `maxLinksPerSite` | 500 | Unique internal link targets per website (0–5,000). |
| `lighthousePagesPerSite` | 1 | Lighthouse audits per website (0–50), start page first; 0 turns Lighthouse off. |

**Two websites, small sample** (this is the input of the output example below):

```json
{
  "startUrls": ["https://example.com", "python.org", "ftp://example.org"],
  "maxPagesPerSite": 10,
  "maxLinksPerSite": 10,
  "lighthousePagesPerSite": 1
}
```

**Full audit of one website** (up to 5,000 pages and 5,000 links, Lighthouse on 10 pages; this takes much longer than the first example):

```json
{
  "startUrls": ["https://www.example.com"],
  "maxPagesPerSite": 5000,
  "maxLinksPerSite": 5000,
  "lighthousePagesPerSite": 10
}
```

**Status check of many websites without Lighthouse** (you can set the memory to 1 GB in the run options; the
default is 4 GB):

```json
{
  "startUrls": ["example.com", "example.org", "example.net", "example.co.uk"],
  "maxPagesPerSite": 100,
  "checkLinks": false,
  "lighthousePagesPerSite": 0
}
```

## Output

Every dataset row has a `type`. Only the fields below exist.

- **`page`** — one checked page: `site`, `url`, `source` (`start`, `sitemap`, `crawl`), `status`, `verdict`,
  `finalUrl`, `redirects` (each hop with URL and status), `contentType`, and raw SEO fields for HTML pages:
  `title` (up to 300 characters), `titleLength` (of the stored title: 300 means the title may have been cut),
  `metaDescriptionLength`, `canonical`, `noindex` (meta tag or `X-Robots-Tag`), `h1Count`, `lang`, `inSitemap`,
  `outLinks` (unique http(s) link targets, internal and external, at most 2,000 per page; links to non-standard ports, IP addresses or
  with credentials are not counted); plus `checkedAt` and `error`. `status` is the status of the last answer.
- **`link`** — one internal link target that was not checked as a page: `site`, `url`, `status`, `verdict`,
  `finalUrl`, `redirects`, `foundOn` (up to 5 pages linking to it), `foundOnCount`, `checkedAt`, `error`.
- **`lighthouse`** — one Lighthouse run: `site`, `url`, `scores` (performance, accessibility, best-practices, seo; 0–100),
  `metrics` (`fcpMs`, `lcpMs`, `tbtMs`, `cls`, `speedIndexMs`, `ttfbMs`), `opportunities` (up to 10 failed audits
  with id, title, score and display value, heaviest first), `lighthouseVersion`, `benchmarkIndex`, `warnings`
  (count only), `checkedAt`, `error`.
- **`summary`** — one per website: `site`, `finalHost`, `robotsTxt` (`found`, `missing`, `error`; `null` if the website
  was not started), `crawlDelay`, `sitemaps` (files found, URLs, errors, URLs on other hosts; `null` if not read), `discovery` (`sitemap` or `crawl`), `pagesChecked`,
  `linksChecked`, `lighthouseRuns`, `counts` per verdict, `truncatedBy`, `findings`, `checkedAt`, `error`
  (`start_unreachable`, `offsite_redirect`, `budget_exhausted` or `internal`: an unexpected error ended this
  website, the run went on with the next). `pagesChecked` and `linksChecked` count every row of the website,
  `unchecked` and `restricted` ones included; use `counts` for the split by verdict.
- **`error`** — an input entry that is not a usable website: `input`, `error` (e.g. `invalid_url`), `checkedAt`.

The rows below come from a real run of the first input example on 2026-10-09 (26 rows in total).

**`page`** (the start page of python.org, which redirects to `www`):

```json
{
  "type": "page",
  "site": "python.org",
  "url": "https://python.org/",
  "source": "start",
  "status": 200,
  "verdict": "redirect",
  "finalUrl": "https://www.python.org/",
  "redirects": [{"url": "https://python.org/", "status": 301}],
  "contentType": "text/html",
  "title": "Welcome to Python.org",
  "titleLength": 21,
  "metaDescriptionLength": 52,
  "canonical": null,
  "noindex": false,
  "h1Count": 5,
  "lang": "en",
  "inSitemap": false,
  "outLinks": 127,
  "checkedAt": "2026-10-09T16:09:57Z",
  "error": null
}
```

**`link`** (an internal link that redirects; a broken one looks the same with `"verdict": "broken"` and
`"status": 404`):

```json
{
  "type": "link",
  "site": "python.org",
  "url": "https://www.python.org/download/alternatives",
  "status": 200,
  "verdict": "redirect",
  "finalUrl": "https://www.python.org/download/alternatives/",
  "redirects": [{"url": "https://www.python.org/download/alternatives", "status": 301}],
  "foundOn": [
    "https://python.org/",
    "https://www.python.org/psf/",
    "https://www.python.org/jobs/",
    "https://www.python.org/community/",
    "https://www.python.org/community/irc/"
  ],
  "foundOnCount": 10,
  "checkedAt": "2026-10-09T16:10:09Z",
  "error": null
}
```

**`lighthouse`** (example.com):

```json
{
  "type": "lighthouse",
  "site": "example.com",
  "url": "https://example.com/",
  "scores": {"performance": 100, "accessibility": 100, "best-practices": 100, "seo": 80},
  "metrics": {"fcpMs": 776, "lcpMs": 815, "tbtMs": 23, "cls": 0, "speedIndexMs": 776, "ttfbMs": 18},
  "opportunities": [
    {"id": "link-text", "title": "Links do not have descriptive text", "score": 0, "displayValue": "1 link found"},
    {"id": "meta-description", "title": "Document does not have a meta description", "score": 0, "displayValue": null},
    {"id": "network-dependency-tree-insight", "title": "Network dependency tree", "score": 0, "displayValue": null},
    {"id": "uses-long-cache-ttl", "title": "Serve static assets with an efficient cache policy", "score": 0.5, "displayValue": "1 resource found"}
  ],
  "lighthouseVersion": "12.8.2",
  "benchmarkIndex": 2341,
  "warnings": 0,
  "checkedAt": "2026-10-09T16:09:56Z",
  "error": null
}
```

**`summary`** (python.org: no sitemap, so the actor crawled; both of your limits cut the check):

```json
{
  "type": "summary",
  "site": "python.org",
  "finalHost": "www.python.org",
  "robotsTxt": "found",
  "crawlDelay": null,
  "sitemaps": {"found": 0, "urls": 0, "errors": 0, "foreignUrls": 0},
  "discovery": "crawl",
  "pagesChecked": 10,
  "linksChecked": 10,
  "lighthouseRuns": 1,
  "counts": {"ok": 16, "redirect": 4, "broken": 0, "server_error": 0, "restricted": 0, "unchecked": 0},
  "truncatedBy": ["maxPagesPerSite", "maxLinksPerSite"],
  "findings": [{"code": "SITEMAP_MISSING", "count": 1}],
  "checkedAt": "2026-10-09T16:10:28Z",
  "error": null
}
```

**`error`**:

```json
{"type": "error", "input": "ftp://example.org", "error": "invalid_url", "checkedAt": "2026-10-09T16:09:42Z"}
```

## How much does it cost?

The actor itself is free: there is no per-URL or per-audit fee. You only pay Apify's platform usage (compute
units, i.e. memory × run time) in your own Apify account, at the rates of your Apify plan; Apify's free plan
includes a monthly amount of platform usage.

The run time is what costs you. Plain checks (status, redirects, links, sitemaps) are light and run fine with
1 GB. Lighthouse starts a Chrome browser, needs at least 2 GB (the default is 4 GB) and takes about 15 to 40
seconds per audited page in local tests, so it uses much more compute time than the same number of plain URL
checks. Set `lighthousePagesPerSite` to 0 to skip it. Each website is also slowed to about two requests per second
and by its `Crawl-delay`, so large websites take a while. A measurement of the compute cost on Apify is still
to come. You can set a maximum run time (timeout) in the run options.

The run log may show "Ignored attempt to charge" lines; they are expected and cost nothing.
Pricing may be introduced later. Apify announces such changes to users 14 days in advance.

## Error codes

Errors never stop the run; they are stored in the row.

**`page` and `link` rows** (`error` explains the verdict):

| Verdict | `error` values |
|---|---|
| `broken` | `null` (the server answered 4xx; see `status`), `nxdomain` (the host name does not exist), `redirect_loop`, `too_many_redirects` |
| `restricted` | `bot_protection`, `rate_limited` (429), `auth_required` (401), `forbidden` (403) |
| `unchecked` | `robots_disallowed`, `robots_unreachable` (robots.txt could not be read, so nothing is requested), `timeout`, `network_error`, `connection_refused`, `tls_error`, `blocked_host` (the host failed three times in a row), `budget_exhausted`, `unsafe_address` (the host points to a private network), `invalid_url`, `unsupported_scheme`, `unsupported_port`, `head_unconfirmed`, `unreadable_body` (the server sent the page, or a Cloudflare 503, in a compression the actor does not read, such as Brotli, or cut it off before any content), `internal` |

**`lighthouse` rows**: `timeout` (more than 120 seconds), `out_of_memory`, `incomplete` (a category
score is missing), `lh_<code>` (an error reported by Lighthouse itself, for example `lh_no_fcp` when the page
paints nothing), `lh_failed`, `lh_missing`, `lh_interrupted` (platform restart), and the request checks
`blocked_host`, `unsafe_address`, `nxdomain`, `network_error`, `robots_disallowed`, `robots_unreachable`.

**`summary.error`**: `start_unreachable`, `offsite_redirect` (the start page redirects to another host, which
is not audited), `budget_exhausted` (the run's shared time budget was used up by earlier websites, so this one
was not started), `internal`.

**`error` rows**: `invalid_url` (not http(s), a non-standard port, an IP address, credentials, or not a valid
host name).

If the run itself cannot start, it fails before the first request: invalid input (for example an empty `startUrls` or a value out of range), more than 100 websites after
clean-up, Lighthouse with less than 2 GB of memory, or a setup error.

## FAQ

### Can I use it with the Apify API?

Yes. This call runs the actor and returns all rows when it is done:

```bash
curl -X POST "https://api.apify.com/v2/acts/sfa-data~site-seo-audit/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"startUrls": ["https://example.com"], "maxPagesPerSite": 50}'
```

The synchronous call waits at most 5 minutes. For bigger audits, start the run asynchronously and read the
dataset later. Clients for JavaScript and Python are available from Apify.

### Can AI agents use it through MCP?

Yes. Connect your agent (for example Claude, Cursor or another MCP client) to Apify's MCP server at
`https://mcp.apify.com`. The agent can find this actor with the `search-actors` tool, or you can add it
directly with `https://mcp.apify.com?tools=sfa-data/site-seo-audit`. A typical call for a quick check:

```json
{"startUrls": ["https://example.com"], "maxPagesPerSite": 50, "maxLinksPerSite": 100, "lighthousePagesPerSite": 1}
```

The input field descriptions state the limits of each option. The actor is free; the run time is what costs
compute units, and Lighthouse needs the most of it, so an agent that wants a cheap check sets
`lighthousePagesPerSite` to 0.

### Can I run it on a schedule and send the results somewhere?

Yes. Save your input as a task and add an Apify schedule, for example weekly. Apify integrations send the
results to Google Sheets, Slack, e-mail, Zapier, Make or a webhook after each run. Filter the rows on
`verdict` = `broken` to get only the broken pages and links.

### Is it legal? Does it collect personal data?

The actor only requests public pages. Page, link, robots.txt and sitemap requests identify as `SiteAuditBot`
in the user agent, obey robots.txt and `Crawl-delay`, and are limited to about two per second per host, so
website owners can allow or block the actor. A Lighthouse run loads the page like a mobile browser (with
Lighthouse's standard user agent and all page resources); the page URL itself must be allowed by robots.txt.
The actor does not bypass bot protection, logins or captchas. It stores no page content,
only the fields listed above, and it removes URL parts that look like e-mail addresses from all URLs in the
output. This helps you stay within the GDPR and other privacy laws. Only audit websites you are allowed to
check; this is not legal advice.

### How much memory does it need?

Lighthouse needs 2 GB; with `lighthousePagesPerSite` above 0 and less memory the run stops before the first
request. Without Lighthouse 1 GB is enough. The default is 4 GB; Apify gives runs with more
memory more CPU, so Lighthouse runs faster.

### What are the limits?

- **No JavaScript rendering while crawling:** links come from the server's HTML (`<a href>` only). Sites that
  build their links in the browser need a sitemap.
- **Only links within the website** are checked: the host of the start URL and its `www` variant. Links and
  redirects to other hosts are not requested.
- **Lab data, not field data:** Lighthouse runs in the actor's container with mobile emulation; scores differ
  from real-user (CrUX) data and from other Lighthouse runs. `benchmarkIndex` shows the machine speed.
- **Host block:** after three failures in a row (timeouts, network errors, refused connections, 429, bot
  protection) a host is skipped for the rest of the run (`blocked_host`).
- **Time budget per website:** each website gets a starting time budget and earns more with every definite
  check. Very slow websites can run out (`BUDGET_EXHAUSTED`). The run also has a shared time budget; if
  earlier websites used it up, later ones are not started (`summary.error` = `budget_exhausted`).
- **Run limits:** at most 100 websites, 20,000 page checks, 20,000 link checks and 250 Lighthouse runs per
  run. Runs stop at the run timeout (default 1 hour; raise it in the run options) and never run longer than
  4 hours (`LIMIT_REACHED`). Split larger jobs into several runs.

### What does it not do?

It does not grade on-page SEO (titles, descriptions, headings are raw data), check links to other websites,
render JavaScript, check keywords, rankings or backlinks, use Google field data (CrUX) or the PageSpeed API,
or log in to protected pages.

## Related actor

**[Bulk Domain Audit: SSL, WHOIS, DMARC & Noindex Checker](https://apify.com/sfa-data/domain-ssl-audit)** checks
the domain behind the website: registration and expiry date, DNS, SPF and DMARC mail security, the TLS
certificate and whether the homepage blocks search engines, with an A–F grade. Use both for a complete technical check of a client website.
