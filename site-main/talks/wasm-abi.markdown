---
title: The Go WebAssembly ABI at a Low Level
date: 2022-10-17
slides_link: "https://drive.google.com/file/d/1RKitNYC77AYnsstNsYvJcBNnaKT06stb/view?usp=sharing"
tags:
 - wasm
 - golang
 - go
 - ieee754
---

<xeblog-talk-warning></xeblog-talk-warning>

<xeblog-video path="talks/golab-wasm"></xeblog-video>

<xeblog-conv name="Mara" mood="hacker">If that doesn't load, try viewing the
version on [YouTube](https://youtu.be/y-RYxMB4xFE).</xeblog-conv>

This talk was presented at [GoLab 2022](https://golab.io/) in Florence, Italy as
a remote talk. It was fully scripted using a conversational style and
prerecorded ahead of time.

## Talk

<xeblog-slide name="golab-wasm/slides/001" essential></xeblog-slide>

For over a decade, the dominant language for developing applications in browsers
has been JavaScript. Everyone in this room either deals with or touches
something in the world that deals with JavaScript. Recently the Worldwide Web
Consortium published a standard called WebAssembly that is one of the first
steps towards JavaScript no longer being a requirement for developing things
targeting web browsers.
  
The Go 1.11 release in mid-2018 added support for compiling Go to WebAssembly.
It allows you to take a reasonable subset of Go programs and run them in
browsers alongside JavaScript.
  
I'm Xe Iaso and today I'm going to help you understand how this works and the
amazingly terrible hacks that power the core of this. This talk is aimed at
intermediate to expert audiences, it will likely hit best if you have *some*
familiarity with JavaScript and WebAssembly, and especially if you are a fan of
amazingly terrible ideas. To make sure everyone is on the same page, I'm going
to give background and context for everything as it is needed.
  
So come on and join me on this magical journey through calling conventions,
I-triple-E seven fifty-four floating point numbers and more as we learn the
nitty gritty of how Go's WebAssembly support works!

<xeblog-slide name="golab-wasm/slides/002"></xeblog-slide>

As it says on the tin, I'm Xe Iaso. I'm the Archmage of Infrastructure at
Tailscale and I am regularly accused of being an expert in both Go and
WebAssembly. I have an extensive background in development and site reliability
things, but I've been doing developer relations recently.
  
So you are aware, this talk is going to contain opinions about many topics.
These opinions are my own and are not the opinions of my employer.

<xeblog-slide name="golab-wasm/slides/003"></xeblog-slide>

WebAssembly is a specification that defines a bunch of semantics about how a
computer that doesn't exist should work. It defines the virtual machine, how the
stack works, the instructions the machine can run, the format that compilers
should target, and other fiddly details like that. In practice it is somewhere
between native code and a scripting language (much like Java class files), but
at a high level we can think about it like this:

<xeblog-slide name="golab-wasm/slides/004"></xeblog-slide>

WebAssembly itself is a computer that takes your code and executes it.
Inherently it will run any functions you want, store the results in its linear
memory or return values from the stack; but otherwise it's a glorified
reverse-polish-notation calculator that runs very fast.
  
With this you can do simple math all day, but that's not overly useful in the
real world by itself.

<xeblog-slide name="golab-wasm/slides/005"></xeblog-slide>

The real magic for how WebAssembly gets useful comes from external functions
that get imported into WebAssembly-land. These functions can do just about
anything you want from "making an HTTP request with the JavaScript fetch()
function" to "read from and write to local storage".

<xeblog-slide name="golab-wasm/slides/006"></xeblog-slide>

WebAssembly was designed to run very fast on consumer hardware. Is binary format
was designed to be easy to parse, and WebAssembly instructions can easily
compile down to machine code with very little effort in real time.
  
Overall, this lets you have your WebAssembly program do whatever you want,
access whatever it needs and can overall be as powerful as JavaScript. There are
only a few caveats related to performance and translation between the two
worlds, much like the caveats with system calls on Unix. It's really nice in
practice.

<xeblog-slide name="golab-wasm/slides/007"></xeblog-slide>

However, let's take a look at that "functions imported from the environment"
thing a bit closer. The WebAssembly specification only defines the *virtual
machine*, how code is stored into and loaded from dot WASM files, and the
semantics of how all that works. It doesn't specify an API that programs written
to target WebAssembly can use to talk to the outside world.
  
Given the constraints of the WebAssembly team at the time, it's very reasonable
that they didn't try to also shove a stable interoperability API into the mix
when trying to get the minimum viable product out of the door. That could have
taken years. But, as a side effect of this, everyone has had to invent their own
one-off APIs for gluing the two sides together.

<xeblog-slide name="golab-wasm/slides/008"></xeblog-slide>

Oh and to make things even more fun, WebAssembly has no native string type, just
like C! All there is are contiguous blocks of ram terminated by null characters.
Just like our friend, the PDP-11.

<xeblog-slide name="golab-wasm/slides/009"></xeblog-slide>

WebAssembly was originally intended for use in browsers, but there have been
efforts to standardize on an API for WebAssembly programs to function on server
environments. This allows operators to run arbitrary code from users and also
take advantage of the inherent isolation features WebAssembly brings to the
table. WASI (short for WebAssembly System Interface) is an independent standard
that gives WebAssembly programs some Unix-y calls, but overall we are talking
about a browser here, not a Unix system.
  
Go's WebAssembly support also was made before WASI even came out, so at the time
it wasn't a viable option. WASI also doesn't support system calls like "open
network socket", which makes it logistically annoying for writing many
real-world applications. As far as I am aware Go doesn't support WASI at all.

<xeblog-slide name="golab-wasm/slides/010"></xeblog-slide>

There is more than one Go compiler though. TinyGo is a Go compiler built on top
of LLVM that can compile a subset of Go programs to WebAssembly with WASI. I
want to reiterate that the Go WebAssembly port mostly targets browsers, not Unix
systems. Those two are different beasts entirely. For the sake of keeping things
simple in this talk, I'm going to focus on how Google's Go compiler does all of
this.

<xeblog-slide name="golab-wasm/slides/011" essential></xeblog-slide>

So with all of those caveats in mind, it's reasonable to wonder something like
"Why would I even use this in the first place? It's a brand new compiler port
with brand new platform semantics that I have to invent myself.".
  
That's a reasonable thing to conclude, however I counter with these points:

<xeblog-slide name="golab-wasm/slides/012"></xeblog-slide>

Sometimes the one library call you need in JavaScript but have in Go doesn't
exist and you really don't want to have to make an API call for it. WebAssembly
is the only officially sanctioned way to do this.
  
Previously, there was a community effort to compile Go to JavaScript called
GopherJS, but that has fallen out of favour as the WebAssembly port for Go gets
more and more mature.

<xeblog-slide name="golab-wasm/slides/013"></xeblog-slide>

Doing this also lets you run the same code in the same language on both your
browser and servers, which can help reduce cognitive complexity as you switch
between issues on the frontend and backend.

<xeblog-slide name="golab-wasm/slides/014"></xeblog-slide>

It's also new and fun! You're all programmers, right? You know just as well as I do that we have a hard time resisting the siren song of new things.

<xeblog-slide name="golab-wasm/slides/015"></xeblog-slide>

Here are some notable places where you can use Go's WebAssembly port  in order
to get things done.

<xeblog-slide name="golab-wasm/slides/016"></xeblog-slide>

You can embed your already existing peer to peer VPN engine into a browser so
you can SSH into production from a webpage.

<xeblog-slide name="golab-wasm/slides/017"></xeblog-slide>

You can embed the new netip package into your JavaScript applications so you can
do advanced subnet calculations in order to make setting up networks faster.

<xeblog-slide name="golab-wasm/slides/018"></xeblog-slide>

You can write full featured web applications without having to write a lick of
JavaScript.

<xeblog-slide name="golab-wasm/slides/019"></xeblog-slide>

Another place Go's WebAssembly port has been used is as part of the process of
porting over the game "Bear's Restaurant" to the Nintendo Switch. The team made
it work in the WebAssembly port, then had a bunch of custom scripts recompile
that blob of WebAssembly to C++ and then wrapped the input and output layers to
the proprietary APIs that the Nintendo Switch uses.
  
As far as I know, "Bear's Restaurant" is the first commercially released game
that uses Go in any way on actual game console hardware.

<xeblog-slide name="golab-wasm/slides/020"></xeblog-slide>

With all this in mind, you can see how it would be hard to write an API that
would let you do anything you want in the browser with JavaScript like you were
writing native JavaScript code. It's a lot to consider because there is frankly
a lot going on. Computers are surprisingly complicated.

<xeblog-slide name="golab-wasm/slides/021" essential></xeblog-slide>

The Go standard library has a package called `syscall/js`. This defines the
system call API to a bunch of JavaScript code (included with every release of
Go) that helps bridge the gap between  WebAssembly and JavaScript. You can focus
on writing your code in Go and let the system call layer handle the rest.

<xeblog-slide name="golab-wasm/slides/022"></xeblog-slide>

This works by giving you references to JavaScript objects and then also gives
you a set of calls to manipulate them however you want. This will let you do
most of what you can to do JavaScript objects in your Go code.
  
These references are opaque handles to objects outside of the program, just like
file descriptors are opaque handles to kernel objects in Unix.

<xeblog-slide name="golab-wasm/slides/023"></xeblog-slide>

Oh and for extra fun, all of the object references are NaN values.

<xeblog-slide name="golab-wasm/slides/024"></xeblog-slide>

Yes, really. There is more than one NaN (not-a-number) value in floating point
logic. There's actually many more than you'd think possible. You know what,
let's take a moment to learn about how numbers work in computers so we all can
understand how utterly elegant this hack is.

<xeblog-slide name="golab-wasm/slides/025"></xeblog-slide>

As humans, we usually deal with numbers in what we call "base 10" or "decimal".
There are ten options for each digit. As digits go farther to the left on
numbers, those digits signify bigger and bigger values. Let's think about the
number four-hundred twenty-six:

<xeblog-slide name="golab-wasm/slides/026" essential></xeblog-slide>

This number is broken up into digits that correspond to different values. There
are four hundreds, two tens and six ones. Four-hundred twenty-six (426).

<xeblog-slide name="golab-wasm/slides/027"></xeblog-slide>

However, this only covers whole numbers. Many times we will deal with fractional
parts of a whole, such as with making exact change to two decimal points with
coins. Our number system expands to handle this too by adding columns for
tenths, hundredths and so on.

<xeblog-slide name="golab-wasm/slides/028" essential></xeblog-slide>

If we think about the number four-hundred twenty-six point three five, we can
also break it down like we did before. There are four hundreds, two tens and six
ones, and the three tenths and five hundredths come after the decimal point. 

<xeblog-slide name="golab-wasm/slides/029"></xeblog-slide>

That's how us humans deal with numbers. One of the weird things about the sand
we cursed into thinking is that sand deals with numbers in completely different
ways to humans. Our current computers deal with states that are either
completely on or completely off. With some conversion, you can use this to
express all the same mathematical operations as with decimal arithmetic, but
with two digit options instead of ten. We call this "base 2" or "binary"
mathematics.

<details class="warning">
  <summary>This slide was cut from the recording for time constraints</summary>

<xeblog-slide name="golab-wasm/slides/030"></xeblog-slide>

We call this "binary" because it's actually two words smashed together. "Bi"
means two and "ary" is short for the word "airity", which refers to the number
of arguments. Two-arguments, binary.

</details>

<xeblog-slide name="golab-wasm/slides/031" essential></xeblog-slide>

Instead of going by tens, each binary digit goes up by twos. The first digit is
the ones digit, the second is the twos digit, the third is the fours digit, the
fourth is the eights digit, et-cetera.
  
As a cheeky example, consider the base 10 number two-hundred and fifty-five. As
the diagram shows it's got 8 bits set. One for the 1's, the 2's, the 4's, the
8's, the 16's, the 32's, the 64's and the 128's. You can add all those
components up and get the total, 255.
  
Math operations work the same as you'd expect in binary. You just deal with twos
instead of tens.

<xeblog-slide name="golab-wasm/slides/032"></xeblog-slide>

But then we get back to the problem of fractional components in numbers. The
system I just described works great for whole numbers, but fractional components
get a bit messy. You could imagine just slapping on a binary point somewhere and
doing some hacks to call it a day (and I imagine that older computers did just
that to save time in development), but it's the future and we have a standard
for this called I-triple-E 754.

<xeblog-slide name="golab-wasm/slides/033"></xeblog-slide>

IEEE-754 is the de-facto standard for expressing numbers with fractional
components, or floating-point numbers. It defines the binary form of these
numbers for use in computers. It was first defined in 1985 by the Institute of
Electrical and Electronics Engineers, or I-triple-E. This standard was designed
to help make it easier to implement and use code that uses floating-point
numbers by defining the semantics so that electrical engineers could implement
them in hardware.
  
Every major programming language, CPU and GPU made in the last thirty years or
more supports I-triple-E 754 floating point. It's also notably used by Go,
WebAssembly, and JavaScript. This means that you can pass floating point numbers
from JavaScript into your Go functions compiled into WebAssembly.

<xeblog-slide name="golab-wasm/slides/034"></xeblog-slide>

As an aside, for the rest of this bit I'm going to be using the 16 bit encoding
for floating point numbers to make my diagrams easier to understand. Natively,
JavaScript uses 64 bit floating point numbers. Just imagine that there's more
bits.

<xeblog-slide name="golab-wasm/slides/035" essential></xeblog-slide>

One of the cool parts about how this all was implemented is that floating point
numbers are essentially scientific notation. You have a sign bit to tell if the
number is positive or negative, an exponent of two, and the mantissa that you
multiply. This lets you express numbers like two point one two five as the
scientific notation form of two to the power of one times 1.0625. The exponent
is one and the mantissa is 1.0625.

<xeblog-slide name="golab-wasm/slides/036"></xeblog-slide>

So with all this in mind, you'd probably wonder what the result of zero point
three minus zero point two is. First we need to convert these to floating point
numbers:

<xeblog-slide name="golab-wasm/slides/037" essential></xeblog-slide>

One of the first gotchas we will run into is the fact that we can't get an exact
replica of zero point three and zero point two in floating point numbers.
  
This is scientific notation, scientific notation gives you an *approximation* of
what the number is. The approximations add up, and the end result is that zero
point three minus zero point two is NOT zero point one in JavaScript, Go or most
other computer programming languages. You get zero point zero nine nine nine
et-cetera.

<xeblog-slide name="golab-wasm/slides/038"></xeblog-slide>

However, if you round this up to two decimal places, you do actually get zero
point one. So there is that.

<xeblog-slide name="golab-wasm/slides/039"></xeblog-slide>

One of the other things in I-triple-E 754 floating point numbers is an explicit
encoding for things that *are not* numbers, like infinity.

<xeblog-slide name="golab-wasm/slides/040" essential></xeblog-slide>

All you have to do is set all of the exponent bits and leave none of the
mantissa bits set. That gets you positive infinity.

<xeblog-slide name="golab-wasm/slides/041" essential></xeblog-slide>

If you flip the sign bit, you get negative infinity.

<xeblog-slide name="golab-wasm/slides/042" essential></xeblog-slide>

And if you set any of the other mantissa bits, you get a Not-a-Number value,
also known as NaN. The Go to JavaScript interoperability uses NaN-space numbers
to encode object ids in the same way that Unix uses numerical file descriptors
to encode kernel objects.
  
With a 64 bit floating point number, this gives the Go to JavaScript bridge
something hilarious like 4.5 quadrillion (ten to the power of fifteen) possible
object IDs.

<xeblog-slide name="golab-wasm/slides/043" essential></xeblog-slide>

With a simple bitwise exclusive or (xor) on the exponent bits, you can extract
the NaN space number into a normal integer that the JavaScript side uses to
address objects it knows about.

<xeblog-slide name="golab-wasm/slides/044"></xeblog-slide>

The reason why you'd want to do this has to do with an absurdly ugly hack that
has been baked into the core of nearly every JavaScript engine.
  
They use NaN values as object IDs because then the object IDs can fit in a
machine register. This means that you can pass JavaScript object IDs as register
values to functions and then the function can look up things on it if it
actually needs to care. If it doesn't, the only thing that's copied around is
the very small object ID. NaN values also have a fast path in most CPU floating
point units, making this faster than you'd expect. Computers are very fast at
copying things, but it adds up when you do it a lot.
  
As above, so below, eh?

<xeblog-slide name="golab-wasm/slides/045"></xeblog-slide>

The main thing to take away is that the numbers encoded into NaN values are used
as object IDs. It's a horrifying wrapper that is faster in practice because CPUs
are lazy. A NaN value is not a number, but it can contain a number.

<xeblog-slide name="golab-wasm/slides/046"></xeblog-slide>

If you want to learn more about this, I really do suggest checking out jan
Misali's video ["how floating point works"](https://youtu.be/dQhj5RGtag0). It
covers all of this in so much more detail, including how you would go about
deriving the entire floating point number system from scratch.

Numbers are weird, eh?
  
With all that light thinking out of the way, let's focus on something more exciting. Like calling conventions.

<xeblog-slide name="golab-wasm/slides/047"></xeblog-slide>

When you are writing programs in machine language, sometimes you want to take
common bits of code and reuse them. We can call these bits of code "functions".
At a high level they need to get arguments somehow, return a result somehow, and
figure out how to go back to where the function was called so that the program
continues to work like normal.

<xeblog-slide name="golab-wasm/slides/048"></xeblog-slide>

A famous example of this is in the game Super Mario Brothers for the Nintendo
Entertainment System. Every 21 frames the game will call a function that checks
to see if the level is cleared or not. When that function gets called, if it
doesn't explicitly return to where it was called from then the NES will continue
to execute code after that function. This will probably not do what the
developers of the game intended. It will most likely make the NES crash, which
is not good.

<xeblog-slide name="golab-wasm/slides/049" essential></xeblog-slide>

So to work around that, there's some semantics described in long, boring
documents that specify the conventions of how you call functions. These
conventions spell out how arguments and return values work, the assumptions you
should make about CPU registers and other intermediate state like stack hygiene,
and how data is stored in memory.

<xeblog-slide name="golab-wasm/slides/050"></xeblog-slide>

As a fun aside, there are cases when a function is called that has more
parameters than the CPU has registers. In that case the remaining arguments
would be pushed to the CPU stack (or somewhere else in memory that the function
assumes it should read from). Sometimes you have to make things a little more
complicated to cope with edge cases.

<xeblog-slide name="golab-wasm/slides/051"></xeblog-slide>

Before Go 1.17, Go had a stack-based calling convention for most of its targets,
modelled after Plan 9 from Bell Labs. When you called Go functions, it put those
arguments on the stack, made room for the return parameters, and then told the
CPU to jump to the function in question. That function would pop the things it
needed off of the stack, do what it needs to and then return any results on the
stack.
  
This is technically a bit slow because the stack is stored in system memory, but
realistically computers are pretty darn fast so it mostly works out, mostly.

<xeblog-slide name="golab-wasm/slides/052"></xeblog-slide>

WebAssembly is a stack-based virtual machine on the inside. This means that the
calling convention for WebAssembly functions is a bit similar to reverse Polish
notation:

```lisp
(i32.const 1)
(i32.const 1)
(i32.add)
```

To add two numbers in WebAssembly, you push them to the stack and issue the add
instruction. The add instruction takes those two numbers off of the stack, adds
them and pushes the result back into the stack. WebAssembly functions are called
in the same way. Push arguments to the stack, pop results from the stack.

<xeblog-slide name="golab-wasm/slides/054"></xeblog-slide>

The interesting part about how WebAssembly is specified is that the stack is
*external* to WebAssembly linear memory. This means that there is no "stack
pointer" in WebAssembly because the stack doesn't exist inside WebAssembly.
Functions can manipulate the stack by pushing to it and popping from it, but
they can't actually move it around. 
  
I'm pretty sure they designed it this way so that people could write WebAssembly
with multiple linear memory spaces. Amusingly enough this does also mean that
you can create a WebAssembly module that has *zero* linear memory spaces. I am
not aware of anything significant in the wild actually using that.

<xeblog-slide name="golab-wasm/slides/055"></xeblog-slide>

When doing things like calling functions or switching goroutines, the Go
compiler will tell the stack pointer to move to a new location. Each goroutine
has its own stack and in order to switch to that goroutine you need to be able
to move the location of the stack around.

<xeblog-slide name="golab-wasm/slides/056"></xeblog-slide>

So you need to have a workaround. There's many ways you could do this, but one
way to think about this is the overall flow of the stack pointer as the program
runs.

<xeblog-slide name="golab-wasm/slides/057"></xeblog-slide>

When a goroutine calls a function, it has to change the stack pointer to the new
location. The stack pointer is a CPU register pointing to some place in memory.
This pointer normally gets changed as you manipulate the stack.
  
This means that you could just pass what the stack pointer would have been as an
argument to every function, right? It would be a bit janky and could cause some
extra overhead when you try to reference things in the stack, but it would work.

<xeblog-slide name="golab-wasm/slides/058" essential></xeblog-slide>

So the WebAssembly port does this. Every Go function in WebAssembly takes in the
stack pointer and returns nothing.
  
There's some exceptions for inside the runtime when the program starts up, but
otherwise every function really does just have that one parameter: the stack
pointer.

<xeblog-slide name="golab-wasm/slides/059" essential></xeblog-slide>

Everything is returned by pushing to the stack. Arguments are read by popping
from the stack. Everything is done with type-specific offsets that the compiler
just knows from the types.

The slide shows https://github.com/Xe/olin/blob/0cf90810960ba4d7d80e20448ec08a71a3510deb/abi/wasmgo/abi.go#L242 syntax highlighted

```go
// goRuntimeNanotime implements the go runtime function runtime.nanotime. It uses
// the Go abi.
//
// This has the effective type of:
//
//     func (w *WasmGo) goRuntimeNanotime() int64
func (w *WasmGo) goRuntimeNanotime(sp int32) {
	now := time.Now().UnixNano()
	w.setInt64(sp+8, int64(now))
}
```

So if you wanted to implement one of the internal runtime functions like
runtime.nanotime you need to push your return value 8 bytes ahead of the stack
pointer. Implementing all of this by hand is a huge pain in practice and
requires deep knowledge in how the Go compiler works.
  
This code is from my early attempt at server-side WebAssembly named Olin, where
I tried to do just that: implement the Go compiler WebAssembly support for
server-side execution. I didn't get to the point where I could run arbitrary
programs, but I got very close.

<xeblog-slide name="golab-wasm/slides/061"></xeblog-slide>

This is kinda crazy, but I'm fairly sure this is the secret sauce that makes
Goroutines work in WebAssembly. Goroutine stacks are in memory like normal, and
by changing the stack pointer in function arguments, they can be swapped around.
This lets the runtime execute concurrent code in a browser even without multiple
thread support. It just can't do two unrelated calculations at once.

<xeblog-slide name="golab-wasm/slides/062"></xeblog-slide>

WebAssembly has a stack, but it's not compatible with how goroutine stacks work.
Go works around this by putting goroutine stacks in memory and passing around
the stack pointer as a hot potato.

<xeblog-slide name="golab-wasm/slides/063"></xeblog-slide>

Now that we have the ability to reference objects in the browser and the ability
to run Go functions, what comes next? How do we use this to do something useful?

<xeblog-slide name="golab-wasm/slides/064" essential></xeblog-slide>

We use the global object! In JavaScript there's a magic global object called
`globalThis`. This object will always be present in both browsers and
server-side JavaScript environments and this is where all of the global objects
like Date and WebSocket live as well as functions like fetch.
  
When a Go WebAssembly binary gets started, one of the first objects that it sets
up is a reference to this global object. This allows your Go program to access
things like HTML manipulation in a browser and the filesystem if you run it on a
server.

<xeblog-slide name="golab-wasm/slides/065"></xeblog-slide>

From here you can do whatever you want. You can make HTTP requests like normal
and they will automagically be sent to the fetch function in JavaScript. You're
free to use anything in any way you want.
  
Want to make a button that reads from some input fields and sends a HTTP request
to an API server? You can do that! Want to connect to a WebSocket server? You
can do that! It all works great!

<xeblog-slide name="golab-wasm/slides/066"></xeblog-slide>

Except you have to write a lot of the wrappers for various JavaScript types yourself.
  
Once those are done (it may take a few tries to get something that is completely
correct, sadly), you can use them as you would in JavaScript. This is where
another property of Go comes in handy to make things much more convenient:
interfaces.

<xeblog-slide name="golab-wasm/slides/067"></xeblog-slide>

One of the most unique features of Go are interface types. Interface types let
you describe the "shape" of a type in terms of what methods it exposes.

```go
type Quacker interface {
  Quack()
}
```

This means you can make a Quacker interface that has a method named Quack and
then another type Duck that implements it. Duck is a Quacker.
  
You can also make yet another type named Sheep and make a Sheep into a
Quacker...assuming you can figure out what a sheep that can quack would even
look like.

<xeblog-slide name="golab-wasm/slides/069"></xeblog-slide>

So when you make your wrapper around WebSockets, you can also make your wrapper
an io dot reader so you can read data out of it, an io dot writer so you can
write data into it, and an io dot closer so you can close the socket.
  
Then you can use your WebSocket wrapper from inside your Go code and you won't
even have to switch out most of the types. Shove your websocket into places
where you would put readers. Make it implement net dot conn calls and then also
use it like you would a socket. The world is your oyster and it is full of
pearls.

<xeblog-slide name="golab-wasm/slides/070"></xeblog-slide>

And all those rough edges for using a completely different compiler target in a
completely different environment start to fade away. Worst case you'd need to do
some build-tag specific code for the WebAssembly port (because Linux programs
aren't running in a JavaScript interpreter), but a lot of the time you'll be
fine.

<xeblog-slide name="golab-wasm/slides/071"></xeblog-slide>

Writing those wrapper types will get easier with time too. The first few will be
kind of weird, but once you hit your stride it will become a lot easier. It will
certainly teach you a *lot* about how JavaScript types work, which will make you
write better at JavaScript should you choose to. It'll work out.

<xeblog-slide name="golab-wasm/slides/072" essential></xeblog-slide>

If you want to adapt a Go program to use WebAssembly or make a new program with
WebAssembly in mind, here's some advice on how to make things work best and cope
with your life decisions.

<xeblog-slide name="golab-wasm/slides/073"></xeblog-slide>

The Hexagonal architecture technique (also known as ports and adapters) is a
common pattern that has you describe programs by the ways that they communicate
the outside world.
  
Programs poke outside through ports which have adapters on the other end. An
HTTP client would be a port, and the transport that uses the JavaScript fetch
function would be an adaptor. Imagine how that extends more generically across a
lot of the things your program does.

<xeblog-slide name="golab-wasm/slides/074"></xeblog-slide>

Go's interface system has to be tailor-made to make hexagonal architecture a
first-class citizen in the ecosystem. You use interfaces without even thinking
about it. 
  
A HTTP ResponseWriter is an interface, allowing you to handle HTTP 1.1, 2 and 3
connections with the same handler code. Writing things to files usually has you
use a writer interface as the sink. Logging usually goes to interfaces too. It's
interfaces all the way down!

<xeblog-slide name="golab-wasm/slides/075"></xeblog-slide>

Think about where the ports are in your application. What are the adaptors that
are there? How could you add in a new one? How could users of your library adapt
it to meet their needs?
  
In practice this will usually boil down to things you should already be doing,
like "let people pass in already open network connections" "let people pass in
their own preconfigured HTTP clients".
  
If you let users supply their own preconfigured adaptors, then they can use your
library in any way they want with the flexibility to meet their needs. Even if
they probably shouldn't be doing things like that in the first place. 
  
And when you need to figure out where to put ports and adaptors in your
application, interfaces aren't a bad place to start.

<xeblog-slide name="golab-wasm/slides/076"></xeblog-slide>

Debugging WebAssembly can be really annoying at times. A lot of the time when
you debug WebAssembly you're really not left with many options for actually
doing the debugging. There's a debugger in your browser's development tools, but
the developer experience still leaves a lot to be desired.

Ask me how I know. 

<xeblog-slide name="golab-wasm/slides/077"></xeblog-slide>

Actually, you know what, let's go into story time. Here is my story of one of my
first times debugging a complicated WebAssembly program without using the
debugger in the browser.

<xeblog-slide name="golab-wasm/slides/078" essential></xeblog-slide>

One of my side projects I alluded to earlier is a WebAssembly on the server
environment named Olin. I wanted to create Olin to function as the backbone of
something like AWS Lambda or Google Cloud Functions. I wanted people to be able
to upload WebAssembly modules to a control server and then trigger them to be
run *somewhere* when events happen.

<xeblog-slide name="golab-wasm/slides/079"></xeblog-slide>

So I was hacking up a storm and I managed to get something working. I had
written a program took an HTTP request from the user and spit back an HTTP
response to be send back. It was working. I was excited. Then I added the
ability to poke an outside API and then something went wrong. My WebAssembly
module panicked and I got back a nebulous error message.
  
Helpfully, I there were no debug logs. I adjusted the code to change the panic
handler to explode more loudly. No dice. I had to figure out another way to
understand what was going on.

<xeblog-slide name="golab-wasm/slides/080"></xeblog-slide>

When a WebAssembly module crashes, it can surface as the runtime environment
panicking. In my case, it panicked just after I read a bunch of data into memory
from an API. In WebAssembly, linear memory is just a contiguous series of bytes.
Something was wrong with the data that my code was processing and then I had an
idea.
  
What if I could inspect the memory?

<xeblog-slide name="golab-wasm/slides/081"></xeblog-slide>

So I took a look at the API of the WebAssembly VM I was using. I did have access
to the linear memory for that WebAssembly module as a byte slice. I don't know
what state it is in, but I know that I can get it. I also know that I can write
it to a file with the handy Writer interface.

<xeblog-slide name="golab-wasm/slides/082"></xeblog-slide>

I added a command line flag to unconditionally dump WebAssembly memory to a file
whenever the WebAssembly process finishes. I ran the program again and it
crashed again. Whew, at least it was consistent.
  
So now I have this several megabyte long binary file that I needed to
investigate. I admit that I haven't done much binary file parsing, but in the
past when I've needed to do terrible things I've reached for a tool called `xxd`.

<xeblog-slide name="golab-wasm/slides/083"></xeblog-slide>

`xxd` is a tool that lets you see the raw bytes of a binary file by showing them
both in hexadecimal form and in ascii form (if the character is printable). If
you run it on a longer file, it will run over your terminal screen space and you
will need to pipe the output to the less command to break the output into
"pages".
  
This makes it possible to understand the hexadecimal soup that will pour out of
the memory dump.

<xeblog-slide name="golab-wasm/slides/084"></xeblog-slide>

So I started to read the hex dump of the crashed WebAssembly program, and at
first it felt like I understood nothing. It was overwhelming at first as my
terminal paged through function name after function name (was that for panic
handling?) and then I quit out of that and took a moment to look at the code
again and think.

<xeblog-slide name="golab-wasm/slides/085"></xeblog-slide>

The program was just making an HTTP request, this means that there's going to be
an HTTP response somewhere. The HTTP response had JSON in it. JSON has lots of
curly braces. If I want to find out what was going on, I need to look for curly
braces.

<xeblog-slide name="golab-wasm/slides/086"></xeblog-slide>

And like that, I was in, I found the JSON in memory and then I took a look at
the JSON compared to my code. After a couple double takes I felt flabbergasted.
I had the wrong type for a variable in a structure, and I was using the `unwrap`
function in Rust to parse it. The type was wrong, so it tried to log an error
message, panicked, and crashed. Beyond that my panic handler didn't work, so I
had to fix that too.
  
But, after I fixed everything it worked. And that felt so, so good.

<xeblog-slide name="golab-wasm/slides/087"></xeblog-slide>

I'm pretty sure that this is a lot easier in browsers now. I'm not sure if Go
supports source maps though, those let you see the source code of the programs
you're debugging in the browser inspector. This is most commonly used when
you're trying to debug minified JavaScript code. It would be really convenient
if that support was added in the future.

<xeblog-conv name="Cadey" mood="coffee">Author's note: turns out the WebAssembly
debugger in the browser didn't exist when I did the hacking I alluded to in this
part of the talk. I couldn't have used it if I wanted to.</xeblog-conv>

<xeblog-slide name="golab-wasm/slides/088"></xeblog-slide>

Something else to keep in mind is that you need to keep things cognitively
simple. Don't overcomplicate things. Things are going to be complicated enough
because of the weirdness involved in writing your Go code to target a new
platform. Don't be clever.

This is going to spend a lot of "innovation points", so be sure to make things
as easy to understand as much as you can. It makes debugging crashes so much
easier.

<xeblog-slide name="golab-wasm/slides/089"></xeblog-slide>

I've spent a lot of time on the state of the world as it is and the hacks we
needed to get there, but I'm a lot more excited about what the future could
bring. Here's some things that I can't wait for.

<xeblog-slide name="golab-wasm/slides/090"></xeblog-slide>

WebAssembly is still under active development to improve the state of the world.
One of the proposals I've kept my eye on is the component model proposal. It
introduces the concept of "component model types". They are similar to Go
interfaces, but they go a step farther. They define complicated objects and
fields on them, not just methods.

You will be able to code generate native bindings for these interface types. You
will be able to do DOM manipulation in Go with the same calls as you will in
Rust or Zig. You will be able to import things like WebRTC into your Go programs
and operate on it like it was written in Go in the first place.

Everything would be taken care of for you. There would be no more drastic
wrapper types all over the place. There would be no more NaN-boxed object IDs.
You'd just write Go and it'd just work. You would also no longer need to do so
much feature detection compared to what you need to do currently.

The worst part about this is that browsers don't support this yet. There is one
server-side runtime that has experimental support for them, but nothing else
really does so it's not overly usable yet.

God I want these though, sooner is better.

<xeblog-slide name="golab-wasm/slides/091"></xeblog-slide>

One of the weaknesses of WebAssembly is that it's a single-threaded environment.
A WebAssembly program can only really do one calculation at once. There's a
threading extension that will change that. It allows for WebAssembly programs to
have multiple threads that execute in parallel.
  
This will allow your Go programs in browsers to do multiple calculations at
once, just like your Go programs on servers.
  
Chrome and Firefox have experimental support for threads, but Safari (and by
extension every iOS device on the planet) does not. As such, the Go WebAssembly
port doesn't take advantage of these yet. It will be really great when we can
though!

<xeblog-slide name="golab-wasm/slides/092"></xeblog-slide>

But, we can't do that yet. It'll be better soon. I have faith in the system. It
looks slow now, but that's because people are trying to make sure that they
don't mess it up. That hesitance is surely going to end up being a benefit.

<xeblog-slide name="golab-wasm/slides/093"></xeblog-slide>

Well, we covered a lot today. We learned a bunch of things:

<xeblog-slide name="golab-wasm/slides/094"></xeblog-slide>

The Go compiler has a WebAssembly port. You can use it to run your Go code in a
browser.

<xeblog-slide name="golab-wasm/slides/095"></xeblog-slide>

The WebAssembly port can be used to poke the browser and manipulate webpages
with the same calls that you use in JavaScript.

<xeblog-slide name="golab-wasm/slides/096"></xeblog-slide>

We learned the terrible secret of NaN-boxing and why someone would envision, let
alone implement, such an accurse-ed thing.

<xeblog-slide name="golab-wasm/slides/097"></xeblog-slide>

We learned about Go's calling convention, stack manipulation, and how Go works
around platform limitations in WebAssembly.

<xeblog-slide name="golab-wasm/slides/098"></xeblog-slide>

We learned about Go interfaces and hexagonal architecture to help you fit square
pegs into round holes.

<xeblog-slide name="golab-wasm/slides/099"></xeblog-slide>

And finally I got your hopes up for the future before smashing them down back to
Earth by saying that we can't have those nice things yet.

<xeblog-slide name="golab-wasm/slides/100"></xeblog-slide>

If you haven't played with the WebAssembly port yet, I'd suggest trying it out.
It's a totally different way of writing Go than what you do on a regular basis.
Web apps are also very easy to share with your friends slash group chat because
everyone has a web browser installed. You may just be able to make something
useful that people come back to. Try it and see!

<xeblog-slide name="golab-wasm/slides/101" essential></xeblog-slide>

These talks take a lot of effort, time and research to turn into the reality
you've seen here today. I want to especially shout out everyone on this list for
helping make this talk shine in some way.

Thanks! You all really help more than you can imagine.

<xeblog-slide name="golab-wasm/slides/102" essential></xeblog-slide>

And thank you for watching! I'm going to be available to answer any questions I
haven't answered already. If I miss your question somehow or you really want an
answer privately, please email it to
[webassemblyabi@xeserv.us](mailto:webassemblyabi@xeserv.us).

I'll have a written version of this talk including my slides, a recording of the
talk and everything I've said today on my blog pretty soon.

If you have questions, please speak up. I love answering them and I am more than
happy to take the time to give a detailed answer.

Be well, all.
