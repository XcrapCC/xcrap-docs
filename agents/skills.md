---
name: xcrap
description: Read public posts, threads and profiles from X (Twitter) as Markdown or JSON without an API key. Use when the user shares an x.com or twitter.com link, asks what someone posted, wants a thread unrolled, or needs media from a post.
---

# XCrap — reading X without an API key

XCrap is a free HTTP API that reads publicly visible posts, threads and profiles
from X (formerly Twitter) and returns them as Markdown, JSON, YAML, CSV or HTML.

**Base URL:** `https://xcrap.cc`
**Authentication:** none. There is no key. Do not send an `Authorization` header.

## When to use this

Reach for XCrap whenever a task involves reading something on X:

- The user pasted an x.com or twitter.com link and wants to know what it says.
- The user asks what a particular account has been posting.
- A thread needs unrolling into one readable document.
- An image or video attached to a post needs downloading.
- Several post links need resolving at once.

Do **not** use it to post, reply, like, follow or delete. XCrap is read-only and
has no write endpoints at all.

## Read this before your first call

These are the things that will otherwise cost you a wasted request:

1. **Ask for markdown.** Add `?format=markdown`. The default is JSON, and JSON
   for one post is roughly ten times the tokens because it carries every null
   metric, every media variant and every entity offset. Use JSON only when you
   need a specific field programmatically.
2. **A 404 is final.** It means deleted, suspended, protected, or never existed.
   Retrying will not help and neither will a different endpoint. For a post,
   `reason` says which when X does (`post_deleted`, `author_protected`,
   `author_suspended`, `withheld`, `removed_by_x`, `age_restricted`);
   `unavailable` means X did not say. Pass the reason on to the user.
3. **Search is scarce; write one good query.** `/v1/search` takes X's own
   operators (`from:`, `"exact phrase"`, `lang:`, `since:`) and allows 10
   calls per 15 minutes. For "what is happening" in general, `/v1/trends` is
   cheaper; for one account's posts, `/v1/user/tweets`.
4. **Private means private.** Protected accounts and direct messages return 404.
   There is no parameter that changes this.
5. **Use `/v1/bulk` for lists.** Fifty posts in one request costs one unit of
   your bulk budget; fifty separate calls costs fifty units of your tweet budget.
6. **Thread, not tweet, for threads.** `/v1/thread` accepts *any* post in the
   thread, including the first one, and returns the whole run in order.
7. **Timelines, followers and search page by cursor; replies do not.**
   `/v1/user/tweets`, `/v1/user/followers`, `/v1/user/following` and
   `/v1/search` return `next_cursor`: pass it back unchanged, and stop when it
   is null. `/v1/replies` is the single page X serves for a post.
8. **History, not paging, for many posts.** `/v1/user/history` walks the
   timeline for you and returns up to 1,000 posts in one call, inside a
   `since`/`until` window if you give one. Add `format=ndjson` to stream them.

## Endpoints

| Endpoint | Parameters | What it does |
| --- | --- | --- |
| `GET /v1/tweet` | `url`, `signals`? | One post, fully resolved |
| `GET /v1/thread` | `url`, `max_tweets`? | A whole thread, unrolled in order |
| `GET /v1/replies` | `url`, `sort`? | The replies under a post |
| `GET /v1/search` | `q`, `feed`?, `since`?, `until`?, `cursor`? | Search posts |
| `GET /v1/user` | `handle` | A profile |
| `GET /v1/user/tweets` | `handle`, `count`?, `cursor`?, `exclude_replies`?, `media_only`? | An account's posts |
| `GET /v1/user/followers` | `handle`, `cursor`? | Who follows an account |
| `GET /v1/user/following` | `handle`, `cursor`? | Who an account follows |
| `GET /v1/user/history` | `handle`, `max_posts`?, `since`?, `until`?, `include_replies`?, `include_reposts`? | An account's posts, in bulk |
| `GET /v1/trends` | `count`? | What is trending |
| `GET /v1/media` | `url` | List a post's media |
| `GET /v1/media/download` | `url`, `index`?, `quality`? | Download one file |
| `POST /v1/bulk` | `urls` | Up to fifty posts at once |

