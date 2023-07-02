---
title: "JSX is quasi-quoting"
date: 2023-01-23
tags:
 - JSX
 - JavaScript
 - Lisp
---

I've been writing a fair bit of JSX/TSX code lately and something has felt oddly
familiar about that programming model. It was something that I couldn't really
place until I had a breakthrough after hacking at my Emacs config again. When
you are using JSX to write HTML in your JavaScript functions, you are using
quasi-quoting.

<xeblog-conv standalone name="Aoi" mood="wut">Quasi-quoting? What's
that?</xeblog-conv>

<xeblog-hero ai="Waifu Diffusion" file="sky-sigils" prompt="glowing sigils, sigils, zen, yin yang, taoism, landscape, world trade center, peaceful, arknights, scifi, runic energy, spellcraft"></xeblog-hero>

I think one of the easiest ways to explain this is to use Emacs Lisp. Emacs Lisp
is the extension language for [GNU Emacs](https://www.gnu.org/software/emacs/),
the text editor that I ~~am horribly addicted to using~~ have used for the last
ten years.

One of the major concepts in Lisp is that code is data, and data is code. When
you are writing in Lisp, you are writing linked lists that the computer
interprets as code. For example, consider this small bit of Lisp:

```lisp
(quote (+ 3 4))
```

You can evaluate this with `ielm` in Emacs by using `M-x ielm`:

<xeblog-conv standalone name="Mara" mood="hacker">`M-x` is Emacs-speak for
"alt-x".</xeblog-conv>

```
*** Welcome to IELM ***  Type (describe-mode) for help.
ELISP> (quote (+ 3 4))
(+ 3 4)
```

Quoting code into data is something you do very often in Lisp, so there's a
shortcut for doing this using the single quote character `'`:

```
ELISP> '(+ 3 4)
(+ 3 4)
```

This works great, but sometimes you need to put the value of a variable into a
bit of data. Let's say you have this snippet of Lisp code:

```lisp
(let ((filename "foobar.txt"))
     '(filename))
```

<xeblog-conv standalone name="Mara" mood="hacker">`(let ((var1 value) (var2
value)) code)` is how you declare temporary variables in Lisp. Here the variable
name `filename` is set to `"foobar.txt"`. Each variable declaration is a list of
two elements: the variable name and its value. Values can be data or code that
is evaluated down to data.</xeblog-conv>

If you put this into the Emacs Lisp interpreter, you won't get back what you
think:

```lisp
ELISP> (let ((filename "foobar.txt"))
            '(filename))
(filename)
```

You need to have some way to quote code you want to be data, and then some way
to unquote data back into code. Luckily, Emacs Lisp lets you do this with a
construct they call
[Backquoting](https://www.gnu.org/software/emacs/manual/html_node/elisp/Backquote.html):

```lisp
ELISP> (let ((filename "foobar.txt"))
            `(,filename))
("foobar.txt")
```

The backtick <code>`</code> lets you quote everything like the single quote
<code>'</code>, but you can also _unquote_ data using the comma <code>,</code>
to turn your data back into code. This lets you construct complicated data
structures like an attribute list to convert into HTTP form data:

```lisp
ELISP> (let ((fname (buffer-name))
             (content "Hi there"))
            `((fname . ,fname)
              (content . ,content)))
((fname . "*ielm*")
 (content . "Hi there"))
```

<xeblog-conv name="Aoi" mood="cheer">So quasi-quoting lets you mix data and
code, but how does this relate to JSX?</xeblog-conv>
<xeblog-conv name="Mara" mood="aha">JSX is the same thing with slightly
different syntax.</xeblog-conv>

[JSX](https://reactjs.org/docs/introducing-jsx.html) is a syntax extension for
JavaScript that lets you mix HTML data with JavaScript code. It's a lot like
quasi-quoting in Lisp. Consider this small block of JSX code:

```typescript
const name = "Aoi";
const header = (
  <div>
    <h2>{name}</h2>
    <p>Hi there, {name}! How are you doing today?</p>
  </div>
);
document.write(header);
```

This writes the equivalent of this HTML to the current page:

```html
<div>
  <h2>Aoi</h2>
  <p>Hi there, Aoi! How are you doing today?</p>
</div>
```

You quote HTML data inside parentheses `()` and use curly braces `{}` to unquote
JavaScript code into HTML data.

<xeblog-conv name="Aoi" mood="grin">So JSX does the same thing for
HTML in JavaScript that quasi-quoting does for lists in Lisp! It lets you mix
code and data so that you can assemble whatever you want easily.</xeblog-conv>
<xeblog-conv name="Mara" mood="happy">Yep! You end up finding a lot of these
things across different programming tools. A lot of tools steal ideas from
eachother and there are many more similar patterns across the industry. What
other ones can you find?</xeblog-conv>
