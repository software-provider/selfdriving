---
title: "Xeact 0.70.0: now with the useState hook"
date: 2023-04-03
tags:
 - jsx
 - react
 - frontend
 - javascript
series: xeact
---

Xeact continues to dominate the front-end ecosystem. Every facet of
the industry has been forever changed by Xeact, and we are all better
for it. Today I have the momentous pleasure of introducing the newest
version of Xeact: version 0.70.0. This allows you to track stateful
values using the new `useState` hook.

<xeblog-hero ai="Anything v3" file="solo-journey" prompt="1girl, green hair, hoodie, outdoors, breath of the wild, space needle, walking, long hair, highly detailed, futuristic"></xeblog-hero>

To help illustrate the point, I have copied the entire source code of
the useState function below:

```javascript
/**
 * Allows a stateful value to be tracked by consumers.
 *
 * This is the Xeact version of the React useState hook.
 *
 * @type{function(any): [function(): any, function(any): void]}
 */
const useState = (value = undefined) => {
  return [() => value, (x) => {
    value = x;
  }];
};
```

Mimi, would you care to explain this?

<xeblog-conv name="Mimi" mood="coffee">This code defines a function
called `useState` that returns an array with two elements. The first
element of the array is a function that takes no arguments and returns
the initial value passed in as a parameter to the `useState` function
or `undefined` if no value is given. This function acts as a getter
for the current state value. The second element of the array is a
function that takes a single argument and sets the value of `value`.
This function acts as a setter for the state value. This code is
implementing a simplified version of the `useState` hook in React, a
commonly used library in JavaScript for building user
interfaces.</xeblog-conv>
<xeblog-conv name="Aoi" mood="wut">Why would I want to use this
instead of normal variables?</xeblog-conv>

The main reason you want to use a stateful hook like this is when you
write more complicated components, like the [Mastodon share
button](https://github.com/Xe/site/blob/7b691babb3312d9c2284416f03b50de34638bc58/src/frontend/components/MastodonShareButton.tsx)
at the bottom of every post. At a high level, pulling values from HTML
elements requires you to either declare each element in variables and
then assemble the component [like this waifud admin panel
component](https://github.com/Xe/waifud/blob/aa7d983fb5c703a623f4fadf24e17fd4f531a688/frontend/instance_create.tsx#L25):

```javascript
// SomeForm.jsx

export default function SomeForm() {
  const input = <input type="text" placeholder="words" />;
  
  return (
    <div>
      {input}
      <button onClick={(e) => alert(input.value)}>Alert</button>
    </div>
  );
}
```

This creates an input box and a button. When you click on the button,
it turns whatever you put into the box into an alert.

However, we can do better.

The `useState` hook in React allows you to associate components with
stateful values, so you can write out all the code like this:

```javascript
// SomeForm.jsx

import { useState } from "react";

export default function SomeForm() {
  const [msg, setMsg] = useState("");
  
  return (
    <div>
      <input
        type="text"
        placeholder="words"
        onInput={(e) => setMsg(e.target.value)}
      />
      <button onClick={(e) => alert(msg)}>Alert</button>
    </div>
  );
}
```

As you can see, this lets you have the state be updated in the
`oninput` handler of the `<input>` box and then used in the `onclick`
handler of the button.

The `useState` Xeact hook also lets you do this, but with one
significant difference: updating the value in the state container
_does not_ trigger a redraw of the relevant components that use those
values. This does limit the usefulness of the Xeact `useState`
container, but I bet that I'll figure out a way around this should
this ever become relevant.

<xeblog-conv name="Aoi" mood="coffee">Or you could just use
[preact](https://preactjs.com) like a normal person...</xeblog-conv>
<xeblog-conv name="Mara" mood="happy">Doing that would not only make
sense, that would mean we need to be more inventive for the
blog!</xeblog-conv>
<xeblog-conv name="Aoi" mood="facepalm">You people astound
me.</xeblog-conv>

Oh, and the state reader is a function instead of a value:

```javascript
// SomeForm.jsx

import { useState } from "xeact";

export default function SomeForm() {
  const [getMsg, setMsg] = useState("");
  
  return (
    <div>
      <input
        type="text"
        placeholder="words"
        onInput={(e) => setMsg(e.target.value)}
      />
      <button onClick={(e) => alert(getMsg())}>Alert</button>
    </div>
  );
}
```

Either way, I suspect that this will propel Xeact towards its goal of
getting many GitHub stars. I've incorporated this state hook into my
blog and will write more about it when I've gotten more practical
experience with it.

Thank you for following the development of Xeact! There's sure to be
more in the future as I figure out what the hell I am supposed to be
doing. I'm also starting to realize that I don't suck at frontend
development, what I really suck at is design, which is what manifests
as feeling like you suck at frontend development.

I'm sure I'll figure it out. Gotta burn sticks to make fire.
