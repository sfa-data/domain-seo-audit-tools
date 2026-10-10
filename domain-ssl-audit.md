# domain-ssl-audit

Store page: https://apify.com/sfa-data/domain-ssl-audit

**Bulk SSL expiry checker, domain expiry (WHOIS/RDAP) lookup, SPF and DMARC checker, and noindex checker for up to
1,000 domains in one run.** Paste a list, click Start, get one row per domain with an A–F grade. The actor is free;
you only pay Apify's compute time in your own account.

- **SSL certificate expiry:** issuer, expiry date, days left, TLS version, trusted or not.
- **Domain expiry date:** registrar, created and expiry date where the registry publishes them (RDAP, or WHOIS for `.eu`, `.it`, `.at`).
- **Mail security:** SPF and DMARC checker, plus MX, NS, CAA and DNSSEC.
- **Noindex checker and robots.txt test:** is the homepage accidentally blocking Google or Bing?
- **Score 0–100 and grade A–F** with the reasons, exportable as CSV, Excel, JSON or HTML. No WHOIS contact data is stored.

**Quick start:** the input is prefilled with three domains. Click **Start**; a few seconds later the **Overview**
table shows one row per domain. Then paste your own list (URLs work too). Example from a real run (2026-10-10):

| domain | domain expires | SSL expires | days left | DMARC | Google/Bing | grade | findings |
|---|---|---|---|---|---|---|---|
| python.org | 2033-03-28 | 2027-02-14 | 127 | `p=none` | indexable | B (85) | `DMARC_NONE`, `NO_CAA`, `DNSSEC_OFF` |

**For AI agents:** input is `{"domains": ["example.com", ...]}` (1–1,000 per run). Output is one JSON row per
unique domain, plus one row per invalid entry (invalid entries first). Read `security.grade` and
`security.findings[].code` for the result; `null` means "not assessed", never "good".

## What does Bulk Domain Audit do?

For every domain you give it, the actor runs up to four checks:

