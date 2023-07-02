---
title: The GraphicalEmoji hack
date: 2023-03-08
tags:
 - cursed
 - JavaScript
 - JSX
 - fontRendering
 - dearGodHelpMe
---

Today, I tried to write a JSX component that included an emoji in it.
Specifically this emoji: <span style="font-family: Times New Roman">⚠️</span>.

This emoji is special because there's actually two forms of it:

- &#x26A0;&#xFE0E; (the textual form)
- <span style="font-family: Times New Roman">⚠️</span> (the graphical
  form)

EDIT: Apparently this difference isn't showing up on every browser
engine the same way. Please trust me that there is a difference, one
of them on my MacBook running Microsoft Edge is a text-only emoji that
has no color in it. This is why I was so confused, scared, and on the
verge of tears after being gaslit by my browser. God is dead because
font rendering killed him.

When I was making this component, I wanted the graphical form of it.
The following things did not work:

- Adding the explicit "make this graphical" Unicode instruction:
  `\u{FE0F}`
- Declaring the emoji as the string `"\u{26A0}\u{FE0F}"` and then
  using it as a variable: `<span>{warningEmoji}</span>`
- Using the variable in a format string:
  <code>&lt;span&gt;{&grave;${warningEmoji}&grave;}&lt;/span&gt;</code>

Turns out, this is actually a fairly widespread problem with fonts
that have the _textual_ form of emoji defined but not the _graphical_
form of it defined. The font my blog uses is one of them, so to get
the graphical <span style="font-family: Times New Roman">⚠️</span> I've
been using above, I had to paste this HTML snippet:

```html
<span style="font-family: Times New Roman">⚠️</span>
```

<xeblog-conv standalone name="Aoi" mood="facepalm">Oh god. Really?
That is so, so cursed.</xeblog-conv>

Yes, really. In order to make the emoji render correctly, I had to
instruct the browser to render it in 
<span style="font-family: Times New Roman">Times New Roman</span>
because that _does not_ have the emoji defined. It will then fall back
to the system font, giving us the
<span style="font-family: Times New Roman">⚠️</span>
that we truly desire.

Here is the JSX component I had to write:

```js
export interface GraphicalEmojiProps {
  emoji: string;
}

/** Listen to me for my tale of woe:
 * Fonts are complicated. Fun fact: fonts are actually Turing-complete programs
 * that run in browsers. Yes, font rendering is really that complicated. This
 * component is a dirty, ugly, disgusting HACK that works around font
 * rendering in order to forcibly display the graphical form of an emoji.
 *
 * This works because times new roman always displays the graphical forms of
 * emoji. No, I don't know why either. It slightly scares me.
 *
 * Either way, this works and I'm not brave enough to question why.
 */
export default function GraphicalEmoji({ emoji }: GraphicalEmojiProps) {
  return <span style={{ fontFamily: 'Times New Roman' }}>{emoji}</span>;
}
```

This code is free as in mattress. If you decide to use it, it's your
problem.

<xeblog-conv standalone name="Cadey" mood="coffee">I hate fonts.</xeblog-conv>

