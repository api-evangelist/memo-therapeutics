---
name: memo-therapeutics-corporate-documents
description: >-
  Retrieve Memo Therapeutics AG's published corporate and patient-facing policy documents — the
  Expanded Access Policy, privacy policy, cookie policy and terms — as structured JSON rather than
  scraped HTML. Use when you need the current text of a clinical-stage sponsor's stated policies.
generated: '2026-08-25'
method: generated
api: memo-therapeutics:memo-therapeutics-pages-api
base_url: https://memo-therapeutics.com/wp-json
operations:
- listPages
- getPage
auth: none
---

# Retrieve Memo Therapeutics corporate documents

The company's policy documents are pages in the same WordPress content API as its news. Ten pages
were live on 2026-08-25 and the set is small enough to enumerate in one call.

## 1. Enumerate every page

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,modified
```
`operationId: listPages`

On 2026-08-25 this returned, with their last-modified dates:

| slug | title | modified |
|---|---|---|
| *(homepage)* | Homepage | 2026-07-21 |
| `about-us` | About us | 2026-07-21 |
| `bkv` | BKV | 2025-08-07 |
| `latest-news` | Latest News | 2026-07-22 |
| `contact-us` | Contact us | 2026-07-21 |
| `further-information` | Further Information | 2026-07-22 |
| `expanded-access-policy` | Expanded Access Policy | 2026-07-21 |
| `privacy-policy` | Privacy Policy | 2025-04-14 |
| `terms-and-conditions` | Terms and Conditions | 2024-08-08 |
| `cookie-policy` | Cookie Policy | 2024-09-12 |

## 2. Address a document by slug, not by id

```
GET /wp/v2/pages?slug=expanded-access-policy
```

Ids are database-sequential and do not survive a site migration; `slug` and `link` do. This matters
here more than usual — the company was acquired by Ipsen in July 2026 and the site is a candidate
for consolidation into `ipsen.com`.

## 3. Get the document body

```
GET /wp/v2/pages/{id}?_fields=title,content,modified,link
```
`operationId: getPage`

`content.rendered` is HTML. `modified` is the field to diff on if you are watching a policy for
change — the Expanded Access Policy and About us pages both moved on 2026-07-21, the day before
the acquisition completed.

## 4. Watch for change

```
GET /wp/v2/pages?modified_after=<last-seen>&orderby=modified&order=desc&_fields=id,slug,modified
```

Store the highest `modified` you have seen. A policy page changing is the signal; the diff of
`content.rendered` is the payload.

## Constraints

- `per_page` is capped at 100 (**400 `rest_invalid_param`** above that).
- `context=edit` is unavailable anonymously; you get the `view` projection.
- Read-only. Every write route returns 401 and there is no reversal operation to worry about.
- No rate limit is published or signalled — self-throttle.
