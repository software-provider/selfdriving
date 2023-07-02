---
title: Time is not a synchronization primitive
date: 2023-06-25
---

Programming is so complicated. I know this is an example of the
nostalgia paradox in action, but it easily feels like everything has
gotten so much more complicated over the course of my career. One of
the biggest things that is really complicated is the fact that working
with other people is always super complicated.

One of the axioms you end up working with is "assume best intent".
This has sometimes been used as a dog-whistle to defend pathological
behavior; but really there is a good idea at the core of this:
everyone is really trying to do the best that they can given their
limited time and energy and it's usually better to start from the
position of "the system that allowed this failure to happen is the
thing that must be fixed".

However, we work with other people and this can result in things that
can troll you on accident. One of the biggest sources of friction is
when people end up creating tests that can fail for no reason. To make
this even more fun, this will end up breaking people's trust in CI
systems. This lack of trust trains people that it's okay for CI to
fail because sometimes it's not your fault. This leads to hacks like
the flaky attribute on python where it will ignore test failures. Or
even worse, it trains people to merge broken code to main because
they're trained that sometimes CI just fails but everything is okay.

Today I want to talk about one of the most common ways that I see
things fall apart. This has caused tests, production-load-bearing bash
scripts, and normal application code to be unresponsive at best and
randomly break at worst. It's when people use time as a
synchronization mechanism.

## Time as an effect

<xeblog-conv name="Aoi" mood="wut">What do you mean by that? That
sounds mathy as all heck.</xeblog-conv>

I think that the best way to explain this is to start with a flaky
test that I wrote years ago and break it down to explain why things
are flaky and what I mean by a "synchronization mechanism". Consider
this Go test:

```go
func TestListener(t *testing.T) {
  ctx, cancel := context.WithCancel(context.Background())
  defer cancel()

  go func() {
    lis, err := net.Listen("tcp", ":1337")
    if err != nil {
      t.Error(err)
      return
    }
    defer lis.Close()
    for {
      select {
      case <- ctx.Done():
        return
      default:
      }
      conn, err := lis.Accept()
      if err != nil {
        t.Error(err)
        return
      }

      // do something with conn
  }()

  time.Sleep(150*time.Millisecond)
  conn, err := net.Dial("tcp", "127.0.0.1:1337")
  if err != nil {
    t.Error(err)
    return
  }
  // do something with conn
}
```

This code starts a new goroutine that opens a network listener on port
1337 and then waits for it to be active before connecting to it. Most
of the time, this will work out okay. However there's a huge problem
lurking at the core of this: This test will take a minimum of 150
milliseconds to run no matter what. If the logic of starting a test
server is lifted into a helper function then every time you create a
test server from any downstream test function, you spend that
additional 150 milliseconds.

Additionally, the TCP listener is probably ready near instantly, but
also if you run multiple tests in parallel then they'll all fight for
that one port and then everything will fail randomly.

This is what I mean by "synchronization primitive". The idea here is
that by having the main test goroutine wait for the other one to be
ready, we are using the effect of time passing (and the Go runtime
scheduling/executing that other goroutine) as a way to make sure that
the server is ready for the client to connect. When you are
synchronizing the state of two goroutines (the client being ready to
connect and the server being ready for connections), you generally
want to use something that synchronizes that state, such as a channel
or even by eliminating the need to synchronize things at all.

Consider this version of that test:

```go
func TestListener(t *testing.T) {
  ctx, cancel := context.WithCancel(context.Background())
  defer cancel()

  lis, err := net.Listen("tcp", ":0")
  if err != nil {
    t.Error(err)
    return
  }

  go func() {
    defer lis.Close()
    for {
      select {
      case <- ctx.Done():
        return
      default:
      }
      conn, err := lis.Accept()
      if err != nil {
        t.Error(err)
        return
      }

      // do something with conn
  }()

  conn, err := net.Dial(lis.Addr().Network(), lis.Addr().String())
  if err != nil {
    t.Error(err)
    return
  }
  // do something with conn
}
```

Not only have we gotten rid of that time.Sleep call, we also made it
support having multiple instances of the server in parallel! This code
is ultimately much more robust than the old test ever was and will
easily scale for your needs. If your tests took a total of 600 ms to
run each, cutting out that one 150 ms sleep removes 25% of the wait!

<xeblog-conv name="Aoi" mood="wut">I see, I see. I'm still not sure
what you're getting at though. If time isn't a reliable way to
synchronize things, why do people use it?</xeblog-conv>
<xeblog-conv name="Cadey" mood="coffee">Well, the basic idea is that
time is an _effect_, not a cause. When you are trying to make the state of multiple
concurrent/parallel tasks synchronized to the same state, you can
imagine time as incidental to the actions, not an inherent cause of
them. Consider what happens when you leave wet bread out in the open
for a while: it gets moldy. Time passing didn't cause the mold to
appear on the bread, the bread being in the open and wet caused it to
be a decent substrate for mold to grow on top of. Time is incidental
to the mold developing, not causal. It is the same way with computer
programs. Waiting one second does not make the service ready. The
service being ready makes it ready. The worst part is that waiting a
second or two will usually work well enough that you don't have to
care.</xeblog-conv>
<xeblog-conv name="Aoi" mood="grin">I see, thanks!</xeblog-conv>

## Putting it into practice

So let's put this into practice and make this kind of behavior more
difficult to cause. Let's add a roadblock for trying to use
`time.Sleep` in tests by using the `nosleep` linter. `nosleep` is a Go
linter that checks for the presence of `time.Sleep` in your test code
and fails your code if it finds it. That's it. That's the whole tool.

You can run it against your Go code by installing it with `go install`:

```
go install within.website/x/linters/cmd/nosleep@latest
```

And then you can run it with the `nosleep` command:

```
nosleep ./...
```

I do recognize that sometimes you actually do need to use time as a
synchronization method because god is dead and you have no other
option. If this does genuinely happen, you can use the magic command
`//nosleep:bypass here's a very good reason`. If you don't put a
reason there, the magic comment won't work.

Let me know how it works for you! Add it to your CI config if you dare.
