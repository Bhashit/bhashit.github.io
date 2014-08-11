---
layout: post
title: emacs- the text editor of the Jedi
date:   2014-08-11
comments: true
---

Don't get misled by the title. I am not saying that only the alpha-geeks can use [emacs][gnu-emacs]. The reason for that phrasing is that it sounds really cool; much better than something like "emacs: a text editor for any competent programmer (who knows at least one dialect of lisp)". That just sounds lame, even though that's what I think. The title is also a reference to a [Steve Yegge post][steve-yegge-emacs] that I read a long time ago where he says that

> Emacs is to Eclipse as a light saber is to a blaster — but a blaster is a lot easier for anyone to pick up and use

Besides, I don't think I come even close to be considered a Jedi when it comes to programming. I am closer to an apprentice who just got his hands on a lightsaber and is just trying poke it around; see what cool things he can do. Poking around is what I have been doing with emacs for a week; and after the first day, I have been thinking why I didn't start doing  it sooner.

Whenever I have tried to learn some new or non-mainstream programming language, like `scheme`, getting a decent text editor has always been a problem. Most of the time I have used [vim][vim]. And `vim` is a great editor by all accounts. I have been using it ever since I switched to Linux 7-8 years ago. However, I never got around to learning any advanced features. Neither did I ever learn `vimscript`. I tried learning `emacs` by following the built in tutorial twice in the past, but never got any further than that. Learning to wield one of these two editors well had been in my todo list for at least 3-4 years.

Then it finally happened, the [`M-x butterfly`][emacs-xkcd] was pressed somewhere and the forces of the nature combined to crash my [scala-ide][scala-ide] twice in the span of a few minutes, while I was working on an important side project. With eclipse, crashing is something that happens on almost daily basis with me. But this time, there were two things different:

1. I had a few days on my hand.
2. I was using a standalone `scala-ide` download for my work, so it wasn't installed as a plugin in eclipse.

Let me explain why the second point is important. The side project I was working on was something I wanted to get done before the end of the week. Whenever I have used eclipse with `scala-ide` installed as a plugin, it has been incredibly slow (I have a Core-i7 with 16 cores and 8 GB RAM). I mean, it would just freeze in the middle of some editing, with at least 2 GB heap size.  And of course, crashing once or twice a day was quite usual. So, I decided to use a standalone `scala-ide`. The standalone version worked well and froze much less, the problems were not eliminated. Not to mention that fact that if I wanted to work on `clojure` I would have to use a separate instance of eclipse. The whole situation was pretty frustrating [^1].

## FAQ: Why not switch to IntelliJ? It's frickin' awesome.

It might be, byt there are several reasons I never switched to IntelliJ.

1. Eclipse was used at my previous job
2. [Spring Tool Suite][SpringToolSuite] is built on top of pse. It's also developed by the same people who develop the [Spring Framework][SpringFramework].
3. [`scala-ide`][scala-ide] is developed by [TypeSafe][type-safe]], whose founder is Martin Odersky; the man who created Scala.
4. When I started developing in `clojure`, [counteclockwise][counterclockwise] was overall the best choice, and it's an eclipse plugin. This plugin was also endorsed by the authors of one of [the best books on clojure][clojure-prog].
5. The official clojure plugin for IntelliJ is only available with the paid version.
6. Choosing IntelliJ, or any other popular IDE/editor,  doesn't solve the problem of working with non-mainstream languages.

I haven't used IntelliJ much, so, there is still the question of whether it suffers from the same crashing/performance problems as eclipse when several plugins are active. I've had enough of that.

## FAQ: Why choose emacs over vim?

In one word: [`elisp`][elisp], or [`emacs-lisp`][elisp]. I have learned three different dialects of Lisp: clojure, common-lisp and scheme. When I started learning `emacs` this week, the one thing that I found out about myself was that I can read `elisp` code as easily as if I have been using it for years. Even though, I hadn't even gone through the introductory material on `elisp`. Yeah, it was that easy. I just looked at the code and it was instantly readable. I surprised even myself.

