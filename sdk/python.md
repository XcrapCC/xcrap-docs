# The Python client

[![PyPI](https://img.shields.io/pypi/v/xcrap-sdk?color=f62d00&label=xcrap-sdk)](https://pypi.org/project/xcrap-sdk/) [![GitHub](https://img.shields.io/badge/source-xcrap-python-141312?logo=github)](https://github.com/XcrapCC/xcrap-python) ![License: MIT](https://img.shields.io/badge/license-MIT-141312)

A thin typed wrapper over the same HTTP endpoints. Nothing it does is impossible with fetch; it just saves you writing the error classes, the retry and the format negotiation again.

> [!NOTE]
> **Kept in step with the API.** Every time an XCrap endpoint is added or changed, this SDK is updated and released alongside it, so the methods and types here always match what the API returns.

**Contents:** [Install](#install) · [Quick start](#quick-start) · [Markdown](#markdown-with-the-token-cost) · [Threads and timelines](#threads-and-timelines) · [Search](#search) · [Errors](#errors) · [Every method](#every-method) · [Examples](#examples)

## Install

```bash
pip install xcrap-sdk
```

<details>
<summary>Other package managers</summary>

```bash
uv add xcrap-sdk
poetry add xcrap-sdk
pdm add xcrap-sdk
```

</details>

**Requires:** Python 3.9 or newer. One dependency: httpx.

| | |
| --- | --- |
| 🔑 **No key, ever** | Nothing to sign up for, nothing to store in an environment variable, nothing to rotate when somebody commits it. |
| 🧩 **Typed all the way down** | Full declarations for every method and every response shape, so a wrong field name is a red squiggle and not a runtime surprise. |
| 📝 **Markdown built in** | One flag turns any response into prompt-ready markdown, with an estimate of what it will cost your context window. |

## Quick start

There is no key to pass and no client to configure. Construct it and call it.

```python
from xcrap import Xcrap

with Xcrap() as xcrap:
    tweet = xcrap.tweet("https://x.com/jack/status/20")
    print(tweet.text)            # just setting up my twttr
    print(tweet.metrics.likes)   # 308067
```

## Markdown, with the token cost

Every read method takes a markdown flag. The response comes back as a string ready for a prompt, and the client records how many tokens that string is likely to cost so a budget can be checked before it is spent.

```python
# Markdown, ready to drop into a prompt.
md = xcrap.tweet("https://x.com/jack/status/20", markdown=True)

# How many tokens that will cost, before you spend them.
print(xcrap.last_meta.markdown_tokens)
```

## Threads and timelines

A thread comes back unrolled and in order, from the first post rather than the last. A timeline paginates through cursors so you do not have to hold them.

```python
thread = xcrap.thread(
    "https://x.com/naval/status/1002103360646823936",
)

print(thread.count)                  # 31
for post in thread.tweets:
    print(post.text)

# And a timeline paginates itself.
for post in xcrap.iter_user_tweets("naval", limit=200):
    print(post.id)
```

## Search

Pass the query exactly as you would type it into X, operators included. A page comes back with the cursor for the next one. Search has the tightest budget of any method, so one precise query beats five broad ones.

```python
page = xcrap.search(
    "from:nasa mars",
    feed="latest",          # or "top", "photos", "videos"
    since="2026-01-01",
)

for post in page.tweets:
    print(post.author.screen_name, post.text)

# The next page, when there is one.
if page.next_cursor:
    xcrap.search("from:nasa mars", cursor=page.next_cursor)
```

## Errors

Every failure is a class you can catch by name. A 404 is a deleted or private post, a 429 carries the seconds until the window resets, and a transport failure is distinguishable from a refusal.

```python
from xcrap import Xcrap, XcrapNotFound, XcrapRateLimited

try:
    xcrap.tweet("https://x.com/jack/status/1")
except XcrapNotFound:
    return None
except XcrapRateLimited as error:
    # The client already knows when the window resets.
    time.sleep(error.retry_after)
    raise
```

## Every method

All of them accept a format and a markdown flag, and all of them return the same shapes the HTTP API returns.

| Method | What it returns |
| --- | --- |
| `tweet(url, **opts)` | One post, fully resolved: text, author, metrics, media, poll and any quoted post. |
| `thread(url, **opts)` | A whole thread from any post in it, unrolled into posting order. |
| `search(query, **opts)` | Posts matching a query — latest, top, photos or videos — inside an optional date window. |
| `replies(url, **opts)` | One page of the direct replies to a post, most liked or newest first. |
| `user(handle, **opts)` | A profile: bio, counts, join date, verification, location and website. |
| `user_tweets(handle, **opts)` | One page of an account's posts, with the cursor for the next. |
| `iter_user_tweets(handle, limit=…)` | The same, as a generator that follows the cursors for you and stops at the limit. |
| `user_history(handle, **opts)` | Up to a thousand of an account's posts in one call, newest first, inside an optional date window. |
| `followers(handle, **opts)` | One page of the accounts following an account. |
| `following(handle, **opts)` | One page of the accounts an account follows. |
| `trends(**opts)` | What is trending, with post counts where X gives them. |
| `media(url, **opts)` | Every attached file, with every bitrate X encoded, as direct links. |
| `download_media(url, **opts)` | Streams one of those files back as bytes rather than as a link. |
| `bulk(urls, **opts)` | Up to fifty posts in one request. A dead link fails that entry alone. |

## Examples

Ready-to-run projects live in the [`examples/`](https://github.com/XcrapCC/xcrap-python/tree/main/examples) folder of the SDK repository: a command-line tool, a Discord bot, a Telegram bot, a thread archiver and an AI summary script.

---

Also available: [Node SDK](node.md) · [API reference](../reference/api.md) · [xcrap.cc/sdk/python](https://xcrap.cc/sdk/python)
