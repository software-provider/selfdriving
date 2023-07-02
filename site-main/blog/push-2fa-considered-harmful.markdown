---
title: Push notification two-factor auth considered harmful
date: 2022-09-17
tags:
 - security
 - infosec
 - webauthn
 - web3
 - collab
---

<xeblog-hero ai="Waifu Diffusion v1.2" file="evil-hacker-lain" prompt="an evil hacker at a laptop hacking into the pentagon, anime style, hacker den, monitors everywhere, serial experiments lain, evangelion"></xeblog-hero>

Sooooo [Uber got
popped](https://www.nytimes.com/2022/09/15/technology/uber-hacking-breach.html).
The hacker-internal rumor mill is churning out tons of scuttlebutt, but
something came up that I've seen before and has made me adopt a radical new
stance. I hope to explain it in this post for you all to learn from. Come along
and join [Cendyne](https://cendyne.dev/) and I on this magical journey!

<details>
<summary>tl;dr</summary>
<xeblog-conv name="Numa" mood="delet">tl;dr: push notification two-factor
authentication is a great way to produce alarm fatigue, especially since threat
actors can just spam prompts over and over until the employee accepts to make it
stop.<br /><br />You should really use <a
href="https://webauthn.io/">WebAuthn</a> as your two-factor auth solution with
dedicated security hardware like Yubikeys or the security chip burned into most
corp owned devices. Finally there’s a good use for the <a
href="https://www.fsf.org/blogs/sysadmin/the-management-engine-an-attack-on-computer-users-freedom">manglement
engine</a> that Stallman said was going to literally destroy
Linux!</xeblog-conv>
</details>

Authentication is a hard problem for humans. It is even harder for computers. In
general though, we have three methods or "factors" that we can use to do
authentication:

* Something you are, like a retina or fingerprint scan
* Something you know, like a password
* Something you have, like a hardware two-factor auth token or authentication
  app that supports push notification based two-factor authentication
  
Most of the time you don't see retina and fingerprint scans required to log into
Facebook. For a lot of people it's usually just that single password between
anyone and their entire reputation being spoiled. There are efforts to try and
get hardware authentication tokens into the hands of as many people as possible,
but those are honestly a bad UX (user experience) compared to the "simplicity"
of using a password. I doubt I'd be able to explain how to use a yubikey to my
grandmother. Much less how to use a TOTP code generator.

<xeblog-conv name="Cadey" mood="coffee">I want to stress that I am not a
security engineer, I am just some random person on the internet who is tired of
everything going to shit because someone got had.</xeblog-conv>

I think that issuing everyone in the company a
[Yubikey](https://www.yubico.com/) and making every internal system work with
that would be a better option. I think this because of the core problem of
phishing: it works best when you are less vigilant. Many two factor
authentication mechanisms lend themselves to phishing because of how they work.
Here are my cynical thoughts about some common ones.

<xeblog-conv name="Cendyne" mood="snoot">Hello there! I’ll be contributing to
this article too. In fact, Google partnered with Yubikey to deploy security keys
to all staff and contractors, they reduced security incidents ten fold by moving
to security keys. You can find out more on [Google Eliminates Account Takeover
with
YubiKey](https://www.yubico.com/resources/reference-customers/google/).</xeblog-conv>

## TOTP Codes

Many online services support [TOTP code
generation](https://www.twilio.com/docs/glossary/totp) as a two-factor
authentication mechanism. These work by sharing a secret value between the
client and server. Both sides take this secret, put in the current time and
generate a six digit code that way.

<xeblog-conv name="Cendyne" mood="laptop-thinking">When you sign up for a TOTP
on a service, the server may change a few defaults when it generates the shared
secret (see [RFC6238](https://www.rfc-editor.org/rfc/rfc6238)) like what hash
function to use and how long the code should be. These codes are often tolerant
of time differences, so multiple codes are valid at the same point for a given
shared secret.</xeblog-conv>

However, six digits is a very small space to search through when you are a
computer. The biggest problem is going to be getting lucky, it's quite literally
a one-in-a-million shot. Turns out you can brute force a TOTP code in [about 2
hours if you are careful](https://pulsesecurity.co.nz/articles/totp-bruting) and
the remote service doesn't have throttling or rate limiting of authentication
attempts.

<xeblog-conv name="Numa" mood="delet">Guess what most services don't have
lol.</xeblog-conv>

<xeblog-conv name="Cadey" mood="facepalm">You've got to be kidding me. You
honestly, seriously, have got to be kidding me.</xeblog-conv>

<xeblog-conv name="Cendyne" mood="panic2">Account specific rate limits are not
enough! An attacker could go after multiple accounts at once from multiple IP
addresses and get in any one of them. Most of the time users enter a TOTP
_after_ a password. Consider forcing a password reset after five failed
subsequent code attempts.</xeblog-conv>

Oh not to mention, the way that TOTP codes work basically means that it's
trivial for an attacker to create an identical flow to your authentication setup
on a phishing site hosted on a bulletproof host and then sniff credentials that
way. Once the phisher has your password and a TOTP code, they can just use it
and redirect you to the actual login form, making you think it failed and you
need to try again. The protocol practically enables phishing with relative ease.
It's horrifying that this is the security best practice for Discord and Twitter
users.

<xeblog-conv name="Cadey" mood="coffee">Don't even get me started on how messed
up things like SMS based two factor authentication are. They use OTP codes on
the backend for them, but then send them over the one carrier channel that is
the most likely to have [support be
phished](https://www.vice.com/en/article/3kx4ej/sim-jacking-mobile-phone-fraud)
in order to have an attacker intercept those codes. It's a mess. I'm amazed that
it was allowed to happen in the first place. The early 2010’s were
wild.</xeblog-conv>

<xeblog-conv name="Cendyne" mood="thumbs-down">It’s 2022 and this is still a
thing. I think many users and developers are conditioned to think this is a safe
implementation because everyone is still doing SMS OTPs.</xeblog-conv>

<xeblog-conv name="Cendyne" mood="stonks">It took us years to move user
expectations towards unique passwords, social login and single sign on. A few
big companies led the industry by example after recognizing the risk shared
passwords posed. The rest who resist hip and trendy changes respond to
regulatory and industry standards like PCI-DSS. <em>At least these industry
standards include password management training (even if some of it is bad, like
rotating passwords constantly).</em></xeblog-conv>
 
<xeblog-conv name="Cendyne" mood="gimme2">
We may see greater adoption of phishing resistant technology when it is cheap, widely available, and a part of industry standards or regulations.
Most phones come with a trusted platform module (TPM) chip or secure enclave.
Mac has had a secure enclave since 2020.
Windows 11 now requires a TPM by default.
So I believe that platform support will be considered available to the masses within a year or two.
</xeblog-conv>

<xeblog-conv name="Cadey" mood="coffee">Until then, we continue to see more and
more reports of stolen credentials causing untold amounts of damage to
companies. Both reputationally and financially. It should not take a company
almost getting hacked out of existence to make the security state of the world
better, but sometimes that’s what it takes to take it seriously. I really hate
that this is the case.</xeblog-conv>

## Yubikey Press

Yubikeys have multiple authentication methods built in. One of them is a legacy
authentication method that makes the yubikey pretend to be a keyboard and type
out a complicated long code based on a private key burned into the firmware of
the device. This can also be phished. This was the main barrier between any
employee and arbitrary user accounts at a past job. You got a corp issued
yubikey and you had to use yubikey presses to get into secure areas of the admin
panel.

This is also phishable, such as by shitposters on twitter:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In honor of the Uber breach, it&#39;s time for a yubikey press thread!<br><br>vvccccfrjcfnkkncrcuujlhktiredllitinttnkuddrh</p>&mdash; Xe Iaso | @cadey@pony.social (@theprincessxena) <a href="https://twitter.com/theprincessxena/status/1570626940989763584?ref_src=twsrc%5Etfw">September 16, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<xeblog-conv name="Mara" mood="wat">Wait, people fell for this? Holy
crap.</xeblog-conv>

<xeblog-conv name="Cadey" mood="coffee">Apparently people call this a
"yubisneeze". It happens more often than you'd think. I really hope that
shitpost didn't cause anyone's production environment to get breached. It'd be
hilarious if this actually got added to any security training guides. You can
disable the OTP interface by following [this
guide](https://support.yubico.com/hc/en-us/articles/360013714379-Accidentally-Triggering-OTP-Codes-with-Your-Nano-YubiKey)
from Yubico.</xeblog-conv>

<xeblog-conv name="Cendyne" mood="heartburn">Check out [OTPs
Explained](https://developers.yubico.com/OTP/OTPs_Explained.html) from Yubico. A
"yubisneeze" has several pieces of info in it, such as a counter to prevent
replay. However, it does not contain the time. As a timeless token, it may be
scraped and silently submitted elsewhere in the future. The only mitigation
would be to immediately verify a newer token with Yubico. Yubikey OTPs are
technically interesting but I am not convinced they raise the bar for security
proportionate to their proprietary implementation.</xeblog-conv>

<xeblog-conv name="Numa" mood="delet">vvccccfrjcfnncggttgrbgnevtrfcljdrrkntttcgrun!</xeblog-conv>

So those are out of the question when it comes to protecting production access.
All similar code based two-factor authentication methods suffer from the same
phishablity problem.

## Notification Fatigue As-A-Service

Another common way to do two factor authentication is to have a user sign into a
mobile app. After that, when you sign in on your computer, it sends a push
notification to your phone. Then you accept the authentication and you can go
post your minecraft seeds to facebook marketplace or whatever.

Google and Facebook have these available as two-factor authentication methods.
This means that a significant percentage of people _on the planet_ likely use
this authentication method. It is hilariously insecure in practice but makes
people _think_ that it is safe. I mean, you trust your phone, right?

Apple is another example, but instead of an app they hook into the desktop and
mobile operating system with your iCloud account. Though, Apple requires a code
to be manually relayed into the other device, which makes these a little harder
to accidentally accept. Nothing a little phishing prep can’t get around.

<xeblog-conv name="Numa" mood="delet">Codes are lame, but it's the future!
Everyone has phones now! People trust their phones. Can't we just have the
computer spit out a notification to a person's phone so they use _that_ for
authentication instead of having to type in a code?</xeblog-conv>

<xeblog-conv name="Mara" mood="hmm">Wait, isn't that whole "send a push
notification" thing abusable? Couldn't a hacker send like a billionty push
notifications over and over until the poor user hits accept to make it
stop?</xeblog-conv>

Allegedly that's what the Uber hacker did. They spammed two factor auth
notifications over and over for an hour while the poor victim was trying to
sleep. You know, when your guard is down by instinct and you are more likely to
act on instinct to just make the noise box shut up. Honestly as an information
security professional I have to almost give that attack method credit,
especially the part where they sent the person a message on WhatsApp pretending
to be IT saying they need to approve the request to secure access to their
account. It's ingenious and I'd probably fall for it.

Maybe we should consider this entire two factor authentication mechanism to be
harmful towards reaching security goals. It looks impressive to executives, who
are the ones that are usually making the decisions about the security products
for some reason.

<xeblog-conv name="Cendyne" mood="take-my-money">There is a lot of money
sloshing around security vendors who fail to provide secure software in the most
basic sense. For example,
[CVE-2022-1388](https://packetstormsecurity.com/files/167007/F5-BIG-IP-Remote-Code-Execution.html)
allowed anyone to run shell commands on a public firewall product. The real
money seems to be in selling "security" rather than providing security.
</xeblog-conv>

<xeblog-conv name="Numa" mood="delet">You could probably make a killing on
selling literal snake oil in the security scene! So much seems to add theatre
and inconvenience in the name of being secure". It’s laughable but it’s the same
kind of thing that leads to alarm fatigue as a service to begin
with.</xeblog-conv>

<xeblog-conv name="Cendyne" mood="wheeze">You got me there. The most efficient
code is code that doesn’t exist after all.</xeblog-conv>

---

So you may be wondering something like:

<xeblog-conv name="Mara" mood="hmm">What can we even do to make authentication
safer? Push notifications make people suffer from alarm fatigue. Code-based
solutions can be phished. What _can_ we use to help solve this
_today_?</xeblog-conv>

I'm not really sure what the best solution is here, but I can suggest something
that I think would help reduce harm: [Webauthn](https://webauthn.io/).

<xeblog-conv name="Cadey" mood="coffee">Please note that we are intentionally
focusing on what we can do _today_, because today is what we have. Tomorrow
doesn’t exist yet.</xeblog-conv>

<xeblog-conv name="Cendyne" mood="whoa">"Tomorrow" we are likely to see passkeys
displace yubikey and other roaming hardware tokens. [Apple
Passkeys](https://developer.apple.com/passkeys/) and Google [FIDO authentication
with passkeys](https://developers.google.com/identity/fido) are coming soon to
let your phones authenticate your computer as a second factor. If you’d like to
hear more, check out this podcast [Passkeys featuring Adam
Langley](https://securitycryptographywhatever.buzzsprout.com/1822302/11122508-passkeys-feat-adam-langley).</xeblog-conv>

<xeblog-conv name="Cendyne" mood="surprised-pikachu">But uh… Good luck buying
any hardware tokens in bulk right now. After the Uber incident, many are
emptying Yubico's stock.</xeblog-conv>

## WebAuthn

[WebAuthn](https://webauthn.io/) is a protocol for using hardware devices in
order to authenticate users by proof of ownership. The basic idea is that you
have a hardware security module of some kind (such as an iPhone’s Secure Enclave
or a Yubikey) that contains a private key, and then the server validates
signatures against the public key of the device to authenticate sessions. It is
also set up so that phishing attacks are impossible to pull off, each WebAuthn
registration is bound to the domain it was set up with. A keypair for
`xeiaso.net` cannot be used to authenticate with `evil-xeiaso.net`.

The user experience is fantastic. A website makes a request to the
authentication API and then the _browser_ spawns an authentication window in
such a way that cannot be replicated with web technologies. The _browser_ itself
will ask you what authenticator you want to use (usually this lets you pick
between an embedded hardware security module or a USB security key) and then
proceed from there. It is impossible to phish this. Even if the visual styles
were copied, the authenticator will do nothing to authenticate the browser!

<xeblog-picture path="blog/webauthn-test"></xeblog-picture>

Since the browser may not know which authenticator the user intends, it will
prompt the user for the platform authenticator or for a roaming authenticator
like a Yubikey. Platform authenticators combine biometrics like Face ID, Touch
ID, or Windows Hello with the trusted platform module or secure enclave to
authenticate. While roaming authenticators may require a presence test and
optionally a PIN. This extra step also protects the user’s privacy from any
drive-by client calls. When a biometric or PIN is verified, the server receives
a flag that the user has been verified. Biometric and PIN data is not sent to
the server. A standard yubikey without a PIN will appear as unverified.

<xeblog-conv name="Mara" mood="happy">To find out more about WebAuthn, I’d
suggest checking out the BlackHat talk ["Demystifying
WebAuthn"](https://www.youtube.com/watch?v=RFACQvL_8S4) ([slide
deck](https://i.blackhat.com/USA-19/Thursday/us-19-Brand-WebAuthn-101-Demystifying-WebAuthn.pdf)).
It will help you get an understanding of how it works.</xeblog-conv>

The work continues on making WebAuthn easier and better to use. Apple recently
released [Passkeys](https://developer.apple.com/passkeys/) with iOS 16 that will
allow you to use your iPhone as a hardware authenticator for any other device,
including iPads and Windows machines. This effectively turns your iPhone’s
Secure Enclave into a roaming security key that you use on other machines,
giving you the best of both worlds. You benefit from Apple’s industry-leading
on-device security processors _and_ also have the ability to use that on your
other machines too. All the existing guarantees of WebAuthn are carried over,
including the fact that each WebAuthn credential is bound to a single website.
Today, you can choose "Add a new Android phone” in Chrome and scan it with your
iOS device. Safari has not caught up yet, we expect this later.

<xeblog-conv name="Cendyne" mood="hooray">WebAuthn’s strength against phishing
is built on origin binding, that is whatever the user’s browser sees as the
website is what the authenticator sees. Another website? Another key! One site’s
WebAuthn authentication cannot be used on another site, since the origin (the
site domain) is bound to the key that the authenticator uses! Not only that, but
the user too!</xeblog-conv>

<xeblog-conv name="Cendyne" mood="guess-i-will-die">Though if the site does not
bind usernames (or uses a dummy username) when authenticating, you will have a
collision between users.</xeblog-conv>

---

In conclusion, push notifications for authentication should be considered
harmful. You should not use them and you should prioritize moving towards
hardware authentication tokens such as Yubikeys. It is worth the hardware and
training cost to do this.

---

<details>
  <summary><xeblog-conv name="Mara" mood="hacker">This part of the article is optional.</xeblog-conv></summary>

As a bonus, here's one of the ways that the web3 people get this kind of thing
more right than wrong. They use the Ethereum cryptosystem as an authentication
factor. 

## EIP-4361: Sign-In with Ethereum

WebAuthn is a protocol where you get a hardware element to sign a message to
prove that you own the keypair in question, and that allows authentication to
happen via the "something you own" flow. In Ethereum and other blockchain
ecosystems, everyone has a keypair that signs messages for instructions like
"transfer an NFT to another address" or "send this address some money". This is
enough to let you construct an authentication factor. The strength of this
authentication factor is...questionable, but by conforming to
[EIP-4361](https://eips.ethereum.org/EIPS/eip-4361), you can turn an Ethereum
keypair into an authentication factor for web applications. This will hook into
a hardware wallet with WalletConnect or a software wallet with a browser
extension like [MetaMask](https://metamask.io). This works in a very similar way
to how WebAuthn works, but with these core principles:

* Authentication is done by confirming that the user has access to a private key
  to sign a message that is validated with the public key, much like WebAuthn.
* Wallet apps show you the URL of the site you are trying to authenticate with
  and there is no way to easily forge that.
* Generating a new Ethereum keypair is free and anyone can do it without having
  to purchase hardware to act as key escrow.

<xeblog-conv name="Mara" mood="hacker">The last point is significant because
export restrictions on cryptography can make it _very difficult_ for people in
countries like Russia to purchase FIDO2 keys. Cryptography isn’t treated as a
munition legally anymore, but it is something powerful enough to give government
people pause. For example, OpenBSD is based out of Canada just in case export
laws about cryptography in the US become relevant again.</xeblog-conv>

But due to the implementation of all of this, it has the following weaknesses:

* There’s no protocol level way to tell what kind of secure element the user is
  using, if any. There are several different types of hardware "cold wallets"
  and one of the most commonly used strategies is to make a "hot wallet" that’s
  just a keypair managed by a browser extension such as MetaMask.
* In many web3 applications, the Ethereum keypair is _the only authentication
  factor_. This means if someone manages to get access to your keypair or
  recovery phrase somehow, they can [steal your
  apes](https://twitter.com/joshuamartian/status/1460929458152431622) and you
  can’t get them back without trying to negotiate with the threat actor.
* Extracting the private key of an Ethereum address is considered a security
  feature and people are encouraged to write their private keys down on paper
  and store it somewhere safe. The first-time user experience of many Ethereum
  space things will force you to write down the recovery phrase by quizzing you
  on what it is and I can only imagine how many people are doing that with an
  actually secure mechanism. There are some interesting products available for
  doing things like [punching your seed phrase into
  metal](https://shop.ledger.com/products/cryptotag-zeus) so you can put that
  metal object in a safety deposit box.
* This is implemented with browser extensions instead of properly embedded into
  the OS itself, meaning that it is theoretically possible to phish, but it
  looks like that is very difficult in practice.

<xeblog-conv name="Mara" mood="hacker">It’s worth noting that there’s such a
thing as a multi-signature contract wallet where the wallet itself is a smart
contract in the blockchain. An example of this is
[Safe](https://gnosis-safe.io/), which I am told is the most widely used
implementation. This lets you use additional means to ensure that m-of-n private
keys sign a message instead of just putting all of your eggs in one basket with
one key. Doing this does mean that you have to pay a gas fee for every time that
you sign in, executing a smart contract isn’t free.</xeblog-conv>

<xeblog-conv name="Cadey" mood="coffee">Honestly I’m not sure if all of this is
worth it, but at the very least it is _something_ and it is embedded deep enough
into the ecosystem that it’s done _decently_ across multiple websites. There’s a
lot of UX warts involved, but a lot of that is kind of endemic to grafting
something onto browsers after the fact. However it is _something_ and it is more
than just a TOTP code.</xeblog-conv>

<xeblog-conv name="Numa" mood="happy">You can use one of those fancy hardware
bitcoin wallets as a WebAuthn credential though!</xeblog-conv>

</details>