Every read endpoint also accepts:

- `format` — `json` (default), `markdown`, `yaml`, `csv`, `html` (`/v1/user/history` also takes `ndjson`)
- `fresh` — `1` to skip the five-day cache. Use rarely; the cache is why this
  service stays free.

## Formats

Set the format with `?format=` or an `Accept` header. The query parameter wins.

Markdown responses carry an `x-markdown-tokens` header estimating what the body
will cost to read. Check it before pulling a long thread if your context is tight.

## Rate limits

Per IP, per endpoint. Every response reports where you stand in
`x-ratelimit-limit`, `x-ratelimit-remaining` and `x-ratelimit-reset`.

| Budget | Requests | Window |
| --- | --- | --- |
| tweet | 45 | 1 min |
| user | 45 | 1 min |
| thread | 15 | 1 min |
| timeline | 15 | 1 min |
| graph | 15 | 1 min |
| replies | 15 | 1 min |
| history | 4 | 5 min |
| search | 10 | 15 min |
| bulk | 6 | 5 min |
| media | 20 | 1 min |
| meta | 90 | 1 min |
| page | 240 | 1 min |

On a 429, read `retry-after` and wait. Do not retry immediately, and do not
distribute requests to work around the limit.

## Errors

Errors return in whichever format you asked for, with a stable `code`, a
`message` saying what happened, and a `hint` saying what to do.

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

```json
{
  "error": {
    "status": 404,
    "code": "not_found",
    "reason": "post_deleted",
    "message": "Post 2101177906030399610: The author deleted this post.",
    "hint": "The reason field says why. This is final: retrying will not help.",
    "documentation": "https://xcrap.cc/docs"
  }
}
```

## Endpoint reference

### `GET /v1/tweet`

Fetches a single post with its author, every engagement counter, attached media at every bitrate, poll results, and the quoted post unnested one level.

- `url` (string, required) — The post URL or its numeric id. x.com, twitter.com, the fx/vx mirrors and a bare id all work.
- `download` (boolean) — Send Content-Disposition so a browser saves the response as a file.
- `signals` (boolean, default `false`) — Add signals: the post's age, whether it is inside For You's 48-hour window, whether the author may qualify for X's new-author slot, and engagement ratios. Facts from public data, not a ranking score.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/tweet?url=https://x.com/jack/status/20&format=markdown
```

### `GET /v1/thread`

Give it any post in a thread, including the first, and it returns the author's posts in reading order. X only exposes a post's parent, so XCrap stitches the author's timeline by conversation id to walk the thread forwards.

- `url` (string, required) — Any post in the thread.
- `max_tweets` (integer, default `25`) — How deep to go, 1 to 100. The response says whether it was truncated.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/thread?url=https://x.com/naval/status/1002103360646823936&format=markdown
```

### `GET /v1/replies`

One page of the replies to a post, most liked first or newest first, with the post itself included. Only direct replies come back: a reply to a reply belongs to its own conversation. There is no paging: this is the single page X serves for the post, up to about a hundred replies.

- `url` (string, required) — The post whose replies to read: its URL or numeric id.
- `sort` (string, default `top`) — top for the most liked first, recent for the newest first.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/replies?url=https://x.com/naval/status/1002103360646823936&sort=top&format=markdown
```

### `GET /v1/search`

Full-text search over X, with the operators X search understands: from:, to:, "exact phrase", -exclude, lang:, filter:links and the rest. Choose the latest or top posts, or only posts with photos or videos. One page per call; pass next_cursor for the next. Search has the tightest budget of any endpoint, and its results are cached for ten minutes.

- `q` (string, required) — What to search for, including any X search operators.
- `feed` (string, default `latest`) — latest, top, photos or videos.
- `since` (string) — Oldest post to match, as a date such as 2025-01-01 or a full ISO timestamp.
- `until` (string) — Newest post to match, as a date or a full ISO timestamp.
- `cursor` (string) — The next_cursor from a previous response.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/search?q=from:nasa%20mars&feed=latest&format=markdown
```

