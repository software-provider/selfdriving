---
title: "We're never getting rid of ChatGPT"
date: "2023-03-04"
tags:
 - ChatGPT
 - rant
 - emacs
 - philosophy
---

This week, OpenAI announced that they have created a public facing API for ChatGPT. At this point, I think it's over. We are going to have to learn to live with large language models and all of the other types of models that people will inevitably cook up.

<xeblog-hero ai="Counterfeit v2.5" file="miku-impact" prompt="masterpiece, best quality, 1girl, green hair, green eyes, sweater, hoodie, looking at viewer, adorable, cute, outdoors, watercolor, anime, afternoon, mountains, breath of the wild, black hoodie, ahoge, long hair, happy, genshin impact, hatsune miku, twintails, pigtails"></xeblog-hero>

It is probably going to end up being better to learn how to use these tools to our advantage rather than trying to fight them at this point. I suspect that integration with tools like ChatGPT is going to become like a language server is now. You can code without a language server, but do you really want to? A language server allows you to do so many things automatically, such as formatting, code completion, refactoring, and so much more. You can query documentation from inside your source code, and overall it ends up being an asset to the editing experience instead of an impediment.

I think that these large language models are going to do the same thing for a lot of other people that language servers have done for programmers. This can be a good thing, this can be a bad thing, and I don't know how it's going to turn out.

In an effort to try to get expertise in these tools, I thought it would be interesting to create some bindings for the ChatGPT API for my emacs config. So I [did that](https://tulpa.dev/cadey/nixos-configs/src/branch/main/common/home-manager/emacs/packages/xe-chatgpt.el). This allows me to ask the language model questions, and have it explain bits of code that I've selected. I tried to make the user experience as minimal as possible.

In order to ask the AI a question, I press `M-x xe/ask-chatgpt`. I will be prompted for a question to ask, and once I hit enter it will be sent to the API. The overall user experience looks something like this:

<xeblog-conv name="Mara" mood="hacker" standalone>`M-x` means "alt-x" in Emacs land.</xeblog-conv>

![](https://cdn.xeiaso.net/file/christine-static/blog/M-x-ask-chatgpt.gif)

<details>
<summary>The contents of that buffer</summary>

How do I update packages in CentOS?

To update packages in CentOS, follow these steps:

* Open a terminal window.
* Type the following command and press Enter:
  ```
  sudo yum update
  ```

This command will fetch the latest updates for all installed
packages on your system.
  
* If prompted, enter your sudo password and press Enter.
* The system will now check for updates, and if any updates are
  available, it will ask you to confirm installation. Type "y" and
  press Enter to proceed.
* Wait for the updates to get installed. Depending on the number of
  updates available, it may take a while.
* Once the updates are installed, restart your system to apply the
  changes.

That's it! Your CentOS system is now up-to-date with the latest
package updates.

</details>

You type in your question, it opens a buffer, and then you wait for your answer. This is a prototype of this, writing this was kind of difficult and exposed me to a lot of sides of emacs lisp that I don't totally understand. In the future I'd like to make this an interactive mode similar to ielm or an IRC client.  I think it would be cool for you to type in your questions and get responses from the AI one after the other; much like the ChatGPT webpage, but in your editor.

I've used this for a couple days, and it's been helpful for when I've edited things at work. Not to mention, the API is so cheap, that it's literally free for my uses. While I've been testing this and writing it, I have made a number of queries to the API. I haven't spent a penny yet. It's almost worrying.

I'm going to attempt to refine my emacs mode a little bit more, and when I feel that it's a lot less hacky and a lot more close to something reliable I'm going to release it on [MELPA](https://melpa.org). For all that I don't like about it, it is decent at overly verbosely explaining bits of code. I hop between languages a lot, and having something that just goes through the entire syntax tree can be helpful.

Here is an example of that:

<details>
<summary>An overly verbose explanation of some Rust code</summary>

Explain this code. User is in rust mode.

```rust
fn main() {
    let target = env::args().skip(1).next().unwrap();
    let srv = Server::new()
        .hostname("tsnet-rs-echoclient")
        .ephemeral()
        .authkey(env::var("TS_AUTHKEY").unwrap())
        .build()
        .unwrap();

    let mut conn = srv.dial("tcp", &target).unwrap();
    write!(conn, "This is a test of the Tailscale connection service.\n").unwrap();
}
```

This is a Rust program that connects to a Tailscale network endpoint
(`target`) and sends a test message to it. Here is what the code does
step by step:

1. `env::args()` returns an iterator over the program's command-line arguments. `skip(1)` skips the first argument (the name of the program itself), and `next()` takes the next argument, which is assumed to be the target endpoint. `unwrap()` returns the value inside the `Option` that `next()` returns. If there is no such argument, `unwrap()` will panic.
2. `Server::new()` creates a Tailscale connection `Server` object with default options.
3. `hostname("tsnet-rs-echoclient")` sets the hostname of the server to "tsnet-rs-echoclient". This is a specific hostname that the Tailscale administrator may have configured to allow access to certain resources on the network.
4. `ephemeral()` sets the server to be ephemeral, meaning that it will automatically shut down when the program exits.
5. `authkey(env::var("TS_AUTHKEY").unwrap())` sets the Tailscale authentication key to the value of the `TS_AUTHKEY` environment variable, which is assumed to be set. `env::var()` returns a `Result`, which `unwrap()` will panic on if `TS_AUTHKEY` is not set.
6. `build()` builds the `Server` object with the specified options. If the options are invalid, `build()` will return an error.
7. `srv.dial("tcp", &target)` connects to the target endpoint using the TCP protocol. `&target` is a reference to the target IP address or hostname that was specified on the command line. If the connection fails, `unwrap()` will panic.
8. `write!(conn, "This is a test of the Tailscale connection service.\n").unwrap()` writes the test message to the connection. If the write fails, `unwrap()` will panic.
9. The program exits.

</details>

I do like how this explains a lot of the unspoken assumptions about Rust code. I have checked over this response before putting it here, and I am sure that it is accurate. When using these tools, you need to double check the output. You need to treat the output like you read it on 4chan. Output from large language models should not be implicitly trusted, and you should take well more than due diligence in order to ensure that things are accurate before you accept the output effect.

Overall, I'm really mixed about this technology. If done well, this could allow people to have a lot easier access to a wide bank of knowledge. With a lot of guidance, care, and thought this could be used to create custom search tools for internal knowledge bases that would rival everything else on the planet. I can see why Google is afraid of this type of technology. It is an absolute game changer.

Yet, at the same time, it is trained off of basically the entire internet without paying anyone involved. I would be willing to bet that my articles are rumbling through the corpus somewhere. I probably won't see a penny of the money that OpenAI is going to make with this model. I am really happy that my day job is working for a tech company instead of producing what I do on my own. I can only imagine that such arrangements are going to made more common in the future.

I don't know how to feel about all this. This is fantastic technology that can enable so many things to be so much easier, and yet I can see it being used for such evil at the same time. I just keep looking at this, and I wonder what happens when someone tries to use it to radicalize people.

I suspect that Pandora's box is open, and now we must live with the diseases that were trapped inside it. I also suspect, that if we're going to get sick, we might as well learn about the sicknesses before they strike.

In the exact case of asking questions in my text editor, I am fairly sure that it is innocuous to have an AI model explain code bits like this. It is very easy to check if the AI model is accurate. As things get more complicated and the AI delves into topics like sociology, I fear that things aren't going to be as easy. For every innocuous use of this, there are a thousand evil uses waiting to happen. Holding this technology back doesn't help, releasing it to the public doesn't help, it's a proper quandary.

Ain't this some shit?