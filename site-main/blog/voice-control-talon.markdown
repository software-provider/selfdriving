---
title: Talon is amazing
date: 2023-02-28
tags:
 - dictated
 - voiceControl
 - talon
---

<xeblog-conv name="Cadey" mood="coffee">When I was writing this post,
I was in a unique level of flow state working with Talon. It's
probably gonna come across a little bit weird. Sorry!</xeblog-conv>

In my last article I discussed the voice control challenge, it is a
very simple thing: put your hands at your sides and use only your
voice to do your daily tasks. In essence, doing this makes you get
exposed to what it is like for people who have to rely in these
technologies on a daily basis.

For many of you reading this post, this is just a thought experiment.
This is something that you try once or twice and then go back to
normal. It can be frustrating, none of your applications will work
correctly. Selecting some UI elements may be difficult or even
impossible.

One way to look at a challenge like this is to help you train empathy
for those who have to rely on this technology.  it can make you feel
powerless. This is a feeling that we rarely get to have as
technologists.

The thing that I really feel exemplifies the difficulty of the voice
control challenge is controlling a text editor. Text editors require
arbitrary keyboard inputs. They require pressing weird key binds.
Doing this with voice control on a mac or iPad can be difficult to
impossible.

However, dictation like this is really great for writing large blocks
of text. Human speech can peak it around two hundred words per
minute, and that's where most dictation engines can really shine.

But, we're technologists. For many people being able to write basic
emails and messages is more than enough. Statistically, if you're
reading this blog, you probably need to do more. A lot more. We
regularly have to speak in the arcane sigils of foreign languages in
order to command automatons into doing our will. When you think about
it, programming is actually a lot like witchcraft, except we don't
call ourselves witches for some reason.

<xeblog-hero ai="Counterfeit" file="utahime" prompt="1girl, microphone, hoodie, skirt, singing, pink hair, long hair, holding microphone, stockings, stage, nightclub, dubstep"></xeblog-hero>