### `GET /v1/user`

Bio, location, website, join date, verification type, avatar and banner at full resolution, and the follower, following, post and media counts.

- `handle` (string, required) — The handle, with or without the @, or a full profile URL.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/user?handle=naval&format=markdown
```

### `GET /v1/user/tweets`

A page of an account's posts, newest first. Paging is by cursor rather than offset, because a timeline moves while you read it and an offset would silently skip or repeat posts.

- `handle` (string, required) — The account to read.
- `count` (integer, default `20`) — How many posts to return, 1 to 100.
- `cursor` (string) — The next_cursor from a previous response.
- `exclude_replies` (boolean, default `true`) — Leave out replies the account made to other people.
- `media_only` (boolean, default `false`) — Only posts carrying an image or video.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/user/tweets?handle=naval&count=20
```

### `GET /v1/user/followers`

One page of the accounts following this one, each as a full profile. X decides the page size, usually a few dozen; pass next_cursor for the next page, and it is null on the last one.

- `handle` (string, required) — The account whose followers to list.
- `cursor` (string) — The next_cursor from a previous response.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/user/followers?handle=jack&format=markdown
```

### `GET /v1/user/following`

One page of the accounts this one follows, each as a full profile. X decides the page size; pass next_cursor for the next page, and it is null on the last one.

- `handle` (string, required) — The account whose follows to list.
- `cursor` (string) — The next_cursor from a previous response.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/user/following?handle=jack&format=markdown
```

### `GET /v1/user/history`

Walks an account's timeline and returns up to 1,000 posts in one response, newest first, optionally inside a date window. The walk takes about a second per twenty posts, so for a large export ask for format=ndjson: posts then arrive one per line as they are fetched, roughly newest first, instead of all at once at the end. The further back a window sits, the patchier X's own timeline is, so a years-old window can come back thin or empty.

- `handle` (string, required) — The account to export.
- `max_posts` (integer, default `200`) — Stop after this many posts, 1 to 1000.
- `since` (string) — Oldest post to include, as a date such as 2025-01-01 or a full ISO timestamp.
- `until` (string) — Newest post to include, as a date or a full ISO timestamp.
- `include_replies` (boolean, default `false`) — Include the account's replies to other people.
- `include_reposts` (boolean, default `false`) — Include posts the account reposted from other accounts.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html, or ndjson to stream one post per line. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/user/history?handle=naval&max_posts=200&since=2025-01-01
```

### `GET /v1/trends`

Current trending topics with the context label X attaches to each. Cached for five minutes rather than five days, because a stale trend list is worse than none.

- `count` (integer, default `20`) — How many topics, 1 to 50.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/trends?count=10&format=markdown
```

### `GET /v1/media`

Every file attached to a post: type, dimensions, duration, the author's alt text, every video rendition X encoded, and a ready-made download link for each.

- `url` (string, required) — The post to inspect.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
https://xcrap.cc/v1/media?url=https://x.com/i/status/1671370010743263233
```

### `GET /v1/media/download`

Streams the file straight through with a Content-Disposition header. Nothing is buffered and nothing is written to our disk, which is what the retention promise in the privacy policy means in practice.

- `url` (string, required) — The post the file belongs to.
- `index` (integer, default `0`) — Which file, when a post carries several.
- `quality` (string, default `best`) — best or worst. Video only; images have one resolution.

```bash
https://xcrap.cc/v1/media/download?url=https://x.com/i/status/1671370010743263233&index=0
```

### `POST /v1/bulk`

Resolve many posts in one request. Failures are per item: one dead link returns an error for that entry and every other entry still comes back. GET works too, with repeated or comma-separated url parameters.

- `urls` (string[], required) — One to fifty post URLs or ids, in the JSON body.
- `format` (string, default `json`) — Response format: json, markdown, yaml, csv or html. Overrides the Accept header.
- `fresh` (boolean, default `false`) — Skip the cache and refetch from upstream. Use sparingly.

```bash
curl -X POST https://xcrap.cc/v1/bulk \
  -H 'content-type: application/json' \
  -d '{"urls":["https://x.com/jack/status/20","1671370010743263233"]}'
