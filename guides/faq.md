# Frequently asked questions

If something is not answered here, the API reference is more detailed.

### What is XCrap?

A free service that reads public posts, threads and profiles from X and hands them back as Markdown, JSON, YAML, CSV or HTML. It exists so that agents, bots and small projects can read X without an API key.

### Is it really free?

Yes. No account, no key, no card, no trial. It is paid for by donations, and the running costs are a domain and one small server.

### Do I need an API key?

No. There is no key to generate and no header to send. Rate limits are applied per IP address and per endpoint.

### What are the rate limits?

Per IP and per endpoint: 45 requests a minute for single posts and profiles, 15 a minute for threads, timelines, replies and follower lists, 20 a minute for media, 4 history exports and 6 bulk requests every five minutes, and 10 searches every fifteen minutes. Every response tells you where you stand in the x-ratelimit headers. Need more? See the enterprise plan.

### Can it read private or protected accounts?

No, and it never will. Protected accounts return the same 404 they would give anyone else, and direct messages are out of reach entirely. XCrap reads public accounts only: their posts, profiles, replies and follower lists.

### Can it read deleted posts?

No. If a post is gone from X it is gone here. The five-day cache can briefly still hold something that was deleted a moment ago, and that copy is dropped when it expires.

### How do I get a whole thread?

Use /v1/thread with the link to any post in it, including the first. X only tells you a post's parent, so we stitch the author's timeline by conversation id to walk the thread forwards.

### Can I download the video or images?

Yes. /v1/media lists every attached file with a download link, at every bitrate X encoded. Files stream straight through and are never written to our disk.

### What formats can I get?

JSON, Markdown, YAML, CSV and a standalone HTML card. Add ?format= to any request, or send an Accept header.

### How do I use it from an AI agent?

Three ways. Add the MCP server to your client config, download the skill file and paste it into your prompt, or point your framework at the OpenAPI spec.

### What is the x-markdown-tokens header?

An estimate of how many tokens a Markdown response will cost, sent with every Markdown response. It lets an agent decide whether a thread fits in its context before it reads the body.

### Is there full-text search?

Yes: /v1/search takes any query X search understands, operators included (from:, "exact phrase", since: and the rest), and returns the latest or top posts, or only those with photos or videos. It has the tightest rate limit of any endpoint.

### How long do you keep my requests?

Request logs hold the path, timestamp, IP and headers so we can find abuse and debug faults. Post content is cached for five days and then deleted. We are a pipe, not an archive.

### Is this legal?

Reading public web pages is not unauthorised access, which US courts confirmed in hiQ v. LinkedIn. It does break X's terms of service, which is a contract matter between us and X, not a criminal one. You are responsible for what you do with the data you receive.

### I am on X and I do not want to be readable through this.

Email legal@xcrap.cc from an address that can be tied to the account, and the handle goes on a blocklist that is checked before any request runs. No argument, no form.

### Why not use the official API?

Use it if you can. It costs $200 a month at the entry tier and $42,000 at the top, and the free tier cannot read posts at all. XCrap exists for everyone below that line.

### What happens when X changes something?

Four sources sit behind every endpoint and they fail independently. When our own client breaks, requests fall through to the next one and keep answering while we fix it.

### Can I self-host it?

No. XCrap is private source, and there is no distribution to run. If you need guaranteed capacity for something that matters, the enterprise plan offers dedicated capacity and higher limits: write to hello@xcrap.cc. The free API stays free as long as the donations cover the server.

### Can I use this commercially?

Yes. There is no licence key and no usage tier on the free API. Stay within the rate limits, and remember you are responsible for your own compliance with X’s terms and with data protection law where you operate. If your product needs more capacity, the enterprise plan is made for that.

### How can I help?

Donate if you can, since servers are the only real cost. Report anything broken. Tell other people building agents that this exists.

### Are you featured anywhere?

Aura++ lists XCrap in its directory, and its badge is below. If you run a directory and want to add it, write to hello@xcrap.cc and your badge goes up beside that one.

- [ ![Featured on Aura++](https://xcrap.cc/badge/aura-plus-plus.svg) ](https://auraplusplus.com/projects/xcrap-public-x-post-reader-api)

- [ ![Featured on Twelve Tools](https://xcrap.cc/badge/twelve-tools.svg) ](https://twelve.tools)

- [ ![Featured on Findly.tools](https://xcrap.cc/badge/findly-tools.svg) ](https://findly.tools)

- [ ![Featured on Dofollow.tools](https://xcrap.cc/badge/dofollow-tools.svg) ](https://dofollow.tools)

- [ ![XCrap on DR Checker](https://xcrap.cc/badge/drchecker.svg) ](https://drchecker.net/item/xcrap.cc)

## Still stuck?

The API reference documents every parameter and error code.

[API reference](https://xcrap.cc/docs) [How to use](https://xcrap.cc/how-to-use) [ legal@xcrap.cc](mailto:legal@xcrap.cc)
