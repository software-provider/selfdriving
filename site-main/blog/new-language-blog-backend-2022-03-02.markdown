---
title: "Want to Learn a New Language? Write a Blog Backend!"
date: 2022-03-02
tags:
 - beginner
 - backend
author: Twi
---

Most of the content on my blog requires a fair bit of technical context to
understand. This post is aimed at beginners as well as people with a lot more
experience in the industry. It also covers a question that I get asked a lot:

[What is a good project to use for learning a new programming language? It seems
like a really open ended question and it can be hard to see where I would end up
with such a thing.](conversation://Mara/hmm)

Here is a project that you can use as an end goal for learning to program in a
new language: I call it Within's Minimal Viable Blog. With this project you will
build a blog engine that can also function as a portfolio site.

[A portfolio site is a personal website where you can show off what you've done
and helps you stand out from the crowd. The blog you are reading right now is a
perfect example of a portfolio site. These kinds of sites are also important in
a "keeping the internet weird" sense. You can do a lot more flair and
customization to a website you own than you can to a Twitter profile or
similiar.](conversation://Mara/happy)

It's also designed to make you dip your toes into a lot of commonly used
technologies and computer science fundamentals in the process. Namely it makes
you deal with these buzzwords:

- State management (remembering/recalling things persistently)
- Basic web serving
- HTML templating
- Static file serving
- Input sanitization (making sure that invalid input can't cause JavaScript
  injections, etc)
- Sessions (remembering who a user is between visits to a page)

[You can rip out a lot of this and still end up with a viable result, such as by
making a static site generator. However if you have never done something like
this before I'd be willing to argue that it's well worth your time to attempt to
do something like this at least once.](conversation://Cadey/enby)

At a high level, here's what you should end up with:

- An abstract "Post" datatype with a title, publication date, a "URL slug" and
  content in plain text
- A home page at `/` that describes yourself to the world
- A list of blog posts at `/blog`
- Individual blog posts at `/blog/{date}/{slug}`
- A `/contact` page explaining how to contact you
- An admin area at `/admin/*` that is protected by a username and password
- An admin form that lets you create new posts
- An admin form that lets you edit existing posts
- An admin form that lets you delete posts
- An admin view that lets you list all posts
- Use a CSS theme you like (worst case: pick one at random) to make it look nice
- HTML templates for the base layout and page details

[All these admin views and forms are needed because they are what allow you to
modify content on the blog. When you upload code to GitHub or use the web
editor, this is the kind of thing that GitHub has implemented on the
backend. You can also likely reuse the "new post" form as the "edit post" form
because they do very similar things.](conversation://Mara/hacker)

For extra credit, you can do the following extra things:

- Add an "updated at" date that shows up if the post has been edited
- Add tags on posts that let users find posts by tag
- [JSONFeed](https://www.jsonfeed.org/) support
- "Draft" posts which aren't visible on the public blog index or feeds, but can
  be shared by URL
- Use CSRF protection on the admin panel
- Deploy it on a VPS and serve it to the internet (use
  [Caddy](https://caddyserver.com/) to make this easier)
- Pagination of post lists

If you've never done a project like this, this may take you a bit longer. You
will have to do some research on the best way to write things in your language
of choice. A good starting language for this is something like Python with
[Flask](https://flask.palletsprojects.com/en/2.0.x/) or Go with the standard
library, namely with [net/http](https://pkg.go.dev/net/http) and
[html/template](https://pkg.go.dev/html/template). This may take you a week or
two. If you've done this kind of thing before, it may take a few days to a week
depending on how much experimentation you are doing.

[What the heck is a "URL slug"?](conversation://Numa/delet)

[Most of the time a "URL slug" means some URL-safe set of characters that help
humans identify what the content is about. If you look at the URL for this
article, you can see that its slug is `new-language-blog-backend`, which is
purely for humans to read. You can either take this as something you generate by
hand on each post, or take the title, remove non-space and non-alphanumeric
characters, lowercase it and then replace spaces with dashes. This would turn
"Hello World!" into "hello-world" or similiar.](conversation://Mara/hacker)

Here are a few steps that may help you get started doing this:

- Serve a static file at `/` that contains `<h1>Hello, world!</h1>`
- Create a SQLite connection and the posts table
- Insert a post into your database by hand with the `sqlite3` console
- Wire up a `/blog/{date}/{slug}` route to show that post
- Wire up `/blog` to show all the posts in the database
- Make static pages for `/` and `/contact`
- Choose a templating language and create a base template
- Edit all your routes to use that base template
- Create the admin login page and a POST route to receive the HTML form and
  check the username/password against something in the SQLite database, creating
  a session for the admin panel
- Create an external tool that lets you manually set your username and password
- Create an admin view that shows all posts
- Create a form that lets you create a new post and its associated POST handler
- Create a form that lets you edit an existing post and its associated POST
  handler
- Use a CSS theme to make it all look nice

There are no wrong ways to do this. You can take shortcuts where you want. You
can use Markdown for formatting posts. You can do anything you want based on
this skeleton and it will be _your_ creation. Nobody else will have something
exactly like this. It will let you stand out as a professional and can help you
create your own home on the internet.

[Something else to keep in mind here is that this first attempt will be ugly as
sin. It will be hacky as all hell. However the important part here is that it
works above all else. You can come back and make it better later. The first pass
is always the most ugly, do not let this discourage you. This is a personal form
of expression. Don't be afraid to let your personality show through your
website. That's what helps to make you stand out in a sea of LinkedIn and
Twitter profiles.](conversation://Numa/happy)

[Yeah, getting started in something you've never really done before can be hard
because you won't really have a good idea of what's "correct". The only
"correct" thing in this project is that you can connect with a browser and see
the things you expect. Take chances, make mistakes, get
messy!](conversation://Mara/happy)

If you do go through with this and want to show it off to people in a followup
post, email your results to
[blogchallenge2022@xeserv.us](mailto:blogchallenge2022@xeserv.us) with the
following information:

- Your name/alias (you don't have to use your legal name)
- How much background you have in programming
- The language you are writing this in
- Screenshots you want to share
- Anything you learned along the way, where you got stuck, what got you unstuck
  and anything else you want to say
- If you uploaded it to GitHub, the repository URL would be nice but is not
  required
- If you deployed it to a server, the link to where it's running would also be
  nice but is definitely not required

The road to expertise is paved with the effort of experimentation. I hope that
this open-ended project can help you learn things.
