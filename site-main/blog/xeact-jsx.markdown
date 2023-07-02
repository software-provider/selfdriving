---
title: "Xeact Version 0.69.71: JSX support"
date: 2022-08-17
tags:
 - jsx
 - frontend
 - javascript
series: xeact
---

<xeblog-hero file="the-magician-anime" prompt="The Magician in the style of an anime as a tarot card"></xeblog-hero>

[Xeact](https://github.com/Xe/Xeact) is the most popular femtoframework for
discerning development teams. It has been used in at least 3 production facing
web applications and has become well-loved by users. You simply cannot argue
with this star count:

![A graph of Xeact's star count going up and to the right](https://cdn.xeiaso.net/file/christine-static/blog/star-history-2022817.png)

No React parody is complete without JSX support, so today we at Xeserv are proud
to announce the immediate availability of [Xeact
0.69.71](https://github.com/Xe/Xeact/commit/a9648c112e0e229f0b21ac52f5fb0694ff4fa1fb).

This monumental release brings in officially sanctioned JSX support to the Xeact
ecosystem. JSX has been criticized and called names such as "reverse PHP in
JavaScript", but the introduction of this critical feature enables users to have
unparalleled freedom in their usage of Xeact technology.

<xeblog-conv name="Numa" mood="delet">Now with additional Funky Kong
mode!</xeblog-conv>

No more complicated build system. No more needless complexity. Xeact has only
the needful amounts of complexity to help you focus on keeping things simple.
Xeact with JSX allows users to create and maintain applications of limitless
scale and complexity without sacrificing the core Xeact values.

<xeblog-conv name="Numa" mood="delet">Makefiles for the winz0rz!</xeblog-conv>

No longer do you need to write Xeact code that looks like this:

```javascript
const app = async () => {
    // do something important.
    
    return div({}, [
        h1("hi there friends!"),
        p([t("holy cow these are some WORDS like 'lumbersexual macchiatto'. I don't know what they mean though!")]),
    ]),
};

r(() => {
    let root = g("app");
    let contents = await app();
    x(root);
});
```

That is so many parentheses and m-expressions, it looks like you are going to
make Batman jealous!

No, this won't do at all! We need something simpler. We need the simplicity and
ease of understanding that Xeact brings to the table. Here is what you can do
with the unrestrained power of Xeact and JSX's unholy matrimony:

```jsx
const Page = async () => {
    // do something important
    
    return (
        <div>
            <h1>hi there friends!</h1>
            <p>holy cow these are some WORDS like 'lumbersexual macchiatto'. I don't know what they mean though!"</p>
        </div>
    );
};

r(() => {
    let root = g("app");
    let contents = await Page();
    x(root);
});
```

Much better. Xeact is freed from the yoke of its own grammar and this allows you
to transcend the boundaries of the flesh to create anything you can imagine with
the unlimited power afforded to you. Imagine what you could create with such
limitless potential!

## How to use this power

Xeact's JSX support has only been tested with [Deno](https://deno.land)'s JSX
(and TSX) compilation support. Write your code in what you wish you could write
and then use Deno to turn that into what you actually have to write. To get
started, first you need to install Deno somehow. If you are using Nix flakes,
add `pkgs.deno` to your `devShell` or run `nix shell nixpkgs#deno`. If you are
using a lesser operating system, follow [Deno's
instructions](https://deno.land/#installation) and press enter until the
messages go away.

Then make a file called `deno.json` and copy this into it:

```json
{
    "compilerOptions": {
        "jsx": "react-jsx",
        "jsxImportSource": "xeact",
    },
    "importMap": "./import_map.json",
}
```

Then make a file called `import_map.json` and copy this into it:

```json
{
    "imports": {
        "xeact": "https://xena.greedo.xeserv.us/pkg/xeact/v0.69.71/xeact.ts",
        "xeact/jsx-runtime": "https://xena.greedo.xeserv.us/pkg/xeact/v0.69.71/jsx-runtime.js",
    }
}
```

<xeblog-conv name="Cadey" mood="coffee">I wish you could do this with less
files.</xeblog-conv>

<xeblog-conv name="Numa" mood="delet">Why? This is perfect as it is. Any less
would ruin the beautiful vision we have for Xeact and absolutely stymie the ease
of use! Once you get your JavaScript environment to fuck off and die then
everything becomes so beautifully simple you could eat off of it!</xeblog-conv>

<xeblog-conv name="Cadey" mood="facepalm">If you say so...</xeblog-conv>

Finally you can make your `src/test.tsx` file for your Xeact project:

```typescript
// src/test.tsx

/** @jsxImportSource xeact */

import { g, r, x } from "xeact";

export const Page = async () => {
    return (
        <div>
            <h1>hi there friends!</h1>
            <p>holy cow these are some WORDS like 'lumbersexual macchiatto'. I don't know what they mean though!"</p>
        </div>
    );
};

r(async () => {
    let root = g("app");
    let contents = await Page();
    x(root);
});
```

You can test this awesome power with `deno bundle`:

```console
$ mkdir -p static/js && deno bundle ./src/test.tsx ./static/js/test.js
Download https://xena.greedo.xeserv.us/pkg/xeact/v0.69.71/jsx-runtime.js
Download https://xena.greedo.xeserv.us/pkg/xeact/v0.69.71/xeact.ts
Download https://xena.greedo.xeserv.us/pkg/xeact/v0.69.71/xeact.js
Check file:///home/cadey/code/Xe/Xeact-demo/src/test.tsx
Bundle file:///home/cadey/code/Xe/Xeact-demo/src/test.tsx
Emit "./static/js/test.js" (1.25KB)
```

Finally you can witness your wondrous code with some crappy HTML like this:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Page Title</title>
        <link rel="stylesheet" href="/static/xess.css">
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    </head>
    <body id="top">
        <main>
            <div id="app">Loading...</div>
            <script src="/static/js/test.js" type ="module"></script>
        </main>
    </body>
</html>
```

This will do everything you need. You can write all the modern code you want and
with the power of Xeact, everything will just work out. Component functions work
too!

```typescript
/** @jsxImportSource xeact */

import { g, r, x, u } from "xeact";

type HipsumProps = {
    type: "hipster-centric" | "hipster-latin";
    sentences: number;
};

const getHipsum = async ({type, sentences}: HipsumProps): Promise<string[]> => {
    const resp = await fetch(u("http://hipsum.co/api/", {
        type, sentences, "start-with-lorem": "1"
    }));

    const text: string[] = await resp.json()
    return text;
}

const Hipsum = ({text}: {text: string[]}) => {
    return (
        <div>
            {text.map(para => <p>{para}</p>)}
        </div>
    );
}

export const Page = async () => {
    let paragraph = await getHipsum({type: "hipster-centric", sentences: 8});
    return (
        <div>
            <h1>Lumbersexual macchiatto</h1>
            <Hipsum text={paragraph} />
        </div>
    );
};

r(async () => {
    let root = g("app");
    let contents = await Page();
    x(root);
    root.appendChild(contents);
});
```

This gets you art like this:

![Lumbersexual macchiatto I'm baby 90's thundercats chia poke tbh occupy
humblebrag twee prism butcher whatever taxidermy. Brooklyn master cleanse
hashtag beard tumeric twee. Normcore af hexagon gochujang, DSA ascot kitsch
taxidermy. Fanny pack letterpress af squid. Tofu coloring book twee praxis
meggings. Drinking vinegar skateboard try-hard big mood, DSA prism pok pok
shaman. Lyft man bun whatever, fam affogato godard bicycle rights letterpress.
Photo booth sustainable VHS DSA, organic hammock everyday carry cornhole migas
occupy
8-bit.](https://cdn.xeiaso.net/file/christine-static/blog/Screenshot+from+2022-08-17+21-23-37.png)

If you want to witness the fearsome code that powers this humble demo, check
[here](https://github.com/Xe/Xeact-demo) for your gateway to immortality.

<xeblog-conv name="Numa" mood="delet">Please be sure to follow the Xeact style
guide when writing an app that uses Xeact. This means your default component for
each page should be named "Page" and you must use semicolons everywhere to be
sure the JavaScript compiler knows you are terminating a
statement.</xeblog-conv>

What could you create if the cloud was no longer the limit? What is the logical
conclusion of your power if you have nothing holding you back? What can you do
with Xeact?

<center>

![](https://cdn.xeiaso.net/file/christine-static/blog/6qaoh2.jpg)

</center>
