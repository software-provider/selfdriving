---
title: Building Xeact components with esbuild and Nix
date: 2023-04-08
tags:
 - nix
 - xeact
 - esbuild
 - frontend
---

[Xeact](https://github.com/Xe/Xeact) has succeeded in its goal of
intergalactic domination of the attention space of frontend
developers. Not only has it been the catalyst towards my true
understanding of front-end development, it is the most popular
frontend femtoframework as signified by [this handy
graph](https://star-history.com/#Xe/Xeact&Date):

<xeblog-picture path="blog/2023/xeact/star-history-202342"></xeblog-picture>

However, my deployment process for Xeact components relies on the
`deno bundle` command, which is [being
deprecated](https://deno.land/manual@v1.32.3/tools/bundler):

```
Warning "deno bundle" is deprecated and will be removed in the future.
Use alternative bundlers like "deno_emit", "esbuild" or "rollup" instead.
```

I have always hated the process of deploying JavaScript code to the
internet. npm creates massive balls of mud that are so logistically
annoying to tease out into actual real files that your browser can
load. The tooling around building JavaScript projects into real files
has historically been a trash fire of philosophical complexity and has
been the sole reason that I have avoided digging too deep into how
frontend development works.

<xeblog-conv name="Cadey" mood="coffee">Really though, it really sucks
that the way that Node.js development is so different than what
browsers actually support. I know that browsers getting import map
support can help with this, but it would be nice if that wasn't a
problem at all.</xeblog-conv>

I understand why the Deno team is getting rid of the `deno bundle`
command (and at some level, based on what I've learned it's actually
quite amazing that I have gotten this all working in browsers to begin
with), but just dumping the horrors of esbuild onto unsuspecting
people is logistically frustrating. Especially with how much you need
to hack at all of this.

<xeblog-conv name="Cadey" mood="coffee">I really don't want all of
this to sound negative and hateful. But really, from the perspective
of someone used to doing distribution packaging with languages that
normally compile to single binaries, the Node ecosystem is a huge pain
in the ass. At one time Debian tried to enforce a policy of every
separate dependency being compiled to its own unversioned package.
All this fell apart when they realized that different node packages
will depend on the unspoken behavior of separate transitive
dependencies. This has lead to them giving up and shipping vendored
`node_modules` folders.<br /><br />Can you see why I like Rust as a
distribution packager? I don't have to deal with any problems other
than making sure the binary builds and I can slap it in the package.
It is so _easy_ in comparison.</xeblog-conv>

However, I really don't want my builds to randomly start breaking at
some uncertain point in the future when I upgrade Deno. So, I got
bored and then decided to convert my entire build system over to
[esbuild](https://esbuild.github.io/). Here is the entire stack and
how I got it working for my builds.

<xeblog-hero ai="Ligne Claire v1" file="inspiration-vibes" prompt="1girl, green hair, green eyes, tshirt, sweatpants, long hair, full body, sitting, outside, landscape, chromatic aberration, smile, looking to the side, backpack, space needle"></xeblog-hero>

## Deno and esbuild

<xeblog-conv standalone name="Aoi" mood="wut">So if esbuild is the
thing you are supposed to be using, why would you not want to use
it? Are you not using NPM or something?</xeblog-conv>

You're right my [foxy friend](https://xeiaso.net/characters#aoi), I'm
not using NPM or even node.js at all. I'm using
[Deno](https://deno.land), which is an alternative
JavaScript/TypeScript runtime that is written in Rust and makes
dependency management so much easier. One of the main ways that it
makes dependency management easier is by having you pull packages from
URLs running normal static file servers instead of having to put
everything into NPM and then hope that NPM doesn't go down.

<xeblog-conv name="Cadey" mood="coffee">Yes, yes, this does mean that
you need to hope that the other fileservers hosting your dependencies
don't go down. However when Nix and the like is brought into the
situation later in the article, this becomes less of a
problem.</xeblog-conv>

When you install a package with Deno, it downloads all of the relevant
JavaScript and TypeScript files to somewhere on disk and stores them
based on their origin server and SHA256 checksum. This is very unlike
what NPM does. As a comparison, here's what the file tree for NPM
installing [Xeact](https://github.com/Xe/Xeact):

```
./node_modules/
`-- @xeserv
    `-- xeact
        |-- CODE_OF_CONDUCT.md
        |-- LICENSE
        |-- README.md
        |-- default.nix
        |-- jsx-runtime.js
        |-- package.json
        |-- shell.nix
        |-- site
        |   |-- gruvbox.css
        |   |-- index.html
        |   `-- index.js
        |-- types
        |   |-- jsx-runtime.d.ts
        |   `-- xeact.d.ts
        |-- xeact.js
        `-- xeact.ts
```

And here's what Deno's local file tree looks like:

<span id="deno-fs"></span>
```
/deno-dir/deps/
`-- https
    `-- xena.greedo.xeserv.us
        |-- 15c8dd50d4aede83901b65e305f1eca8dd42955da363aca395949ce932023443
        |-- 15c8dd50d4aede83901b65e305f1eca8dd42955da363aca395949ce932023443.metadata.json
        |-- 6291a9332210dc73f237e710bb70d6aab7f8cd66ea82cb680ed70f83374b34a3
        `-- 6291a9332210dc73f237e710bb70d6aab7f8cd66ea82cb680ed70f83374b34a3.metadata.json
```

You can see how this would give existing tooling a lot of trouble.

Luckily, [esbuild](https://esbuild.github.io) has support for
[plugins](https://esbuild.github.io/plugins/#using-plugins). These let
you override behavior like how esbuild looks for dependencies. There
is a [Deno plugin](https://deno.land/x/esbuild@v0.17.15) for esbuild,
but it is chronically under-documented. Here is how I got it working.

First, I added esbuild and the deno plugin to my import map:

```json
{
  "imports": {
    "@esbuild": "https://deno.land/x/esbuild@v0.17.13/mod.js",
    "@esbuild/deno": "https://deno.land/x/esbuild_deno_loader@0.6.0/mod.ts",
  }
}
```

<xeblog-conv name="Mara" mood="hacker">You can use any name you want
for this, but `@esbuild` and `@esbuild/deno` looks cool.</xeblog-conv>

Then write a file named `build.ts` with the following things in it:

```javascript
import * as esbuild from "@esbuild";
import { denoPlugin } from "@esbuild/deno";

const result = await esbuild.build({
  plugins: [denoPlugin({
    importMapURL: new URL("./import_map.json", import.meta.url),
  })],
  entryPoints: Deno.args,
  outdir: Deno.env.get("WRITE_TO")
    ? Deno.env.get("WRITE_TO")
    : "../../static/xeact",
  bundle: true,
  splitting: true,
  format: "esm",
  minifyWhitespace: !!Deno.env.get("MINIFY"),
  inject: ["xeact"],
  jsxFactory: "h",
});
console.log(result.outputFiles);

esbuild.stop();
```

<xeblog-conv name="Mara" mood="hacker">Don't forget the `esbuild.stop`
call. If you don't, the script will hang infinitely. This was "fun" to
discover on the fly.</xeblog-conv>

This will do the following:

- Configure esbuild to read from our Deno dependencies with the import
  map
- Sets all of the command line arguments to be the build inputs, so
  you can call it with `deno run -A build.ts **/*.tsx` and get it to
  magically build all of the files in `./components`
- Sets the output paths and if the output should be minified based on
  environment variables (used in the Nix build)

This builds everything correctly, and puts each component in its own
`.js` file where my site expects to serve it. This will be important
later.

## Nix

Now comes the fun part, making all of this work deterministically in
Nix so that I can inevitably forget how all of this works because it
all happens behind the scenes. When I build my site's frontend with
Nix, I use my fork of [deno2nix](https://github.com/Xe/deno2nix) to
automate the process of setting up a local copy of all the
dependencies my website depends on.

The deno2nix
[internal.mkDepsLink](https://github.com/Xe/deno2nix/blob/main/nix/internal.nix#L24)
function allows you to take a
[`deno.lock`](https://deno.land/manual@v1.32.3/basics/modules/integrity_checking)
file and turn that into a folder in the Nix store. This does all of
the hard parts of making a Deno build work in Nix. It converts the
`deno.lock` file into the folder structure that [Deno
created](#deno-fs).

There's only one small problem: I pull dependencies from
[esm.sh](https://esm.sh), which sometimes has you include files with
`@` (at-signs) in their paths. For example:

```
http://esm.sh/@xeserv/xeact@0.70.0
```

This would be pulled into the Nix store as
`/nix/store/if9bjhar81hhm7rqrlb4rfs65k2rwnp0-xeact@0.70.0`, which doesn't work because of this error:

```
error: store path 'if9bjhar81hhm7rqrlb4rfs65k2rwnp0-xeact@0.70.0' contains illegal character '@'
```

There's two ways of fixing this:

- Fix deno2nix so that it strips `@` from the basename (final path
  component) of URLs
- Take advantage of how `esm.sh` works to require from the _exact_
  files instead of the parent level re-exports
  
When you read from `esm.sh`, you get a file that re-exports the actual
NPM package like this for `http://esm.sh/@xeserv/xeact@0.70.0`:

```javascript
/* esm.sh - @xeserv/xeact@0.70.0 */
export * from "https://esm.sh/v114/@xeserv/xeact@0.70.0/es2022/xeact.mjs";
export { default } from "https://esm.sh/v114/@xeserv/xeact@0.70.0/es2022/xeact.mjs";
```

This means that you can just change the imports to the path
`https://esm.sh/v114/@xeserv/xeact@0.70.0/es2022/xeact.mjs` instead of
making it import the top-level `/@xeserv/xeact@0.70.0`, and this will
work because the basename is `xeact.mjs`, not `xeact@0.70.0`. This
will let it fit into the Nix store.

After changing over all the import paths to pull from exact files
instead of the top-level packages, `deno2nix` worked with the old
build flow. Now all that is left is running the `esbuild` wrapper.
After noodling for a while, I came up with this derivation:

```nix
frontend = pkgs.stdenv.mkDerivation rec {
    pname = "xesite-frontend";
    inherit (bin) version;
    dontUnpack = true;
    src = ./src/frontend;
    buildInputs = with pkgs; [ deno jq nodePackages.uglify-js ];
    ESBUILD_BINARY_PATH = "${pkgs.esbuild}/bin/esbuild";

    buildPhase = ''
        export DENO_DIR="$(pwd)/.deno2nix"
        mkdir -p $DENO_DIR
        ln -s "${pkgs.deno2nix.internal.mkDepsLink ./src/frontend/deno.lock}" $(deno info --json | jq -r .modulesCache)
        export MINIFY=yes

        mkdir -p dist
        export WRITE_TO=$(pwd)/dist

        pushd $(pwd)
        cd $src
        deno run -A ./build.ts **/*.tsx
        popd
    '';

    installPhase = ''
        mkdir -p $out/static/xeact
        cp -vrf dist/* $out/static/xeact
    '';
};
```

This uses the `internal.mkDepsLink` function to create everything we
need, minifies the output, writes it all to a folder named `dist` and
finally plunks everything into `$out/static/xeact`, such as with
[`MastodonShareButton.js`](/static/xeact/MastodonShareButton.js).

It worked, and I was so relieved when it did.

## Migration

Now that I had the ability to build all of my dynamic components, I
had to take a moment to design something I've wanted to make for a
while, the Xeact Component Model. At a high level, Xeact is built for
stateless components. These components are functionally identical as
React components: functions that take in properties and turn them into
HTML nodes.

<xeblog-conv name="Mara" mood="hacker">In more computer science
terminology, we can call these monomorphisms. Monomorphisms are
functions that take one argument and produce one result, such as `(x)
-> x + 1` in JavaScript. State really muddies up the waters, but let's
not think about that for now.</xeblog-conv>

Here's an example Xeact component, the one that handles the "No fun
allowed" button for talk pages:

```javascript
import { c } from "xeact";

const onclick = () => {
  Array.from(c("xeblog-slides-fluff")).forEach((el) =>
    el.classList.toggle("hidden")
  );
};

export default function NoFunAllowed() {
  const button = (
    <button
      class=""
      onclick={() => onclick()}
    >
      No fun allowed
    </button>
  );
  return button;
}
```

This creates a function called `NoFunAllowed` that shows a button that
says `"No fun allowed"`. When a user clicks on it, it toggles the
"hidden" class on every element with the CSS class
`xeblog-slides-fluff`. When I write talks I usually use my slides as
tools to help me visually explain what's going on. Combined with a
healthy dose of surrealism, this means that some people may find the
written forms of my talks jarring due to all of the tweet-length
paragraphs combined with memes and absurdism, such as what is quite
possibly my favorite slide I've ever made:

<xeblog-slide essential name="2023/wazero-lightning/14"></xeblog-slide>

<xeblog-conv name="Cadey" mood="coffee">At some level, I really get
that people don't like this. It's slightly frustrating to me as an
artist to know that so many people don't really see the value in the
art that I pour so much effort into, but I get it. Humor is hard.
Abstract humor is harder. Abstract humor about abstractions stapled on
top of abstractions is something that many people will miss. I enjoy
using absurdist/surrealist humor to really communicate the inherent
surrealism of our profession to people. For many people it's just
fundamentally a swing and a miss. I'll live.</xeblog-conv>

It was a simple
copy/paste/[`useState`](https://xeiaso.net/blog/xeact-0.70.0-useState)
job to migrate over all of the other dynamic components. Then came
wiring it up on the server side.

## The server

Let's go back to what Xeact components really are: functions that take
attributes and return HTML nodes. When I write the HTML for my
server-side components, I really am writing things like this in the
markdown:

```html
<xeblog-conv name="Mimi" mood="coffee">As a large language model, I
can serve to provide some example text. I don't know what "Hipster
Ipsum" is. But the Lorem Ipsum text...</xeblog-conv>
```

This gets expanded into what you see in the document with `lol_html`:

<xeblog-conv name="Mimi" mood="coffee">As a large language model, I
can serve to provide some example text. I don't know what "Hipster
Ipsum" is. But the Lorem Ipsum text...</xeblog-conv>

So at some level I need to do the following things to support
Xeact components:

- Create a HTML wrapper that imports the component and places it into
  the HTML tree with a unique UUID
- Have some kind of `<noscript>` tag to warn people that they need to
  have JavaScript enabled
- A little hacky JavaScript that imports the component and executes it
  with JSON passed from Rust
  
I think I have most of this with my
[`xeact_component`](https://github.com/Xe/site/blob/d4211559c10b6e8c4b647759a15da5ac59e1f144/lib/xesite_templates/src/lib.rs#L185)
template. It creates a unique ID (UUIDv4), serializes data from Rust
into JSON so that it can be used as inputs to Xeact components, and
then slaps the results of the component function into the element
tree.

I used this basic process to port over my other dynamic components
such as the video player:

<xeblog-video path="blog/2023/returnal-preview"></xeblog-video>

<xeblog-conv standalone name="Cadey" mood="enby">I have plans to write
a review about Returnal when I finish it. It's an absolute
masterpiece.</xeblog-conv>

My hope is that this will make it easier for me to maintain and expand
on the other components on my website. Eventually I want to make a
HTML tag that is like `<xeact-component xeact_filename="Thing"
foo="bar">`, and then have all the rest of the stack do the right
thing, but that will take a bit more creativity than I can muster
right now.

I am really happy that this all works, not only will this make it
easier for me to run my website, extending it in the future should be
even easier.

<xeblog-conv name="Cadey" mood="coffee">This is starting to break my
"no structural JavaScript" rule that I put on myself, but
realistically this is the direction the entire ecosystem is going
anyways. I don't like that this will reduce compatibility with other
browsers like LibWeb from SerenityOS, but I need to have access to
fancy toys to be able to experiment.</xeblog-conv>

I would love to reuse this logic in
[waifud](https://github.com/Xe/waifud) eventually. Its admin panel is
also vulnerable to the same eventual breakage, and I suspect that I
will inevitably reuse this build logic there too. Not to mention the
Xeact component model and `useState` allowing me to simplify a lot of
the logic there too.
