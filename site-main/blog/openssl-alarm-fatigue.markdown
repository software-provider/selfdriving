---
title: "OpenSSL gave everyone alarm fatigue"
date: 2022-11-01
tags:
 - openssl
 - rant
 - security
 - noxp
---

<xeblog-hero ai="Waifu Diffusion v1.3 (float16)" file="angy-foxgirl-disapproves" prompt="1girl, kimono, animal crossing, klaxon, loud noises, overwhelming, red sky, clouds, storms, long hair, purple hair, yellow eyes, fox ears, thick outlines, ink outlines, black outlines"></xeblog-hero>

So, the OpenSSL security issue embargo ended today and the patches dropped.
Based on the contents of the security issue, the difficulty of exploiting it in
practice, and the fact that most Linux distributions take basic precautions to
prevent it from being a viable attack vector: this issue doesn't affect nearly
any users of OpenSSL in the real world.

For more details on CVE-2022-3786 and CVE-2022-3602, see the OpenSSL team's post
titled [CVE-2022-3786 and CVE-2022-3602: X.509 Email Address Buffer
Overflows](https://www.openssl.org/blog/blog/2022/11/01/email-address-overflows/).

<xeblog-conv name="Mara" mood="hacker">tl;dr the vulnerability is a combination
of two forces of complexity converging in a way that nobody expected:
[X.509](https://en.wikipedia.org/wiki/X.509) formats for digital certificates
and [Punycode](https://en.wikipedia.org/wiki/Punycode), an encoding standard for
putting characters that are unrepresentable in DNS into DNS (such as `我很伤心.example`
being encoded into `xn--hpq721bdf90g.example`). This works in email
addresses too. The issue was that if you gave a malformed email address in a TLS
certificate, then that could percolate up into remote code execution through a
buffer overflow on some platforms. Thankfully, that definition of "some
platforms" does not include Ubuntu on amd64 CPUs.</xeblog-conv>

The main reason why this isn't a big deal for most people is the fact that most
modern compilers enable [stack overflow
protection](https://security.stackexchange.com/questions/158609/how-is-the-stack-protection-enforced-in-a-binary/158616#158616),
which will stop this buffer overflow from causing arbitrary code to be executed.
In practice, the stack protector will realize that something is wrong and make
the program explode. This reduces the attack from an arbitrary code execution
attack to a simple denial of service attack, which is a lot more boring in
practice.

For more details on this, see [this
writeup](https://github.com/colmmacc/CVE-2022-3602) that explains how this
attack works, how you can replicate it at home with a vulnerable version of
OpenSSL, and ultimately why it's not a practical problem. If any organization
wanted to use this, it would almost certainly result in the certificate being
noticed on certificate transparency logs and then potentially the entire
certificate authority's ability to issue certificates would be brought into
question. This would likely end up with the certificate authority not being
allowed to issue certificates until the situations that allowed that authority
to issue certificates were fixed and independently evaluated to be fixed.

I guess it is most viable in environments with custom TLS root certificates, but
those are rare enough in practice to not really make this a useful attack
mechanism. There's much lower-hanging fruit in the equation. Maybe people really
do distrust certificate authorities that much, though.

<xeblog-conv name="Numa" mood="delet">Yeah, it's not like
[mTLS](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/)
exists, requires you to use custom certificate authorities and _absolutely_
would be a great way to use this vuln to get up to no good. We don't have
anything like that at all.</xeblog-conv>

<xeblog-conv name="Mara" mood="hmm">Oh, that could be _very bad_. Oh god that
could be so bad.</xeblog-conv>

I feel somewhat compelled to apologize for how I've been treating the OpenSSL
issue on my blog. I took the OpenSSL team at their word, and I feel that I have
done the right thing, even if it turned out to be a giant nothingburger in
practice. If my coverage of the issues harmed you or made you worry beyond
normal, I'm sorry. I was going off of the information I was given and I had
heard from several sources I trust that it was something to really care about. I
will use this experience to help shape how I report on these things in the
future.

<xeblog-conv name="Cadey" mood="angy">This is _technically_ meeting the
definition of "CRITICAL" in [the OpenSSL
definitions](https://www.openssl.org/policies/general/security-policy.html#critical).
This _can_ lead to arbitrary attacker-controlled code execution in theory, but
in practice it won't for the majority of users.</xeblog-conv>

## Why I'm frustrated

There are two big things that frustrate me about this pair of vulnerabilities.

The first is that apparently the OpenSSL team _does not know what compiler flags
are used to compile OpenSSL on popular targets_. Like, I get it, at some level
this makes sense. OpenSSL is used in a billionty different environments and all
of them are unique snowflakes.

However, at some level this feels a bit inexcusable. Ubuntu and CentOS are some
of the biggest users of OpenSSL. People that work for Canonical and Red Hat are
contributors to OpenSSL. I feel that they should have _some_ sense of what the
most common environments do. Sure they are not going to be able to stop some
embedded device in Palau from ACE-ing a pointer from an email address in a
certificate, but I'd certainly hope that Ubuntu and CentOS are the targets that
they _really_ care about and validate against.

The second thing that bothers me is that it seems that it was downgraded from a
"CRITICAL" bug to a
["HIGH"](https://www.openssl.org/policies/general/security-policy.html#high) bug
earlier on today, November 1st, 2022. To give the OpenSSL team the *widest
possible* benefit of the doubt, this is somewhat reasonable. As they continued
to poke at things and research how vulnerable people were, they realized that
their initial assessment of the issue was wrong. This means that people that
were saying it was very bad were accurately reporting on what they understood at
the time. At some level this is how the process of science works: you have a
hypothesis, you test it, you report the results. It is just frustrating that it
took so long to downgrade it. I was prepared for a critical vulnerability and
I'm sure other people were too.

I'm worried that this is going to be seen as a reason to not take "CRITICAL"
disclosures seriously at first glance like we _should_. A "CRITICAL" bug
**MUST** be treated as if it was critically bad. From a community health
perspective, people have been told that something really bad is about to come
out for _a week_ and then had the rug pulled out from under them and now it's
"nah we were wrong you're probably fine".

Again, this happens, and it's perfectly rational for them to change their minds.
I just worry that this level of alarm fatigue has just trained people to not
take things seriously because this time it wasn't so bad.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Turns out the OpenSSL advisory was more disruptive than the vulnerability.</p>&mdash; dkp (@tweetdkp) <a href="https://twitter.com/tweetdkp/status/1587537224618369032?ref_src=twsrc%5Etfw">November 1, 2022</a></blockquote>

NixOS has stack protection enabled, so it was never vulnerable to this issue in
the first place. My [article](https://xeiaso.net/blog/nixos-nginx-openssl-1.x)
telling people to recompile nginx with OpenSSL 1.x wasn't needed. However, it
was a cool way to show off `overrides` in nixpkgs, so I'm going to consider that
a win regardless.

## Hot Take Zone™️

Maybe we need to stop writing new security-critical code in C. Here is the
_entire contents_ of one of the patches that fixed
[CVE-2022-3602](https://github.com/openssl/openssl/commit/fe3b639dc19b325846f4f6801f2f4604f56e3de3):

```diff
--- a/crypto/punycode.c
+++ b/crypto/punycode.c
@@ -181,7 +181,7 @@ int ossl_punycode_decode(const char *pEncoded, const size_t enc_len,
         n = n + i / (written_out + 1);
         i %= (written_out + 1);
 
-        if (written_out > max_out)
+        if (written_out >= max_out)
             return 0;
 
         memmove(pDecoded + i + 1, pDecoded + i,
```

For those of you that don't know C that well, here's my attempt to explain
what's going on. This is in a function that decodes
[punycode](https://en.wikipedia.org/wiki/Punycode) text into its corresponding 
Unicode text. One of the things it does is copy data from the input buffer,
decode it, and then put the results in an output buffer. This if statement is
here to prevent the buffer from running over and into whatever comes next. If
that "whatever comes next" is in the stack, then the value can potentially
overflow and then that value becomes the return address for when that function
is called.

The root cause of this is an off-by-one out-of-bounds write to the output
buffer. This is made possible because C does not have ways for you to statically
prove that you cannot overwrite buffers like this such that the compiler will
refuse to compile the code.

Maybe as an industry we need to start taking formal verification, memory safety,
and related things a bit more seriously. Maybe adding Rust to the Linux kernel
is a good thing. [Rust was also recently added to the Windows
kernel](https://twitter.com/dwizzzleMSFT/status/1578532292662005760) without
anyone really noticing its presence.

<xeblog-conv name="Cadey" mood="coffee">Or at least we can work towards
lessening our dependencies on C for security-critical infrastructure like TLS
session management with things like
[rustls](https://docs.rs/rustls/latest/rustls/) or using Go's
[crypto/tls](https://pkg.go.dev/crypto/tls).</xeblog-conv>

<xeblog-conv name="Numa" mood="delet">But if we write things in Rust then it
won't work on my Alpha VAX cluster full of OpenVMS machines that runs the logic
for a secure cluster of Gopher servers!</xeblog-conv>

<xeblog-sticker name="Cadey" mood="coffee"></xeblog-sticker>

<xeblog-toot url="https://pony.social/@cadey/109270702249158842"></xeblog-toot>
