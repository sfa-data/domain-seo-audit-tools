# sfa-data tools

Free website and domain checks that run on [Apify](https://apify.com/sfa-data). Paste a list, click Start,
export the result as CSV, Excel or JSON. You only pay Apify's compute time in your own account.

| Tool | What it checks | Docs |
|---|---|---|
| <img src="icons/domain-ssl-audit.png" width="32"> [Bulk Domain Audit](https://apify.com/sfa-data/domain-ssl-audit) | SSL certificate expiry, domain expiry (RDAP/WHOIS), SPF and DMARC, CAA, DNSSEC, and whether Google or Bing may index the homepage — up to 1,000 domains per run, with an A–F grade. | [domain-ssl-audit.md](domain-ssl-audit.md) |
| <img src="icons/site-seo-audit.png" width="32"> [Site SEO Audit](https://apify.com/sfa-data/site-seo-audit) | Broken links with the pages they are on, redirect chains, HTTP status, sitemap and robots.txt, title and meta data, optional Lighthouse scores — up to 100 websites per run. | [site-seo-audit.md](site-seo-audit.md) |

Both tools also work for AI agents through Apify's MCP server, for example
`https://mcp.apify.com?tools=sfa-data/domain-ssl-audit`.

**Questions or a wrong result?** Open an issue here or on the tool's Apify page. Answers usually within a day.

This repository holds documentation only. The documentation is licensed under CC BY 4.0.
