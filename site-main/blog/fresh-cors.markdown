---
title: "How to enable API requests in Fresh"
date: 2023-06-07
series: howto
tags:
 - fresh
 - deno
 - preact
---

<xeblog-hero ai="SCMix" file="lemonade" prompt="1girl, green hair, green eyes, long hair, kitchen, lemon, juicer, black hoodie"></xeblog-hero>

We can't trust browsers because they are designed to execute arbitrary
code from website publishers. One of the biggest protections we have
is [Cross-Origin Request
Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
(CORS), which prevents JavaScript from making HTTP requests to
different domains than the one the page is running under.

<xeblog-conv name="Mimi" mood="happy">Cross-Origin Request Sharing
(CORS) is a mechanism that allows web browsers to make requests to
servers on different origins than the current web page. An origin is
defined by the scheme, host, and port of a URL. For example,
https://example.com and https://example.org are different origins,
even though they have the same scheme and port.<br /><br />The browser
implements a CORS policy that determines which requests are allowed
and which are blocked. The browser sends an HTTP header called Origin
with every request, indicating the origin of the web page that
initiated the request. The server can then check the Origin header and
decide whether to allow or deny the request. The server can also send
back an HTTP header called `Access-Control-Allow-Origin`, which
specifies which origins are allowed to access the server's resources.
If the server does not send this header, or if the header does not
match the origin of the request, the browser will block the
response.</xeblog-conv>

[Fresh](https://fresh.deno.dev) is a web framework for
[Deno](https://deno.land) that enables rapid development and is the
thing that I am rapidly reaching to when developing web applications.
One of the big pain points is making HTTP requests to a different
origin (such as making an HTTP request to `api.example.com` when your
application is on `example.com`). The Fresh documentation doesn't have
any examples of enabling CORS.

In order to customize the CORS settings for a Fresh app, copy the
following middleware into `routes/_middleware.ts`:

```typescript
// routes/_middleware.ts

import { MiddlewareHandlerContext } from "$fresh/server.ts";

export async function handler(
  req: Request,
  ctx: MiddlewareHandlerContext<State>,
) {
  const resp = await ctx.next();
  resp.headers.set("Access-Control-Allow-Origin", "*");
  return resp;
}
```

If you need to customize the CORS settings, here's the HTTP headers
you should take a look at:

| Header | Use | Example |
| :----- | :-- | :------ |
| `Access-Control-Allow-Origin` | Allows arbitrary origins for requests, such as `*` for all origins. | `https://xeiaso.net, https://api.xeiaso.net` |
| `Access-Control-Allow-Methods` | Allows arbitrary HTTP methods for requests, such as `*` for all methods. | `PUT, GET, DELETE` |
| `Access-Control-Allow-Headers` | Allows arbitrary HTTP headers for requests, such as `*` for all headers. | `X-Api-Key, Xesite-Trace-Id` |

This should fix your issues.
