---
layout: post
title:  Suffering from the geek syndrome.
date:   2014-05-25 16:12:30
categories: learning programming
---

First of all, this is just a rant, and a way to start writing something, anything. So, you can probably go away and do something better with your time. So that your visit is not wasted, [here's a really funny paper][funny-link] from the [funniest man in Microsoft research][funniest-man].

So, I still have no idea what I am going to write about. Let me begin with the act of *writing* itself. I have been meaning to start a blog for at least 5 years now. I knew that writing about things may help you understand them better. I was also pushed in that direction further by Steve Yegge's [very old post][steve-yegge-writing], and countless other influential programmers, including Joel Spolsky in [this post][spolsky-writing]. I write notes about things that I learn or create. I have several different notebooks (yeah, the paper ones)  on several topics. Also, I have several notes in evernote, tomboy, one-note and plain text files. Even after all these, I never managed to start a blog. I knew, writing a blog-post would have been easier, and more flexible, than having hand-written notes with code-samples or writing in evernote. So, I come to a point where I'm tired of the constraints of writing notes without an editor. So, then, I decide to start writig a blog; and here's a sample of how it goes:
1. Let me create my own blog engine. Use Clojure, or probably Scala with the play framework. Yeah, that sounds cool. So, I should start thinking about which features I want to support. Man, this is going to be really great.
2. Ah, I am never going to start writing my own blog engine. Compromise and start using some existing solution. When you do write your own blog-engine, migrate all the posts to it.
3. Decide which blogging platform to use. Tumblr? nope, not my style. Blogger? nope. Wordpress? but I don't want to write PHP. I want flexibility, Wordpress is probably the most flexible blogging platform. So, let's go with wordpress.
4. Download wordpress.
5. start customizing wordpress with plugins and templates.
6. Satisfied? Alright, you can start writing tomorrow.
7. Start writing. What do you want to write about? Well, let's see. Clojure? Yeah, sure. Alright. Where do I begin? Start writing. Oh, I haven't finished that Clojure book yet. Let me go through that first.
8. Six months later: My wordpress install is out of date. Think about all the ugliness of upgrading wordpress after you have written several posts. Go to step 1.

So, after 5 years of this, or some variation thereof, here I am. I am finally pushing my first blog post up. I learned about [creating pages on github][gh-pages], with simple markdown based posts. I was finally sold. Although, this wasn't quite easy. I again considered wordpress, about the flexibility of server-side coding, and decided that this was fine; I explored [jekyll][jekyll], I explored [octopress][octopress]; I learned how to work with the [liquid][liquid] templating engine; I explored ways to customize and add features to the site without server side coding, and several other things. After spending a lot of time on this, I decided that nobody is going to read my blog anyway. Just start. Don't worry about `CSS` and making it `responsive` or some other details. Github pages was made for programmers. I add new posts by doing `git push`. Isn't that cool? So, I started.

However, my point here is that I suffer from what I think should probably be termed **The geek syndrome**. I want to learn everything. Even for the simplest of tasks, I go into a lot of details. Googling for things is not satisfying. I read documentation; I read books; I explore the source code. Even the videos don't quite work for me. I like them, but most of the time, they are just not detailed enough.

I have been helping out on an [open-source project][bigtop] from Apache for the past few days. The project revolves around `hadoop`, `pig` and `mahout`. I didn't know much about `hadoop`. So, I decided to spend this weekend on doing some reading aorund it. This is how it went:
1. Go the [`hadoop` website][hadoop] and download it.
2. Read some docs on how to get started
3. Ah, I don't want to make it run on single-node, I should know everything, so, I should probably setup a multi-node system using the two laptops and a PC that I own.
4. Read up on [`Big Table`][bigtable], [`Google File System`][gfs] and [`MapReduce`][mapreduce]. I read these a few years back, but do a refresher anyway.
5. Oh, I recently read somewhere that [`MapReduce`][mapreduce] is quite old. There are faster solutions out there. Read a little bit [`spark`][spark]. Find blogs comparing the two.
6. Should I really spend so much time on [`hadoop`][hadoop]? I want to learn more about `AI` and stuff. That's why I am learning the `Lisp` family of languages.

The end result is that I feel like I need to spend a lot more time on `hadoop` if I want to use it. I am not really complaining. I like the way I am. I like learning. I like reading. These two are the reasons I am reasonably successful at what I do. And I never get bored, ever. However, I think I should prioritize what want to learn in details. I actually do that all the time. But the thing is, I have to spend some effort trying to convince myself that this is not my highest priority right now.

Generally, I deal with this syndrome by always keeping a single topic as the master topic that I should focus on the most. For ex. right now, it is `Clojure` (and `Sceheme` languages).

The list of things that I want to learn is endless. I'll upload the rest of the post later...

[funny-link]: http://research.microsoft.com/en-us/people/mickens/thenightwatch.pdf
[funniest-man]: http://blogs.msdn.com/b/oldnewthing/archive/2013/12/24/10484402.aspx
[steve-yegge-writing]: https://sites.google.com/site/steveyegge2/you-should-write-blogs
[spolsky-writing]: http://www.joelonsoftware.com/articles/CollegeAdvice.html
[gh-pages]: https://pages.github.com/
[jekyll]: http://jekyllrb.com/
[octopress]: http://octopress.org/
[liquid]: http://liquidmarkup.org/
[bigtop]: https://issues.apache.org/jira/browse/BIGTOP-1269
[hadoop]: http://hadoop.apache.org/
[bigtable]: http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf
[gfs]: http://static.googleusercontent.com/media/research.google.com/en//archive/gfs-sosp2003.pdf
[mapreduce]: http://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf
[spark]: http://spark.apache.org/
[emacs]: http://www.gnu.org/software/emacs/
