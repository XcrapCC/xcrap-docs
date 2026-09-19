# API reference

Every endpoint is a GET, takes no authentication, and answers in whichever of five formats you ask for. The base URL is below. There is nothing else to set up.

## Base URL

`https://xcrap.cc/v1`

### No authentication

There is no key, no token and no header to send. Rate limits are per IP and per endpoint.

### OpenAPI

The full spec is served as JSON and YAML. Point any generator or agent framework at it.

[openapi.json](https://xcrap.cc/openapi.json) [openapi.yaml](https://xcrap.cc/openapi.yaml)

## Response formats

JSON is the default. Override it with ?format= or an Accept header. The two agree; the query parameter wins when both are present.

| ?format= | Accept | Content-Type |
| --- | --- | --- |
| `json` | `application/json` | `application/json` |
| `markdown` | `text/markdown` | `text/markdown` |
| `yaml` | `application/yaml` | `application/yaml` |
| `csv` | `text/csv` | `text/csv` |
| `html` | `text/html` | `text/html` |

x-markdown-tokens
Every markdown response carries an estimate of what it will cost to read, so an agent can decide whether a thread fits in its context before it pulls the body.

## Rate limits

Limits are per endpoint rather than one global budget, because a bulk request can cost fifty upstream calls and a cached post costs none. Every response carries x-ratelimit-limit, x-ratelimit-remaining and x-ratelimit-reset.

| Budget | Requests | Window | Applies to |
| --- | --- | --- | --- |
| `tweet` | 45 | 1 min | /v1/tweet |
| `user` | 45 | 1 min | /v1/user |
| `thread` | 15 | 1 min | /v1/thread |
| `timeline` | 15 | 1 min | /v1/user/tweets |
| `graph` | 15 | 1 min | /v1/user/followers, /v1/user/following |
| `replies` | 15 | 1 min | /v1/replies |
| `history` | 4 | 5 min | /v1/user/history |
| `search` | 10 | 15 min | /v1/search |
| `bulk` | 6 | 5 min | /v1/bulk |
| `media` | 20 | 1 min | /v1/media, /v1/media/download |
| `meta` | 90 | 1 min | /v1/trends |
| `page` | 240 | 1 min | Website pages |

Need more than this? The enterprise plan offers higher limits and dedicated capacity. [See Enterprise →](https://xcrap.cc/enterprise)

## Errors

Errors come back in the format you asked for, with a stable machine-readable code, a message describing what went wrong, and a hint describing what to do about it.

| Status | code | What to do |
| --- | --- | --- |
| 400 | `bad_request` | A parameter is missing or malformed. Check it against this page. |
| 404 | `not_found` | The post or account is deleted, suspended, private, or never existed. For a post, reason says which when X tells us: post_deleted, author_protected, author_suspended, withheld, removed_by_x, age_restricted, or unavailable. |
| 429 | `rate_limited` | Endpoint budget exhausted. Wait for the period in retry-after, or see the enterprise plan for higher limits. |
| 451 | `opted_out` | That account asked to be excluded from XCrap. |
| 500 | `internal_error` | Our fault. Already reported. Try again shortly. |
| 502 | `upstream_failed` | Every source refused. Usually brief; retry in a minute. |
| 504 | `upstream_timeout` | A source did not answer in time. Retry in a minute. |
| 503 | `search_unavailable` | Search capacity is used up for now, or search is not set up. Wait for retry-after. |

Try it
Fill the parameters and send. It goes to the real API — there is no key to add, so there is nothing standing between this form and a live response.

GET `/v1/tweet` 45 / 1m

## One post, fully resolved

Fetches a single post with its author, every engagement counter, attached media at every bitrate, poll results, and the quoted post unnested one level.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `url` Required | `string` | none | The post URL or its numeric id. x.com, twitter.com, the fx/vx mirrors and a bare id all work. |
| `download` | `boolean` | none | Send Content-Disposition so a browser saves the response as a file. |
| `signals` | `boolean` | `false` | Add signals: the post's age, whether it is inside For You's 48-hour window, whether the author may qualify for X's new-author slot, and engagement ratios. Facts from public data, not a ranking score. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/tweet?url=https://x.com/jack/status/20&format=markdown
```

### Try it

url _Required_
download
false true

signals
false true

format
json markdown yaml csv html

fresh
false true

`GET /v1/tweet` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/thread` 15 / 1m

## A whole thread, unrolled in order

Give it any post in a thread, including the first, and it returns the author's posts in reading order. X only exposes a post's parent, so XCrap stitches the author's timeline by conversation id to walk the thread forwards.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `url` Required | `string` | none | Any post in the thread. |
| `max_tweets` | `integer` | `25` | How deep to go, 1 to 100. The response says whether it was truncated. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/thread?url=https://x.com/naval/status/1002103360646823936&format=markdown
```

### Try it

url _Required_ max_tweets
format
json markdown yaml csv html

fresh
false true

`GET /v1/thread` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/replies` 15 / 1m

## The replies under a post

One page of the replies to a post, most liked first or newest first, with the post itself included. Only direct replies come back: a reply to a reply belongs to its own conversation. There is no paging: this is the single page X serves for the post, up to about a hundred replies.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `url` Required | `string` | none | The post whose replies to read: its URL or numeric id. |
| `sort` | `string` | `top` | top for the most liked first, recent for the newest first. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/replies?url=https://x.com/naval/status/1002103360646823936&sort=top&format=markdown
```

### Try it

url _Required_
sort
top recent

format
json markdown yaml csv html

fresh
false true

`GET /v1/replies` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/search` 10 / 15m

## Search posts

Full-text search over X, with the operators X search understands: from:, to:, "exact phrase", -exclude, lang:, filter:links and the rest. Choose the latest or top posts, or only posts with photos or videos. One page per call; pass next_cursor for the next. Search has the tightest budget of any endpoint, and its results are cached for ten minutes.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `q` Required | `string` | none | What to search for, including any X search operators. |
| `feed` | `string` | `latest` | latest, top, photos or videos. |
| `since` | `string` | none | Oldest post to match, as a date such as 2025-01-01 or a full ISO timestamp. |
| `until` | `string` | none | Newest post to match, as a date or a full ISO timestamp. |
| `cursor` | `string` | none | The next_cursor from a previous response. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/search?q=from:nasa%20mars&feed=latest&format=markdown
```

### Try it

q _Required_
feed
latest top photos videos

since until cursor
format
json markdown yaml csv html

fresh
false true

`GET /v1/search` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/user` 45 / 1m

## A profile

Bio, location, website, join date, verification type, avatar and banner at full resolution, and the follower, following, post and media counts.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `handle` Required | `string` | none | The handle, with or without the @, or a full profile URL. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/user?handle=naval&format=markdown
```

### Try it

handle _Required_
format
json markdown yaml csv html

fresh
false true

`GET /v1/user` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/user/tweets` 15 / 1m

## An account's posts

A page of an account's posts, newest first. Paging is by cursor rather than offset, because a timeline moves while you read it and an offset would silently skip or repeat posts.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `handle` Required | `string` | none | The account to read. |
| `count` | `integer` | `20` | How many posts to return, 1 to 100. |
| `cursor` | `string` | none | The next_cursor from a previous response. |
| `exclude_replies` | `boolean` | `true` | Leave out replies the account made to other people. |
| `media_only` | `boolean` | `false` | Only posts carrying an image or video. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/user/tweets?handle=naval&count=20
```

### Try it

handle _Required_ count cursor
exclude_replies
true false

media_only
false true

format
json markdown yaml csv html

fresh
false true

`GET /v1/user/tweets` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/user/followers` 15 / 1m

## Who follows an account

One page of the accounts following this one, each as a full profile. X decides the page size, usually a few dozen; pass next_cursor for the next page, and it is null on the last one.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `handle` Required | `string` | none | The account whose followers to list. |
| `cursor` | `string` | none | The next_cursor from a previous response. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/user/followers?handle=jack&format=markdown
```

### Try it

handle _Required_ cursor
format
json markdown yaml csv html

fresh
false true

`GET /v1/user/followers` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/user/following` 15 / 1m

## Who an account follows

One page of the accounts this one follows, each as a full profile. X decides the page size; pass next_cursor for the next page, and it is null on the last one.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `handle` Required | `string` | none | The account whose follows to list. |
| `cursor` | `string` | none | The next_cursor from a previous response. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/user/following?handle=jack&format=markdown
```

### Try it

handle _Required_ cursor
format
json markdown yaml csv html

fresh
false true

`GET /v1/user/following` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/user/history` 4 / 5m

## An account's posts, in bulk

Walks an account's timeline and returns up to 1,000 posts in one response, newest first, optionally inside a date window. The walk takes about a second per twenty posts, so for a large export ask for format=ndjson: posts then arrive one per line as they are fetched, roughly newest first, instead of all at once at the end. The further back a window sits, the patchier X's own timeline is, so a years-old window can come back thin or empty.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `handle` Required | `string` | none | The account to export. |
| `max_posts` | `integer` | `200` | Stop after this many posts, 1 to 1000. |
| `since` | `string` | none | Oldest post to include, as a date such as 2025-01-01 or a full ISO timestamp. |
| `until` | `string` | none | Newest post to include, as a date or a full ISO timestamp. |
| `include_replies` | `boolean` | `false` | Include the account's replies to other people. |
| `include_reposts` | `boolean` | `false` | Include posts the account reposted from other accounts. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html, or ndjson to stream one post per line. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/user/history?handle=naval&max_posts=200&since=2025-01-01
```

### Try it

handle _Required_ max_posts since until
include_replies
false true

include_reposts
false true

format
json markdown yaml csv html ndjson

fresh
false true

`GET /v1/user/history` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/trends` 90 / 1m

## What is trending

Current trending topics with the context label X attaches to each. Cached for five minutes rather than five days, because a stale trend list is worse than none.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `count` | `integer` | `20` | How many topics, 1 to 50. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/trends?count=10&format=markdown
```

### Try it

count
format
json markdown yaml csv html

fresh
false true

`GET /v1/trends` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/media` 20 / 1m

## List a post's media

Every file attached to a post: type, dimensions, duration, the author's alt text, every video rendition X encoded, and a ready-made download link for each.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `url` Required | `string` | none | The post to inspect. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
https://xcrap.cc/v1/media?url=https://x.com/i/status/1671370010743263233
```

### Try it

url _Required_
format
json markdown yaml csv html

fresh
false true

`GET /v1/media` Reset Send request

Rate limits apply here exactly as they do anywhere else.

GET `/v1/media/download` 20 / 1m

## Download one file

Streams the file straight through with a Content-Disposition header. Nothing is buffered and nothing is written to our disk, which is what the retention promise in the privacy policy means in practice.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `url` Required | `string` | none | The post the file belongs to. |
| `index` | `integer` | `0` | Which file, when a post carries several. |
| `quality` | `string` | `best` | best or worst. Video only; images have one resolution. |

### Example

```
https://xcrap.cc/v1/media/download?url=https://x.com/i/status/1671370010743263233&index=0
```

### Try it

url _Required_ index
quality
best worst

`GET /v1/media/download` Reset Send request

Rate limits apply here exactly as they do anywhere else.

POST `/v1/bulk` 6 / 5m

## Up to fifty posts at once

Resolve many posts in one request. Failures are per item: one dead link returns an error for that entry and every other entry still comes back. GET works too, with repeated or comma-separated url parameters.

### Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `urls` Required | `string[]` | none | One to fifty post URLs or ids, in the JSON body. |
| `format` | `string` | `json` | Response format: json, markdown, yaml, csv or html. Overrides the Accept header. |
| `fresh` | `boolean` | `false` | Skip the cache and refetch from upstream. Use sparingly. |

### Example

```
curl -X POST https://xcrap.cc/v1/bulk \
  -H 'content-type: application/json' \
  -d '{"urls":["https://x.com/jack/status/20","1671370010743263233"]}'
```

### Try it

body _application/json_ {"urls":["https://x.com/jack/status/20","1671370010743263233"]}

format
json markdown yaml csv html

fresh
false true

`POST /v1/bulk` Reset Send request

Rate limits apply here exactly as they do anywhere else.