During my adventure through assistive technology, I found a program
named Talon. [Talon](https://talonvoice.com/) is a software that
allows you to control a computer with your voice, but you can use it
for programming and it is programmable. The somewhat fiddly process of
making your own commands in voice control on a Mac or an iPhone
vanishes. You can write commands in something that looks vaguely like
Python, or in Python itself:
 
```
evil normal:
    key(esc)
```

This creates a new command that presses the escape key when you say
"evil normal". You can replace this command with anything you want.
You aren't limited to just pressing individual keys, you can do just
about anything you can imagine. You can insert arbitrary strings of
text, run python functions, bully around google chrome, or just about
anything that you can do with programming.

<xeblog-conv name="Cadey" mood="enby">This entire post was written
with Talon. Even the outline. Even this HTML tag. People on Discord
can confirm this.</xeblog-conv>

Through sheer luck, I actually managed to talk with the person behind
Talon recently when I was on a work trip. After using the software for
about a week, I am convinced that this person is literally making a
positive impact on this planet. This program is an empowerment tool.
It was originally written so this person could continue to code
without incurring massive hand pain, but it is also useful for people
that can't use their hands at all.

This program is a labor of love for this person, and it really shows.

## Talon slaps

After I figured out how to set things up and use it, I can
confidently say that Talon is revolutionary. I can leave my hands in
my lap, and continue to code. I can look at the screen or look off
into the distance and think about what's going on, and I can just let
the words flow out. There's a unique feeling that you end up getting
when you get really good at using vim and other text editors, all the
barriers start to drift away and you really just start to think about
the substance of what you are doing rather than the exact hand
motions. I get this kind of feeling with Talon. All the details float
away and I'm really able to just focus on my mental model of what's
going on, where I am, and where I want to get to. Once you learn the
commands in order to interact with things, it just becomes effortless.

One of the most interesting parts about Talon is how it acts kind of
like vim. There are several different modes that Talon can be in:

- sleep mode - in this mode, Talon is not listening for commands
- command mode - in this mode, Talon listens for commands and executes
  them in the order they are spoken
- dictation mode - in this mode, Talon listens for large blocks of
  text and types them out as you say them
  
This is a lot similar to vim's normal and append modes. A lot of the
commands function similar to vim motions too. A lot of them are simple
instructions that can take arguments much like vim motions. Combine
that with optional repetitions and other suffixes, and it really
becomes a ballet of language and commands together. For example, here
is a little bit of go code and the equivalent Talon motions:

```go
func add(x int, y int) int {
    return x + y
}
```

This code will become the following Talon motions:

```
word funk delete cap space word add args plex sit near trap comma
space yank space sit near trap go right space sit near trap space
brack slap
space fourth word return space plex plus yank slap
r brack
```

This may look incomprehensible, but let me split it out into
individual commands and explain what they do:

| command | meaning |
|---------|---------|
| word funk | type the word "funk" |
| delete | press the backspace key |
| cap | type the letter "c" |
| space | press the spacebar |
| args | type both parentheses and press the left arrow key to go inside them |
| plex | type the letter "x" |
| sit | type the letter "i" |
| near | type the letter "n" |
| trap | type the letter "t" |
| yank | type the letter "y" |
| go right | press the right arrow key |
| brack | type the left curly brace "{" |
| slap | press the enter key |
| space fourth | press the four times |
| plus | type the plus sign "+" |
| r brack | type the right curly brace "}" |

Overall these are all the commands you do by hand with a keyboard. It
just looks a bit more awkward, but this is what happens when you try
to force sigils out of your voice.

This has been bending my brain backwards into a pretzel and then
setting that pretzel on fire. All of the Talon motions are weirdly
useful. They feel like they were created intentionally and with
purpose to help you do your job. Some of them are legitimately faster
than typing things by hand, such as the text formatters:

- all cap - THIS IS IN ALL CAPITAL LETTERS
- camel - thisIsInCamelCase
- dotted - these.are.separated.by.periods
- dub string - "this is a quoted string"
- hammer - ThisIsAnExportedGoName
- kebab - some-kebab-would-be-tasty-for-dinner
- packed - working::with::rust::crates::has::never::gotten::easier
- slasher - /directory/paths/are/easy/now
- smash - sometimesyouneedtosmashwordstogether
- snake - something_about_anacondas_i_guess
- title - This is a Fancy Way to Type

This is really cool. Something interesting about using voice for this
means that you are not limited by the number of keys on a keyboard,
or the programmability of the keyboard in general. The human voice is
a much more flexible tool than we give it credit. This allows us to
create commands that can do just about anything, with just about any
sound to trigger it.

Isn't that metal as all hell?

As I've been going down this rabbit hole, another thing I did not
expect to  have is an intuitive knowledge of how many letters each
word is. Just looking at words I can figure out how many characters
away different things are from other characters. This is very
important when you have to specify the number of times you need to
move an arrow key.

I've always had a sort of intuitive understanding of vim motions.
I've never quite had the need to learn how to say them, or what they
are all doing. Saying them out loud has forced me to understand what
I am saying with vim motions, and as a result I am able to think in
them a lot more clearly. A lot of things make a lot more sense when
you have to say them out loud repeatedly. Changing modes becomes even
more intuitive. Jumping around with the different movement motions
becomes even more easy. And the delete motion just starts making a
lot more sense overall.

I won't lie, this is causing me to bend my brain backwards. A lot of
the programming did I do with Talon ends up sounding like I'm having
a stroke or something though. In vim the main command to save a file
is `:w<enter>`. This means you end up saying "colon whale slap" a lot.
People don't know how to react to this.

<xeblog-conv name="Numa" mood="delet">I guess this is why we call them
key<i>stroke</i>s :3</xeblog-conv>

<xeblog-conv name="Cadey" mood="enby">Someone on Discord asked me
why I was slapping whales so much when I was trying to write some code
with Talon on a voice call. I was saving the file. The person wasn't
watching my stream and had no context. I can't blame them for being
confused.</xeblog-conv>

Overall though, I have infinite respect for anyone who has to use this
type of software on a daily basis. this is very useful software, but
it can be deeply frustrating at first. Struggling through using
applications like Discord or Slack with this technology should be made
mandatory for everyone designing the UI in them. I feel that doing that
would make people a lot more humble and empathetic to users' needs.

This truly is technology where everyone can benefit from it getting
better. Voice recognition software getting better means that automatic
captions for things like YouTube get better. People who are paraplegic
or even quadriplegic can start to live normal lives in spite of their
disability.

The kind of people who have to rely on this technology on a daily
basis have a hard enough life already. As someone who is in a field
that is deeply unethical, this is one of the greatest things we can
due to help truly ensure equitable outcomes.

If you write applications, regularly test them with voice control
software. Especially if it's in a browser. Few companies do
this, so there is a clear market advantage in making things more
accessible.

My hands have been getting better, but I can easily see myself
continuing to use talon in the future even after they fully recover.
It is one of the few dictation engines I have used that is able to
keep up with my rate of speech. I have been using it for most of my
chat messages for the last week or so, and I can easily see myself
using this for chat messages in the future. It makes me feel like I've
wasted all that time learning how to type quickly when I could just
use dictation.

If you have some extra money, please donate to the creators of Talon
on [Patreon](https://www.patreon.com/join/lunixbochs). They deserve it
for making software that it powers people like this.