| Check | What you get |
|---|---|
| `registration` | status (`registered`, `available`, `unknown`), registrar, created and expiry date, nameservers, DNSSEC, registry status codes. Source: the registry's RDAP service, or WHOIS for `.eu`, `.it` and `.at`. |
| `dns` | A, AAAA, MX and NS records, the SPF record, the DMARC policy (`v`, `p`, `sp`, `pct` only), CAA (`issue`, `issuewild` only) |
| `ssl` | certificate issuer, expiry date, days left, number of names on the certificate, TLS version, and whether standard clients trust it. The actor connects to port 443 on the domain; if that connection or handshake fails, it tries `www.`. If the domain has no IP address, no certificate is checked (`no_address`). |
| `indexing` | whether Google and Bing may crawl and index the homepage: robots.txt rules for `Googlebot`, `Bingbot` and `*`, the status after redirects, `noindex` in the `X-Robots-Tag` header or a meta robots tag. See [Search engine visibility](#search-engine-visibility-noindex-checker-and-robotstxt-test-indexing). |

From these facts it builds a fixed score: start at 100 and subtract points for each finding.

| Finding code | Points | Meaning |
|---|---|---|
| `CERT_EXPIRED` | 40 | The TLS certificate has expired. |
| `CERT_UNTRUSTED` | 30 | Standard clients do not trust the certificate. |
| `NO_HTTPS` | 30 | Port 443 refuses the connection or the TLS handshake fails. |
| `SEARCH_BLOCKED` | 30 | Google or Bing cannot index the homepage (`indexing.verdict` is `blocked`). |
| `CERT_EXPIRES_SOON` | 20 | The certificate expires within `sslWarnDays` (default 14). |
| `TLS_OUTDATED` | 20 | The server uses TLS below 1.2. |
| `DOMAIN_EXPIRES_SOON` | 20 | The domain registration expires within 30 days. |
| `NO_DMARC` | 15 | The domain receives mail (has MX) but has no DMARC record. |
| `NO_SPF` | 10 | The domain receives mail but has no SPF record. |
| `DMARC_NONE` | 5 | The domain receives mail and its DMARC policy is `p=none`: spoofed mail is only reported. |
| `NO_CAA` | 5 | No CAA record: any certificate authority may issue certificates. |
| `DNSSEC_OFF` | 5 | DNSSEC is not enabled. |

Grades: A = 90–100, B = 75–89, C = 60–74, D = 40–59, F = below 40. The score only uses the checks you chose.
A domain gets **no grade** (`security: null`) if a chosen check did not answer, if the certificate trust could
not be verified, if the domain does not resolve, or if it is not registered. A missing grade is never a good
grade. It also gets no grade if no chosen check produced anything to score: `registration` alone for an
unsupported ending (`registration.error` is `unsupported_tld`), or `ssl` alone for a domain without an IP
address (`ssl.error` is `no_address`). An `indexing` result of `unknown` (for example bot protection) costs no
points and does not remove the grade.

## Who is it for?

- **Web and SEO agencies** that look after many client websites and want one list of upcoming domain and
  certificate expiries.
- **IT and security teams** that keep an inventory of company domains, including old and parked ones, and
  want to see weak mail-security settings (SPF, DMARC) at a glance.
- **Domain portfolio owners** who need registration dates, registrars and nameservers in one table.
- **AI agents and scripts** that need structured, predictable domain facts with clear error codes.

## How to use it

1. Paste your domains into **Domains**, one per line. URLs are fine: `https://www.example.com/page` becomes
   `example.com`.
2. Optional: choose the **Checks** and the **Certificate warning (days)**.
3. Click **Start**. When the run finishes, open the **Overview** table or export the results as CSV, JSON,
   Excel or HTML.

## Input

| Field | Default | Meaning |
|---|---|---|
| `domains` | — | Domain names or URLs, up to 1,000 per run after clean-up. Schemes, paths, upper case and duplicates are removed. Subdomains are reduced to the registered domain (`shop.example.com` → `example.com`). Internationalized names (IDN) work. Invalid entries get an error row. |
| `checks` | all four | Any of `registration`, `dns`, `ssl`, `indexing`. |
| `sslWarnDays` | 14 | Flag certificates that expire within this many days (1–90). |

**Full check of a mixed list** (this is the input of the output example below):

```json
{
  "domains": ["python.org", "https://www.nic.io/", "bücher.de", "this-name-is-surely-free-7q3x9.com", "not a domain"]
}
```

**Weekly expiry watch** (domain and certificate dates only, earlier warning):

```json
{
  "domains": ["example.com", "example.org", "example.net"],
  "checks": ["registration", "ssl"],
  "sslWarnDays": 30
}
```

**Registration and mail-security inventory** (no TLS check):

```json
{
  "domains": ["example.com", "example.co.uk", "example.io"],
  "checks": ["registration", "dns"]
}
```

## Output

One row per invalid entry first, then one row per unique domain in input order. Entries that become the same
domain after clean-up (for example `example.com` and `https://www.example.com/`) share one row.
Every row has the same keys: `input`, `domain`, `tld`, `registration`, `dns`, `ssl`, `indexing`, `security`,
`checkedAt`, `error`. A check you did not choose is `null`. These rows come from a real run of the first input
example on 2026-10-09, before the `indexing` check existed; an example of that block follows in the next section.

**A checked domain** (`python.org`, grade B):

```json
{
  "input": "python.org",
  "domain": "python.org",
  "tld": "org",
  "registration": {
    "status": "registered",
    "source": "rdap",
    "registrar": "Gandi SAS",
    "created": "1995-03-27",
    "expires": "2033-03-28",
    "nameservers": ["ns-1134.awsdns-13.org", "ns-2046.awsdns-63.co.uk", "ns-484.awsdns-60.com", "ns-981.awsdns-58.net"],
    "dnssec": false,
    "statusCodes": ["client transfer prohibited"],
    "note": null
  },
  "dns": {
    "resolved": true,
    "a": ["151.101.0.223", "151.101.128.223", "151.101.192.223", "151.101.64.223"],
    "aaaa": ["2a04:4e42:200::223", "2a04:4e42:400::223", "2a04:4e42:600::223", "2a04:4e42::223"],
    "mx": ["50 mail.python.org."],
    "ns": ["ns-1134.awsdns-13.org.", "ns-2046.awsdns-63.co.uk.", "ns-484.awsdns-60.com.", "ns-981.awsdns-58.net."],
    "spf": "v=spf1 mx ip4:188.166.95.178/32 ip6:2a03:b0c0:2:d0::71:1 include:stspg-customer.com include:_spf.google.com include:mailgun.org ~all",
    "dmarc": "v=DMARC1; p=none; sp=none; pct=100",
    "caa": []
  },
  "ssl": {
    "reachable": true,
    "host": "apex",
    "issuer": "GlobalSign nv-sa",
    "notAfter": "2027-02-14",
    "daysLeft": 127,
    "sanCount": 3,
    "tlsVersion": "TLSv1.3",
    "trusted": true
  },
  "security": {
    "score": 85,
    "grade": "B",
    "findings": [
      {"code": "DMARC_NONE", "severity": "low", "text": "DMARC policy is p=none: spoofed mail is only reported, not rejected."},
      {"code": "NO_CAA", "severity": "low", "text": "No CAA record: any certificate authority may issue certificates."},
      {"code": "DNSSEC_OFF", "severity": "low", "text": "DNSSEC is not enabled for this domain."}
    ]
  },
  "checkedAt": "2026-10-09T16:12:28Z",
  "error": null
}
```

**An IDN under a registry without public expiry dates** (`bücher.de`, shortened to `registration`): the
domain is returned in ASCII (Punycode) form, and `note` explains the empty dates.

```json
{
  "input": "bücher.de",
  "domain": "xn--bcher-kva.de",
  "tld": "de",
  "registration": {
    "status": "registered",
    "source": "rdap",
    "registrar": null,
    "created": null,
    "expires": null,
    "nameservers": ["ken.ns.cloudflare.com", "sasha.ns.cloudflare.com"],
    "dnssec": null,
    "statusCodes": ["active"],
    "note": "expiry date not published by this registry"
  }
}
```

**A free domain** (available for registration, no grade):

```json
{
  "input": "this-name-is-surely-free-7q3x9.com",
  "domain": "this-name-is-surely-free-7q3x9.com",
  "tld": "com",
  "registration": {"status": "available", "source": "rdap"},
  "dns": {"resolved": false, "a": [], "aaaa": [], "mx": [], "ns": [], "spf": null, "dmarc": null, "caa": []},
  "ssl": {"reachable": false, "error": "no_address"},
  "security": null,
  "checkedAt": "2026-10-09T16:12:28Z",
  "error": null
}
```

**An invalid entry**:

```json
{
  "input": "not a domain",
  "domain": null,
  "tld": null,
  "registration": null,
  "dns": null,
  "ssl": null,
  "security": null,
  "checkedAt": "2026-10-09T16:12:29Z",
  "error": "invalid_domain"
}
```

In the same run, `https://www.nic.io/` became `nic.io` (grade B: `NO_DMARC`, `NO_CAA`, `DNSSEC_OFF`).

## Search engine visibility: noindex checker and robots.txt test (`indexing`)

Is a site accidentally blocking Google? A classic after a relaunch: the `Disallow: /` or the `noindex` tag of the
staging site went live. The `indexing` check answers it for every domain from public signals only:

1. `https://<domain>/` with up to 5 redirects inside the same domain (`www.` and other subdomains). A redirect to
   another domain is not followed (`offsite_redirect`). If HTTPS refuses the connection, `http://` is tried.
2. `robots.txt` of the host the homepage ends on, read like Google does: the group for `Googlebot` or `Bingbot`,
   else `*`; the longest matching rule wins. No `robots.txt` (4xx) means everything is allowed; a server error
   means Google stops crawling the site.
3. `X-Robots-Tag` header and `<meta name="robots|googlebot|bingbot">` of the homepage: `noindex` or `none`.

The actor sends 2 to 6 requests per domain, at most one per second to the same host, with its own user agent
(`DomainAuditBot`, with a link to this page). It does not use proxies or pretend to be a browser. Sites that
refuse it (403, 429, a Cloudflare or DataDome challenge) get `unknown` with `error` `bot_protection`, never
`blocked`; if only `robots.txt` refuses it, an error status of the homepage counts as unknown too (some
sites answer 404 to every bot), but a `noindex` still counts. Only the homepage is checked, not every page.

Illustrative example (a relaunched site whose staging `robots.txt` went live):

```json
{
  "indexing": {
    "verdict": "blocked",
    "findings": [
      {"code": "robots_disallow", "detail": "/", "bot": "googlebot"},
      {"code": "robots_disallow", "detail": "/", "bot": "bingbot"}
    ],
    "robotsTxt": {"status": "found", "blocksGooglebot": true, "blocksBingbot": true},
    "homepage": {
      "url": "https://example.com/",
      "finalUrl": "https://www.example.com/",
      "status": 200,
      "redirects": 1,
      "noindexMeta": false,
      "noindexHeader": false
    }
  }
}
```

`verdict` is `ok`, `blocked` (at least one of the two search engines cannot index the homepage; finding
`SEARCH_BLOCKED` in the score), `warning` (a hint, no points) or `unknown` (no definite answer; `error` says why).
`robotsTxt.status` is `found`, `missing` or `unreachable`; `null` fields were not checked.

| Finding `code` | Verdict | Meaning |
|---|---|---|
| `robots_unreachable` | blocked | `robots.txt` answers with a server error or 429 while the homepage answers: Google then treats the whole site as disallowed. (If the homepage is down as well, or nothing answers within our time limit, the verdict is `unknown`.) |
| `robots_disallow` | blocked | `robots.txt` disallows the homepage for `bot` (`detail` = path). |
| `http_error` | blocked | The homepage ends with a client error status (`detail`, for example `404`, `410`, `401`). A server error (5xx) is often a passing outage and gives `unknown` with `error` `server_error`. |
| `noindex_header` | blocked | `X-Robots-Tag` says `noindex` or `none` for `bot` (`detail` = header value). |
| `noindex_meta` | blocked | A meta robots tag says `noindex` or `none` for `bot`. |
| `offsite_redirect` | warning | The homepage redirects to another domain (`detail`); that domain is not checked. |
| `canonical_offsite` | warning | The homepage's canonical link points to another domain (`detail`), for example a forgotten staging host. |

## How much does it cost?

The actor itself is free: there is no per-domain or per-result fee. You only pay Apify's platform usage
(compute units, i.e. memory × run time) in your own Apify account, at the rates of your Apify plan; Apify's free
plan includes a monthly amount of platform usage.

As a reference point: a run with the default 1 GB of memory, 3 domains and all checks took 5.6 seconds and cost
$0.00037, almost all of it the start fee. Longer lists cost more because the run takes longer, and the run time
depends mostly on the registries: the actor asks each registry at most once per second (RDAP) or once every two
seconds (WHOIS). A list of 1,000 domains under one ending such as `.com` therefore runs for at least about 17
minutes, roughly 0.3 compute units at 1 GB; lists spread over many endings finish faster, slow or silent servers
make it slower. The `indexing` check adds a few seconds per domain (one request per second per host, at most 15
seconds per request); leave it out if you only need expiry dates. You can set a maximum run time (timeout) in the
run options.

The run log may show "Ignored attempt to charge" lines; they are expected and cost nothing.
Pricing may be introduced later. Apify announces such changes to users 14 days in advance.

## Error codes

An error never stops the run. It is stored in the row, in the place where it happened.

**Row `error`** (the whole row):

| Code | Meaning |
|---|---|
| `invalid_domain` | The entry is not a usable domain name. |
| `internal` | An unexpected error in the actor for this domain. |

**`registration.error`** (`status` is then `unknown`; after `internal` the block holds only `error`):

| Code | Meaning |
|---|---|
| `unsupported_tld` | No RDAP service and no supported WHOIS for this ending (for example `.jp`), or a public-sector or academic namespace such as `gov.uk`, `ac.uk`, `gv.at`, `edu.au`. |
| `rate_limited` | The registry asked us to slow down; after repeated limits it is skipped for the rest of the run. |
| `timeout`, `network` | The registry did not answer in time, or the connection failed. |
| `http_404` | A 404 that did not come from the registry's RDAP service (for example a proxy page). |
| `http_<code>` | The registry answered with another HTTP status, for example `http_500`. |
| `bad_response`, `unparsed` | The RDAP or WHOIS answer could not be read. |
| `internal` | An unexpected error in this check. |

**`dns.error`** (the block then holds only `error`): `timeout`, `servfail` (no name server answered), `dns_error` (other DNS failure), `internal`.

**`ssl.error`** (`reachable` is then `false`, except for `bad_certificate`; after `internal` the block holds only `error`):

| Code | Meaning |
|---|---|
| `connect_failed` | Port 443 refused the connection. Finding `NO_HTTPS`. |
| `tls_error` | The server answered the TLS handshake with an error. Finding `NO_HTTPS`. |
| `no_address` | The domain has no IP address (A/AAAA), so no certificate is checked (`www.` is not tried). |
| `timeout`, `network_error`, `dns_lookup_failed`, `bad_certificate`, `internal` | No definite answer: no reply in time, connection dropped, temporary name-lookup failure, a certificate the actor could not read, or an unexpected error. |

**`indexing.error`** (`verdict` is then `unknown` and `findings` is empty, unless a definite block was found):
`bot_protection` (403, 429 or a challenge page), `server_error` (the homepage answered 5xx), `timeout`, `connect_failed`, `network_error`, `redirect_loop`,
`too_many_redirects`, `invalid_url`, `no_address` (the domain has no IP address), `dns_error`, `unsafe_address`
(the domain points at a private or reserved address, checked before and again on the open connection; nothing is sent there), `internal`.

If the run itself cannot start (for example `'domains' must be a non-empty list of domain names or URLs.`, or
more than 1,000 domains after clean-up), it fails with that message before the first query.

## FAQ

### Can I use it with the Apify API?

Yes. This call runs the actor and returns the rows when it is done:

```bash
curl -X POST "https://api.apify.com/v2/acts/sfa-data~domain-ssl-audit/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"domains": ["example.com", "example.org"]}'
```

The synchronous call waits at most 5 minutes. For long lists, start the run asynchronously and read the
dataset later. Clients for JavaScript and Python are available from Apify.

### Can AI agents use it through MCP?

Yes. Connect your agent (for example Claude, Cursor or another MCP client) to Apify's MCP server at
`https://mcp.apify.com`. The agent can find this actor with the `search-actors` tool, or you can add it
directly with `https://mcp.apify.com?tools=sfa-data/domain-ssl-audit`. A typical call:

```json
{"domains": ["example.com"], "checks": ["registration", "ssl"], "sslWarnDays": 30}
```

The field descriptions in the input schema state the limits of each option. The actor is free; the agent's
Apify account pays only the platform usage of the run.

### How do I get a weekly report?

Save your input as a task and add an Apify schedule (for example every Monday). With Apify integrations you
can send the results to Google Sheets, Slack, e-mail, Zapier, Make or a webhook after each run. Filter on
`security.findings[].code` or `ssl.daysLeft` to get only what needs action.

### Is it legal? Does it store personal data?

The actor reads public registry data (RDAP and WHOIS), public DNS records and the certificate that the server
shows to every visitor. It queries registries slowly (at most one RDAP request per second and one WHOIS request every two seconds
per registry) and stops asking a registry that keeps rate-limiting it. It does **not** store registrant, admin or technical contacts, e-mail
addresses, or the DMARC and CAA report addresses (`rua`, `ruf`, `iodef`); they are dropped while parsing. Of
all TXT records only the SPF record is kept. This makes it easier to use the results under the GDPR and
other privacy laws. You are still responsible for how you use the data; this is not legal advice.

### What are the limits?

- 1,000 domains per run after clean-up; split larger lists into several runs.
- Some registries do not publish expiry or creation dates (for example `.de`, `.ch`, `.nl`, `.eu`, `.at`);
  `registration.note` then says so.
- Premium or reserved names can show as `available`.
- The certificate is checked on the domain itself (or `www.`), port 443, not on other subdomains.
- A row may in rare cases be delivered twice if the platform restarts the run at the wrong moment; this is
  limited to one row per restart.

### What does it not do?

It does not crawl the whole website (only the homepage and `robots.txt`), find subdomains, scan ports, check blacklists or mail deliverability, monitor uptime, or send
alerts by itself (use a schedule and an integration for that). It does not register domains and does not
return any owner contact data.

## Related actor

**[Site SEO Audit: Broken Links, Redirects, Sitemap, Lighthouse](https://apify.com/sfa-data/site-seo-audit)** checks the
pages of a website: status codes and redirects, broken internal links with the pages they are on, sitemap and
robots.txt problems, and Google Lighthouse scores. Use it together with this actor for a full technical
check of a client website.
