---
name: memo-therapeutics-track-news
description: >-
  Track Memo Therapeutics AG press releases, interviews and event listings from the company's own
  content API, incrementally and without scraping HTML. Use this to watch a clinical-stage biotech's
  disclosure stream — trial readouts, regulatory designations, partnership and acquisition news.
generated: '2026-08-25'
method: generated
api: memo-therapeutics:memo-therapeutics-posts-api
base_url: https://memo-therapeutics.com/wp-json
operations:
- listPosts
- getPost
- listCategories
auth: none
---

# Track Memo Therapeutics news

Memo Therapeutics AG ("MTx"), an Ipsen company since 22 July 2026, publishes its news archive as
JSON from the WordPress content API behind `memo-therapeutics.com`. No key, no registration, no
rate limit is published. 36 posts were live on 2026-08-25.

**Read this first.** The company runs no developer program and makes no promise about this surface.
It is unversioned, has no status page and no deprecation policy, and the site may be consolidated
into `ipsen.com` — `/careers/` already redirects there. Build for it to disappear.

## 1. Get the current page count before you page

```
GET /wp/v2/posts?per_page=1
```
`operationId: listPosts`

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers. Do not increment `page` blindly:
`page=9999` returns **400 `rest_post_invalid_page_number`**, not an empty array. Either stop at
`X-WP-TotalPages` or follow the `Link: <...>; rel="next"` header.

## 2. Pull only what you need

```
GET /wp/v2/posts?per_page=100&_fields=id,date,modified,slug,link,title,excerpt,categories
```

`per_page` is hard-capped at 100 — `per_page=101` returns **400 `rest_invalid_param`** with the
constraint in `data.params.per_page`. `_fields` is an allow-list and cuts a 34 KB page to a few KB.

## 3. Go incremental

```
GET /wp/v2/posts?modified_after=2026-06-01T00:00:00&orderby=modified&order=desc&per_page=100
```

Persist the highest `modified` you have seen and pass it as `modified_after` next run. Use
`modified_after`, not `after` — `after` filters on publish date and will miss a corrected release.

## 4. Resolve the editorial taxonomy once

```
GET /wp/v2/categories?per_page=100
```
`operationId: listCategories`

Eight terms on 2026-08-25, and they carry real meaning: `blog-posts` (36), `past-events` (24),
`nephrology`, `oncology`, `pharmaceutical-companies`, `research-universities`, `industry-forums`,
`archived`. Cache the id→slug map; do not re-fetch it per post.

## 5. Fetch one post in full

```
GET /wp/v2/posts/{id}?_embed=1
```
`operationId: getPost`

`_embed=1` inlines the author and featured media under `_embedded`. Do this rather than calling
`/wp/v2/users` — that collection returns **401 `rest_user_cannot_view`** to anonymous clients, and
`_embedded` is the only anonymous route to author identity.

## Errors you will actually hit

| Status | `code` | What to do |
|---|---|---|
| 400 | `rest_invalid_param` | Read `data.params` — it names the parameter and its constraint. |
| 400 | `rest_post_invalid_page_number` | You paged past `X-WP-TotalPages`. Stop. |
| 401 | `rest_forbidden` / `rest_user_cannot_view` | Authenticated surface. No credential is issued to outsiders. Do not retry. |
| 404 | `rest_no_route` | Re-read `GET /wp-json/` — the route set is deployment-specific. |
| 403 | *(HTML body)* | The edge answered, not WordPress. Back off; do not parse as JSON. |

Full catalog: `errors/memo-therapeutics-problem-types.yml`.

## Rate limits

None are published and none are signalled — no `X-RateLimit-*`, no `RateLimit-*`, no `Retry-After`
appeared on any observed response. Self-throttle. A Wordfence plugin and a StackCDN edge sit in
front of the origin, so sustained volume is more likely to earn an HTML 403 from the edge than a
429 from the application.

## Writes

There are none you can perform. Every `POST`/`PUT`/`PATCH`/`DELETE` route on this deployment
answers 401 to an anonymous client, so nothing here needs an idempotency key and nothing here can
be reversed — there is no action to take back. Treat this API as strictly read-only.
