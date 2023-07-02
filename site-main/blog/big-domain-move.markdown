---
title: "Site Update: The Big Domain Move To xeiaso.net"
date: 2022-05-28
tags:
 - dns
---

Hello all!

If you take a look in the URL bar of your browser (or on the article URL section
of your feed reader), you should see that there is a new domain name! Welcome to
[xeiaso.net](https://xeiaso.net)!

Hopefully nothing broke in the process of moving things over, I tried to make
sure that everything would forward over and today I'm going to explain how I did
that.

I have really good SEO on my NixOS articles, and for my blog in general. I did
not want to risk tanking that SEO when I moved domain names, so I have been
putting this off for the better part of a year. As for why now? I got tired of
internets complaning that the URL was "christine dot website" when I wanted to
be called "Xe". Now you have no excuse.

So the first step was to be sure that everything got forwarded over to the new
domain. After buying the domain name and setting everything up in Cloudflare
(including moving my paid plan over), I pointed the new domain at my server and
then set up a new NixOS configuration block to have that domain name point to my
site binary:

```nix
services.nginx.virtualHosts."xeiaso.net" = {
  locations."/" = {
    proxyPass = "http://unix:${toString cfg.sockPath}";
    proxyWebsockets = true;
  };
  forceSSL = cfg.useACME;
  useACMEHost = "xeiaso.net";
  extraConfig = ''
    access_log /var/log/nginx/xesite.access.log;
  '';
};
```

After that was working, I then got a list of all the things that probably
shouldn't be redirected from. In most cases, most HTTP clients should do the
right thing when getting a permanent redirect to a new URL. However, we live in
a fallen world where we cannot expect clients to do the right thing. Especially
RSS feed readers.

So I made a list of all the things that I was afraid to make permanent redirects
for and here it is:

* `/jsonfeed` - a JSONFeed package for Go
  ([docs](https://pkg.go.dev/christine.website/jsonfeed)), I didn't want to
  break builds by issuing a permanent redirect that would not match the
  [go.mod](https://tulpa.dev/Xe/jsonfeed/src/branch/master/go.mod) file.
* `/.within/health` - the healthcheck route used by monitoring. I didn't want to
  find out if NodePing blew up on a 301.
* `/.within/website.within.xesite/new_post` - the URL used by the [Android
  app](https://play.google.com/store/apps/details?id=website.christine.xesite)
  widget to let you know when a new post is published. I didn't want to find out
  if Android's HTTP library handles redirects properly or not.
* `/blog.rss` - RSS feed readers are badly implemented. I didn't want to find
  out if it would break people's feeds entirely. I actually care about people
  that read this blog over RSS and I'm sad that poorly written feed readers
  punish this server so much.
* `/blog.atom` - See above.
* `/blog.json` - See above.

Now that I have the list of URLs to not forward, I can finally write the small
bit of Nginx config that will set up permanent forwards (HTTP status code 301)
for every link pointing to the old domain. It will look something like this:

```nginx
location / {
  return 301 https://xeiaso.net$request_uri;
}
```

<xeblog-conv name="Mara" mood="hacker">Note that it's using `$request_uri` and
not just `$uri`. If you use `$uri` you run the risk of [CRLF
injection](https://reversebrain.github.io/2021/03/29/The-story-of-Nginx-and-uri-variable/),
which will allow any random attacker to inject HTTP headers into incoming
requests. This is not a good thing to have happen, to say the
least.</xeblog-conv>

So I wrote a little bit of NixOS config that automatically bridges the gap:

```nix
services.nginx.virtualHosts."christine.website" = let proxyOld = {
    proxyPass = "http://unix:${toString cfg.sockPath}";
    proxyWebsockets = true;
  }; in {
  locations."/jsonfeed" = proxyOld;
  locations."/.within/health" = proxyOld;
  locations."/.within/website.within.xesite/new_post" = proxyOld;
  locations."/blog.rss" = proxyOld;
  locations."/blog.atom" = proxyOld;
  locations."/blog.json" = proxyOld;
  locations."/".extraConfig = ''
    return 301 https://xeiaso.net$request_uri;
  '';
  forceSSL = cfg.useACME;
  useACMEHost = "christine.website";
  extraConfig = ''
    access_log /var/log/nginx/xesite_old.access.log;
  '';
};
```

This will point all the scary paths to the site itself and have
`https://christine.website/whatever` get forwarded to
`https://xeiaso.net/whatever`, this makes sure that every single link that
anyone has ever posted will get properly forwarded. This makes link rot
literally impossible, and helps ensure that I keep my hard-earned SEO.

I also renamed my email address to `me@xeiaso.net`. Please update your address
books and spam filters accordingly. Also update my name to `Xe Iaso` if you
haven't already.

I've got some projects in the back burner that will make this blog even better!
Stay tuned and stay frosty.

What was formerly known as the "christine dot website cinematic universe" is now
known as the "xeiaso dot net cinematic universe".