I could easily find out about all the internal wiring using the incredible help commands that `emacs` has built in (`M-h ?`). There are also [apropos][apropos] commands that allows easy searching. Writing `elisp` and immediately seeing the changes in `emacs`' behavior is incredibly satisfying. `vim` users, please don't get your panties in a bunch, it's an accepted fact that extending `emacs` is much easier than extending `vim`. I have done my research.

Someone might argue that I chose emacs because I knew lisp already. Well, that just might be the case. But, even if you don't already know some dialect of lisp, I think picking up one is very easy. The syntax is pretty minimal. Although, writing idiomatic code could take some time, especially if you are coming from the java (or C#, or any other imperative language) world, like me. But that shouldn't deter you. Learning lisp, and especially lokking at some experienced lisper's code, is an enlightening experience in itself. As Eric Raymond put it in his [very famous essay][eric-raymond-lisp],

> LISP is worth learning for a different reason — the profound enlightenment experience you will have when you finally get it. That experience will make you a better programmer for the rest of your days, even if you never actually use LISP itself a lot.

So, yes, lisp was the primary reason I chose `emacs`. That and the feeling that `emacs` was built for extending. With this choice, I hit two targets: I get to use an extensible text editor, and I get to use lisp.

I think there are several other points which affected this decision. Not the least of which was the fact that I am a hero worshipper. Like some kid who buys Nike because his favorite sport star endorses it, I am inclined towards `emacs` because several hakers I admire, use it. Another was the fact that emacs has facilities for everything. The famous `emacs` joke describes it as an "operating system with a text-editor built in" or some variation thereof. I like the fact that I can use a terminal emulator directly from emacs, and I can commit to my git repo with a few keystrokes, or I can edit remote files (I havent tried that yet), without leaving the editor. The list of things you can do from the editor are endless. It's pretty common for experienced (smug and grey-bearded, with optional pony tail) `emacs` users to discover some surprising functionality that they have never used in the past.


## FAQ: Should I switch all my development work to `emacs`

I don't think so. Especially not if your primary development langauge is a mainstream language like java or C#. For these languages, the IDEs are pretty much irreplacable. Even if you try to match the functionality of eclipse or IntelliJ for java development in `emcas`, you will be severely disappointed. Of course, there is nothing that you cannot do in `emacs`. Everything is possible, but the fact is that building all of the features of those IDEs is an extremely difficult task. And it's very time consuming. If an existing IDE for your developement langauge is not mature or stable enough, by all means, go to, `emacs`. But switching to `emacs` for enterprise java development could be a pretty bad idea. I am switching to `emacs` for scala since `emacs` has become a pretty much an IDE for `scala` with [ensime][ensime]. However, my primary purpose in learning `emacs` is to have a great editor for non-mainstream languages like [`idris-lang`][idris-lang] or [`Go`][go-lang]. For `scala`, I might still go back to `scala-ide` later, once it becomes stable enough. Meanwhile, I am thinking of building an `emacs` extension to support [`play-framework`][play-framework].

## What about LightTable? It seems pretty cool

Yup. [LightTable][LightTable] is something I have been using for 3-4 months for `clojure` development; and it's pretty awesome to say the least. You have to see it to believe it. I mean, really, just take a look at the introduction video on [that site][LightTable], you _will_ be impressed.  Not to mention that fact that it's open-source and it can be tinkered around by writing [`ClojureScript`][ClojureScript]. And I like Clojure.

However, the number of available plugins is still pretty low. The plugin development API is still not available, so extending it yourself is not an option right now. So, until it becomes extensible, I am sticking with `emacs`. Although, I am eagerly waiting for the time when that happens. `emacs` is great, but it's still very old, and sometime archaic. LightTable has all the right ingredients to become the "next emacs": It's written in `ClojureScript`, which is a dialect of lisp; It's open source; and the lead developer, [Chris Granger][chris-granger] seems to have exactly the right kind of attitude. I, for one, am going to keep a close eye on the progress of that text editor.

## emacs is fun

