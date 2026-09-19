# The Node client

[![npm](https://img.shields.io/npm/v/@xcrapcc/sdk?color=f62d00&label=%40xcrapcc%2Fsdk)](https://www.npmjs.com/package/@xcrapcc/sdk) [![GitHub](https://img.shields.io/badge/source-xcrap-node-141312?logo=github)](https://github.com/XcrapCC/xcrap-node) ![License: MIT](https://img.shields.io/badge/license-MIT-141312)

A thin typed wrapper over the same HTTP endpoints. Nothing it does is impossible with fetch; it just saves you writing the error classes, the retry and the format negotiation again.

> [!NOTE]
> **Kept in step with the API.** Every time an XCrap endpoint is added or changed, this SDK is updated and released alongside it, so the methods and types here always match what the API returns.

**Contents:** [Install](#install) · [Quick start](#quick-start) · [Markdown](#markdown-with-the-token-cost) · [Threads and timelines](#threads-and-timelines) · [Search](#search) · [Errors](#errors) · [Every method](#every-method) · [Examples](#examples)

## Install

```bash
npm install @xcrapcc/sdk
```

<details>
<summary>Other package managers</summary>

```bash
pnpm add @xcrapcc/sdk
yarn add @xcrapcc/sdk
bun add @xcrapcc/sdk
```

</details>

**Requires:** Node 18 or newer. No runtime dependencies — it uses the built-in fetch.

| | |
| --- | --- |
| 🔑 **No key, ever** | Nothing to sign up for, nothing to store in an environment variable, nothing to rotate when somebody commits it. |
| 🧩 **Typed all the way down** | Full declarations for every method and every response shape, so a wrong field name is a red squiggle and not a runtime surprise. |
| 📝 **Markdown built in** | One flag turns any response into prompt-ready markdown, with an estimate of what it will cost your context window. |

## Quick start

There is no key to pass and no client to configure. Construct it and call it.

```js
import { Xcrap } from '@xcrapcc/sdk';

const xcrap = new Xcrap();

const tweet = await xcrap.tweet('https://x.com/jack/status/20');
console.log(tweet.text);               // just setting up my twttr
console.log(tweet.metrics.likes);      // 308067
```

## Markdown, with the token cost

Every read method takes a markdown flag. The response comes back as a string ready for a prompt, and the client records how many tokens that string is likely to cost so a budget can be checked before it is spent.

```js
// Markdown, ready to drop into a prompt.
const md = await xcrap.tweet('https://x.com/jack/status/20', { markdown: true });

// How many tokens that will cost, before you spend them.
console.log(xcrap.lastMeta.markdownTokens);
```

## Threads and timelines

A thread comes back unrolled and in order, from the first post rather than the last. A timeline paginates through cursors so you do not have to hold them.

```js
const thread = await xcrap.thread(
  'https://x.com/naval/status/1002103360646823936',
);

console.log(thread.count);                      // 31
for (const post of thread.tweets) console.log(post.text);
```

## Search

Pass the query exactly as you would type it into X, operators included. A page comes back with the cursor for the next one. Search has the tightest budget of any method, so one precise query beats five broad ones.

```js
const page = await xcrap.search('from:nasa mars', {
  feed: 'latest',          // or 'top', 'photos', 'videos'
  since: '2026-01-01',
});

for (const post of page.tweets) console.log(post.author.screen_name, post.text);

// The next page, when there is one.
if (page.next_cursor) await xcrap.search('from:nasa mars', { cursor: page.next_cursor });
```

## Errors

Every failure is a class you can catch by name. A 404 is a deleted or private post, a 429 carries the seconds until the window resets, and a transport failure is distinguishable from a refusal.

```js
import { Xcrap, XcrapNotFound, XcrapRateLimited } from '@xcrapcc/sdk';

try {
  await xcrap.tweet('https://x.com/jack/status/1');
} catch (error) {
  if (error instanceof XcrapNotFound) return null;
  if (error instanceof XcrapRateLimited) {
    // The client already knows when the window resets.
    await sleep(error.retryAfter * 1000);
  }
  throw error;
}
```

## Every method

All of them accept a format and a markdown flag, and all of them return the same shapes the HTTP API returns.

| Method | What it returns |
| --- | --- |
| `tweet(url, opts?)` | One post, fully resolved: text, author, metrics, media, poll and any quoted post. |
| `thread(url, opts?)` | A whole thread from any post in it, unrolled into posting order. |
| `search(query, opts?)` | Posts matching a query — latest, top, photos or videos — inside an optional date window. |
| `replies(url, opts?)` | One page of the direct replies to a post, most liked or newest first. |
| `user(handle, opts?)` | A profile: bio, counts, join date, verification, location and website. |
| `userTweets(handle, opts?)` | One page of an account's posts, with the cursor for the next. |
| `userHistory(handle, opts?)` | Up to a thousand of an account's posts in one call, newest first, inside an optional date window. |
| `followers(handle, opts?)` | One page of the accounts following an account. |
| `following(handle, opts?)` | One page of the accounts an account follows. |
| `trends(opts?)` | What is trending, with post counts where X gives them. |
| `media(url, opts?)` | Every attached file, with every bitrate X encoded, as direct links. |
| `downloadMedia(url, opts?)` | Streams one of those files back as bytes rather than as a link. |
| `bulk(urls, opts?)` | Up to fifty posts in one request. A dead link fails that entry alone. |

## Examples

Ready-to-run projects live in the [`examples/`](https://github.com/XcrapCC/xcrap-node/tree/main/examples) folder of the SDK repository: a command-line tool, a Discord bot, a Telegram bot, a thread archiver and an AI summary script.

---

Also available: [Python SDK](python.md) · [API reference](../reference/api.md) · [xcrap.cc/sdk/node](https://xcrap.cc/sdk/node)