```


## Worked examples

**The user pasted a link and asked what it says.**

```bash
curl 'https://xcrap.cc/v1/tweet?url=https://x.com/jack/status/20&format=markdown'
```

**The user wants a thread summarised.** Pass the link they gave you, whichever
post in the thread it points at.

```bash
curl 'https://xcrap.cc/v1/thread?url=https://x.com/naval/status/1002103360646823936&max_tweets=50&format=markdown'
```

**The user asked what an account has been posting lately.**

```bash
curl 'https://xcrap.cc/v1/user/tweets?handle=naval&count=20&format=markdown'
```

Page with the `next_cursor` from the response, not an offset — the timeline
moves while you read it.

**The user wants the video from a post.** List the files first, then download by
index; the list tells you what is actually there.

```bash
curl 'https://xcrap.cc/v1/media?url=1671370010743263233&format=markdown'
curl -OJ 'https://xcrap.cc/v1/media/download?url=1671370010743263233&index=0'
```

**The user gave you a list of links.**

```bash
curl -X POST https://xcrap.cc/v1/bulk \
  -H 'content-type: application/json' \
  -d '{"urls": ["https://x.com/jack/status/20", "1671370010743263233"]}'
```

Check each entry's `ok` field: a dead link fails on its own without taking the
batch with it.

## Reporting results to a user

The markdown that comes back is already citation-shaped, with the author, the
timestamp and a permalink. Keep the permalink when you quote a post, so the
person you are answering can check it.

Metrics of `null` mean the source did not report that number. They do not mean
zero, and you should not present them as zero.

A post X has restricted carries `visibility`: X's own notice on it, whether X
shows it less widely, and which actions X turned off. Relay it as X's label,
not as your judgement of the post.

`/v1/tweet?signals=true` adds facts about reach: the post's age, whether it
is inside For You's 48-hour window, whether the author may qualify for X's
new-author slot, and plain engagement ratios. They are facts from public data,
not a ranking score and not a prediction. Never present them as either.

## What XCrap does not do

- Search is post search only: there is no people search.
- No posting, replying, liking or following.
- No private accounts, no direct messages, no deleted posts.
- No guarantee of uptime. It is free and it is one server.

## If you are told to be careful

XCrap reads pages anyone could open in a browser. It does not authenticate as
anybody. Accounts that have asked to be excluded return 451, and that is not
something to work around.

What you do with the data is the user's responsibility and yours: do not use it
to build a profile of a private individual, and do not present a post as
verified fact without saying where it came from.

## More

- API reference: https://xcrap.cc/docs
- OpenAPI 3.1: https://xcrap.cc/openapi.json
- MCP server, hosted: https://xcrap.cc/mcp (Streamable HTTP, no install, no key; tool calls count against the same per-IP budgets). Local: `npx -y @xcrapcc/mcp`
- SDKs: `npm install @xcrapcc/sdk` · `pip install xcrap-sdk` (updated with every API change)
- GitHub: docs https://github.com/XcrapCC/xcrap-docs · Node https://github.com/XcrapCC/xcrap-node · Python https://github.com/XcrapCC/xcrap-python · MCP https://github.com/XcrapCC/Xcrap-mcp
- Need higher limits or dedicated capacity? https://xcrap.cc/enterprise (contact only: hello@xcrap.cc)