`emacs` is fun because emacs is actually a bunch of lisp code that you can really debug. If your `emacs` init file is not loading corretly, you can actually debug it. While working in the editor, you can execute commands and change the behavior of the editor instantly. You can change key-bindings and you can do whatever crazy stuff that comes to your mind. Even when using IDEs, people try to customize every last bit of functionality through GUI. Hackers love to have their environment configured to the very last bit. And `emacs` takes this configurability to a whole new dimension. You actually write little pieces of code to configure it, and every single piece of wiring can be taken apart and reconnected somewhere else. Of course, you sometimes risk breaking something, but that's a huge part of the fun. Being able to do so is rewarding in the same sense that it is rewarding to use linux when using windows could make several things much easier.

## emacs as your first editor

I read somewhere that beginner users shouldn't try to use "difficult" editors like `emacs`  when they are just starting out. This might have some truth in it. Nowadays, a newbie programmer is generally learning at least 4-5 different things at the same time. Entering any programming language eco-system can be intimidating at first, especially if you know how much there is to learn. _Learning_ to use an editor on top of that could just add more cognitive load. However, I think that using `emcas` instead of some IDE at the very beginning of your career can be extremely rewarding. To reduce the learning burden, they can just enable syntax highlighting and use it like a plain vanilla text editor. Once they get fairly confortable, they can start tinkering around with all the wiring. By the time they become professional hackers, they would be pretty good at it. Being comfortable with a lisp and its programming paradigm since the beginning of your career can be invaluable. Being proficient with `emacs` automatically translates into having at least a decent development environment for any language you choose. And that is a benefit worth having becasue learning new languages is an essential part of becoming a good hacker. After spending just a week with `emacs`[^2], I, for one, would have been really grateful if somebody had given me this advice when I started learning programming.


IDEs are here to stay, no doubt about that. But real hackers are almost always learning new things, especially programming languages, and if you are one of them, it will pay to learn to use at least one great text editor really well. I am not saying that go with `emacs`. You can choose `vim`. I have never seen an expert `vim` or `emacs` user working but I have heard that they can do pretty incredible things.  I cannot say much about `vim` even though it has been a constant companion. But if you do learn `emacs`, it would definitely be time well spent.

## Footnotes

[^1]: I know, eclipse, and almost all the other development tools are free, and nobody has a right to complain about the free things they get. I am not trying to be ungrateful to the developers of these great tools at all. I am extremely grateful, actually. But, even with free tools, I think people can still get frustrated.

[^2]: Although, the week I spent with emacs was not a normal 9-to-5, 5 days a week kind of week. It was a hacker week, in which I spent almost all my waking hours digging on the internet and experimenting.


[vim]: http://www.vim.org/ "vim"

[gnu-emacs]: http://www.gnu.org/software/emacs/

[steve-yegge-emacs]: https://sites.google.com/site/steveyegge2/you-should-write-blogs

[emacs-xkcd]: http://xkcd.com/378/

[scala-ide]: http://scala-ide.org/

[SpringToolSuite]: https://spring.io/tools/sts

[SpringFramework]: https://spring.io/

[counterclockwise]: https://code.google.com/p/counterclockwise/

[clojure-prog]: http://www.amazon.com/Clojure-Programming-Chas-Emerick/dp/1449394701/ref=sr_1_1?s=books&ie=UTF8&qid=1407764385&sr=1-1&keywords=clojure+programming

[type-safe]: https://typesafe.com/

[elisp]: https://www.gnu.org/software/emacs/manual/eintr.html

[apropos]: https://www.gnu.org/software/emacs/manual/html_node/emacs/Apropos.html

[eric-raymond-lisp]: http://www.catb.org/esr/faqs/hacker-howto.html

[ensime]: https://github.com/ensime/ensime-server

[idris-lang]: http://www.idris-lang.org/

[go-lang]: http://golang.org/

[play-framework]: http://www.playframework.com/

[LightTable]: http://www.lighttable.com/

[ClojureScript]: http://clojure.org/clojurescript

[chris-granger]: http://www.chris-granger.com/

