---
name: memo-therapeutics-content-discovery
description: >-
  Discover what the memo-therapeutics.com content API actually exposes before calling it — read the
  live route index, search across every published object, and resolve any site URL to an oEmbed
  document. Use this as the first step against this host, or as the template for any WordPress-backed
  corporate site.
generated: '2026-08-25'
method: generated
api: memo-therapeutics:memo-therapeutics-discovery-api
base_url: https://memo-therapeutics.com/wp-json
operations:
- getRouteIndex
- getNamespaceIndex
- listTypes
- listTaxonomies
- listStatuses
- search
- getOembed
auth: none
---

# Discover the Memo Therapeutics content surface

Memo Therapeutics publishes no API documentation. The contract is the route index the site serves
about itself, and it is the document every artifact in this profile was derived from.

## 1. Read the route index

```
GET /wp-json/
```
`operationId: getRouteIndex`

Returns `name`, `description`, `url`, `namespaces[]` and `routes{}` — each route with its methods
and its `args` (type, default, enum, minimum, maximum, required). On 2026-08-25 this deployment
advertised **390 routes across 14 namespaces**. The site links it from its HTML:

```
Link: <https://memo-therapeutics.com/wp-json/>; rel="https://api.w.org/"
```

Do not assume a stock WordPress route set. Check `routes` before you call.

## 2. Separate the readable from the gated

Anonymous **200** on 2026-08-25: `/wp/v2/posts` (36), `/wp/v2/pages` (10), `/wp/v2/media` (163),
`/wp/v2/categories` (8), `/wp/v2/tags` (0), `/wp/v2/comments` (0), `/wp/v2/navigation` (1),
`/wp/v2/search` (154), `/wp/v2/types`, `/wp/v2/taxonomies`, `/wp/v2/statuses`, `/oembed/1.0/embed`.

Anonymous **401**: `/wp/v2/users`, `/wp/v2/settings`, `/wp/v2/menus`, `/wp/v2/themes`,
`/wp/v2/plugins`, `/wp/v2/block-types`, `/wp/v2/font-collections`, and the whole `aioseo/v1`,
`elementor*/v1`, `wordfence/v1`, `wp-site-health/v1` and `wp-abilities/v1` administrative surface.

Note `wp-abilities/v1`: the namespace index is public but `GET /wp-abilities/v1/abilities` is
**401**. WordPress core's ability registry — the thing an MCP adapter would bind to — is registered
here and closed. There is no agent surface on this host.

## 3. Enumerate registered shapes

```
GET /wp/v2/types
GET /wp/v2/taxonomies
GET /wp/v2/statuses
```
`operationId: listTypes` / `listTaxonomies` / `listStatuses`

`types` gives each post type its `rest_base`, which is how you build a path for a custom type
without guessing.

## 4. Search across everything at once

```
GET /wp/v2/search?search=potravitug&per_page=100&type=post
```
`operationId: search`

Returns lightweight `{id, title, url, type, subtype}` records across all public types — 154
searchable objects on 2026-08-25. Use it to locate an object, then fetch it from its own
collection for the full representation.

## 5. Resolve any site URL to an oEmbed document

```
GET /oembed/1.0/embed?url=https://memo-therapeutics.com/about-us/
```
`operationId: getOembed`

Returns a genuine oEmbed 1.0 response (`version`, `provider_name`, `provider_url`, `type`,
`width`, `height`, `html`). This is the one place the deployment implements a published
interoperability standard rather than a WordPress-specific shape, and it is the cheapest way to
turn a bare URL into a titled, typed record.

## 6. Know the response contract

- Pagination: `page` / `per_page` (max 100) / `offset`, with `X-WP-Total`, `X-WP-TotalPages` and an
  RFC 8288 `Link: rel="next"`, all CORS-exposed.
- Field selection: `_fields=` allow-list. Embedding: `_embed=1`.
- Errors: `{code, message, data.status}` as `application/json`. **Not** RFC 9457
  `application/problem+json` — do not content-negotiate for it.
- No request-id header, no rate-limit header, no `Sunset`/`Deprecation` header.
