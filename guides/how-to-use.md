# How to use XCrap

Pick whichever of these matches how you work. They all reach the same data.

01

## On this site

Paste a link into the box on the homepage. You get a preview, the Markdown, the raw JSON, and download links for any attached media. Nothing to install.

xcrap.cc
Any post also has a permalink here: swap x.com for xcrap.cc in the address bar and the page renders with a full preview card.

x.com
`x.com/jack/status/20`

xcrap.cc
`xcrap.cc/jack/status/20`

02

## From the command line

Every endpoint is a plain GET. curl, wget, HTTPie, a browser address bar, whatever you have.

```
# One post, as markdown
curl 'https://xcrap.cc/v1/tweet?url=https://x.com/jack/status/20&format=markdown'

# A whole thread, from the first post
curl 'https://xcrap.cc/v1/thread?url=https://x.com/naval/status/1002103360646823936&max_tweets=40&format=markdown'

# Ask by header instead of query string
curl -H 'Accept: text/markdown' 'https://xcrap.cc/v1/tweet?url=20'

# Fifty posts in one request
curl -X POST https://xcrap.cc/v1/bulk \
  -H 'content-type: application/json' \
  -d '{"urls":["https://x.com/jack/status/20","1671370010743263233"]}'

# Save the video from a post
curl -OJ 'https://xcrap.cc/v1/media/download?url=1671370010743263233&index=0'
```

03

## In your agent, over MCP

The MCP server exposes each endpoint as a tool, so a model can read X without you writing glue code. The hosted one needs nothing installed: add this to your client config and restart it.

```
{
  "mcpServers": {
    "xcrap": { "url": "https://xcrap.cc/mcp" }
  }
}
```

For Claude Code, run this in your project instead:

```
claude mcp add --transport http xcrap https://xcrap.cc/mcp
```

Prefer to run it on your own machine? The npm package serves the same tools over stdio:

```
npx -y @xcrapcc/mcp
```

04

## In your agent, without MCP

If your harness does not speak MCP, download the skill file. It is one Markdown document that teaches a model the whole API. Drop it into your prompt, your system message, or your skills folder.

[ skills.md ](https://xcrap.cc/skills.md) [ llms.txt ](https://xcrap.cc/llms.txt) [ openapi.json ](https://xcrap.cc/openapi.json)

05

## In code

Official SDKs for Node and Python. Both are thin wrappers over the same REST API, with types and no runtime dependencies.

```
// Node
npm install @xcrapcc/sdk

import { Xcrap } from '@xcrapcc/sdk';

const xcrap = new Xcrap();
const thread = await xcrap.thread(
  'https://x.com/naval/status/1002103360646823936',
  { markdown: true }
);
```

```
# Python
pip install xcrap-sdk

from xcrap import Xcrap

xcrap = Xcrap()
thread = xcrap.thread(
    "https://x.com/naval/status/1002103360646823936",
    markdown=True,
)
```

06

## In a chat

Paste an xcrap.cc link into Discord, Slack, Telegram or WhatsApp and it unfurls into a card with the author, the text and the post’s image. Swap x.com for xcrap.cc, send it, done. Nobody on the other end needs an account to read it.

You

https://xcrap.cc/jack/status/20

xcrap.cc

jack (@jack)

just setting up my twttr

Open Graph
Nothing to configure: every line of the card is read straight off the post.

og:titlejack (@jack)

og:descriptionjust setting up my twttr

og:imageThe post’s first photo, or the author’s avatar when it has none.

## What next

The API reference lists every parameter. The FAQ covers the questions that come up most.

[API reference](https://xcrap.cc/docs) [FAQ](https://xcrap.cc/faq) [ Node SDK](https://xcrap.cc/sdk/node) [ Python SDK](https://xcrap.cc/sdk/python)
