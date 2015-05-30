---
layout: post
title: Emacs - the text editor of the Jedi
date:   2014-08-11
comments: true
---

Don't get alarmed by the title. I am not saying that only the
alpha-geeks can use [emacs][gnu-emacs]. The reason for that phrasing
is that it sounds really cool; much better than something like "emacs:
a text editor for any competent programmer". That just sounds lame,
even though that's what I think. The title is also a reference to a
[Steve Yegge post][steve-yegge-emacs] that I read a long time ago
where he says that

> Emacs is to Eclipse as a lightsaber is to a blaster — but a blaster
> is a lot easier for anyone to pick up and use

Besides, I don't think I come even close to be considered a Jedi when
it comes to programming. I am closer to an apprentice who just got his
hands on a lightsaber and is just trying poke it around. Poking around
is what I have been doing with emacs for a week; and after the first
day, I have been thinking why I didn't start doing it sooner.

Whenever I have tried to learn some new or non-mainstream programming
language, like `scheme`, getting a decent text editor has always been
a problem. Most of the time I have used [vi][vim]. And vim is a great
editor by all accounts. I have been using it ever since I switched to
Linux 7-8 years ago. However, I never got around to learning any
advanced features or `vimscript`. I tried learning emacs by following
the built in tutorial twice in the past, but never went any
further. Learning to use one of these two editors well had been in my
todo list for at least 3-4 years.

Then it finally happened, the [`M-x butterfly`][emacs-xkcd] was
pressed somewhere and the forces of the nature combined to crash my
[scala-ide][scala-ide] twice in the span of a few minutes, while I was
working on an important side project. With eclipse, crashing is
something that happens on almost daily basis with me. But this time,
there were two things different:

1. I had a few days on my hand.
2. I was using a standalone `scala-ide` download for my work, so it
   wasn't installed as a plugin in eclipse.

Let me explain why the second point is important. The side project I
was working on was something I wanted to get done before the end of
the week. Whenever I have used eclipse with scala-ide installed as a
plugin, it has been incredibly slow (I have a Core-i7 with 16 cores
and 8 GB RAM). I mean, it would just freeze in the middle of some
editing, with at least 2 GB heap size.  And of course, crashing once
or twice a day was quite usual. So, I decided to use a standalone
scala-ide. The standalone version worked well and froze much less, but
the problems were not eliminated. Not to mention that fact that if I
wanted to work on `clojure` I would have to use a separate instance of
eclipse, since I didn't want to activate any bugs by installing a
_major_ plugin. The whole situation was pretty frustrating [^1].

## FAQ: Why not switch to IntelliJ? It's frickin' awesome.

It might be, but there are several reasons I never switched to IntelliJ.

1. Eclipse was used at my previous job
2. [Spring Tool Suite][SpringToolSuite] is built on top of
   eclipse. It's also developed by the same people who develop the
   [Spring Framework][SpringFramework].
3. [`scala-ide`][scala-ide] is developed by [TypeSafe][type-safe],
   whose founder is Martin Odersky; the man who created Scala.
4. When I started developing in `clojure`,
   [counteclockwise][counterclockwise] was overall the best choice,
   and it's an eclipse plugin. This plugin was also endorsed by the
   authors of one of [the best books on clojure][clojure-prog].
5. The official clojure plugin for IntelliJ is only available with the
   paid version.
6. Choosing IntelliJ, or any other popular IDE/editor, doesn't solve
   the problem of working with non-mainstream languages.

I haven't used IntelliJ much, so, there is still the question of
whether it suffers from the same crashing/performance problems as
eclipse when several plugins are active. I've had enough of that.

## FAQ: Why choose emacs over vim?

In one word: [`elisp`][elisp], or [`emacs-lisp`][elisp]. I have
learned three different dialects of Lisp: clojure, common-lisp and
scheme. When I started learning emacs this week, the one thing that I
found out about myself was that I can read `elisp` code as easily as
if I have been using it for years. Even though, I hadn't even gone
through the introductory material on `elisp`. Yeah, it was that
easy. I just looked at the code and it was instantly readable. I
surprised even myself.

I could easily find out about all the internal wiring using the
incredible help commands that emacs has built in (`C-h ?`). There are
also [apropos][apropos] commands that allows easy searching. Writing
`elisp` and immediately seeing the changes in emacs behavior is
incredibly satisfying. It also seemed to be an accepted fact that
extending emacs is much easier than extending vim.

Someone might argue that I chose emacs because I knew lisp
already. Well, that just might be the case. But, even if you don't
already know some dialect of lisp, I think picking up one is very
easy. The syntax is pretty minimal. Although, writing idiomatic code
could take some time, especially if you are coming from the java (or
C#, or any other imperative language) world, like me. But that
shouldn't deter you. Learning lisp, and especially looking at some
experienced lisper's code, could be a great experience in itself. As
Eric Raymond put it in his [very famous essay][eric-raymond-lisp],

> LISP is worth learning for a different reason — the profound
> enlightenment experience you will have when you finally get it. That
> experience will make you a better programmer for the rest of your
> days, even if you never actually use LISP itself a lot.

So, yes, lisp was the primary reason I chose emacs. That and the
feeling that emacs was built for extending. I get to use a really
malleable text editor _and_ I get to use Lisp.

I think there are several other points which affected this
decision. Not the least of which was the fact that I am a hero
worshipper. Like some kid who buys Nike because his favorite sport
star endorses it, I am inclined towards emacs because several hackers
I admire, use it. Another was the fact that emacs has facilities for
everything. The famous emacs joke describes it as an "operating system
with a text-editor built in". I like the fact that I can use a
terminal emulator directly from emacs, and I can commit to my git repo
with a few keystrokes, edit remote files <del>(I haven’t tried that
yet)</del>, [take notes][org-mode] better than evernote, and even send a
[tweet][twitter-mode] without leaving my editor.  The list of things
you can do from the editor are endless. It's pretty common for
experienced, smug and grey-bearded emacs users to discover some new
functionality. Not to mention the fact that emacs itself keeps
evolving, and the list of packages available for it keeps growing.

## FAQ: Should I switch all my development work to emacs?

I don't think so. Especially not if your primary development language
is a mainstream language like java or C#. For these languages, the
IDEs are pretty much irreplaceable. Even if you try to match the
functionality of eclipse or IntelliJ for java development in emcas,
you will be severely disappointed. Of course, there is nothing that
you cannot do in emacs. Everything is possible, but the fact is that
building all of the features of those IDEs is an extremely difficult
task. And it's very time consuming.

If an existing IDE for your development language is not mature or
stable enough, by all means, go to emacs. But switching to emacs for
enterprise java development could be a pretty bad idea. I am switching
to emacs for scala since it has become a pretty much an IDE for scala
with [ensime][ensime]. However, my _primary purpose_ in learning emacs
is to have a great editor for non-mainstream languages like
[`idris-lang`][idris-lang] or [`Go`][go-lang]. For scala, I might
still go back to `scala-ide` later, once it becomes stable
enough. Meanwhile, I am thinking of building an emacs extension to
support [`play-framework`][play-framework].

## What about LightTable? It seems pretty cool

Yup. [LightTable][LightTable] is something I have been using for 3-4
months for `clojure` development; and it's pretty awesome to say the
least. You have to see it to believe it. I mean, really, just take a
look at the introduction video on [that site][LightTable], you _will_
be impressed.  Not to mention that fact that it's open-source and it
can be tinkered around by writing
[`ClojureScript`][ClojureScript]. And I like Clojure.

However, the number of available plugins is still pretty low. If a
new, interesting langauge arrives, emacs is more likely to have a
plugin for it than LightTable. I know, I _can_ write the plugin
myself, and that's a great idea.  But, before I do something about it,
I need to take care of a few things at the top of my todo list. So, I
am sticking with emacs for the time being. Although, LightTable has
all the right ingredients to become the _next emacs_: It's written in
`ClojureScript`, which is a dialect of lisp; It's open source; and the
lead developer, [Chris Granger][chris-granger], seems to have exactly
the right kind of attitude.

Another difference between emacs and LightTable, and one that could
also have a played a role in my selection of emacs, was the fact that
emacs feels very unpolished, with lots of wires visible all around,
making it look like exactly the right kind of tool for hacking around
with new languages.

## emacs is fun

emacs is fun because emacs is actually a bunch of lisp code that you
can really debug. If your emacs init file is not loading correctly,
you can actually debug it. While working in the editor, you can
execute commands and change the behavior of the editor instantly. You
can change key-bindings and you can do whatever crazy stuff that comes
to your mind. It's like [LEGO][lego] for hackers.

Even when using IDEs, people try to customize every last bit of
functionality through GUI. Hackers love to have their environment
configured to the very last bit.  Emacs takes this configurability to
a whole new dimension. You write little pieces of code to configure
it, and every single piece of wiring can be taken apart and
reconnected somewhere else. Of course, you sometimes risk breaking
something, but that's a huge part of the fun. Being able to do so is
rewarding in the same sense that it is rewarding to use linux when
using windows could make several things much easier.


## emacs as your first editor

I read somewhere that beginner users shouldn't try to use "difficult"
editors like emacs when they are just starting out. This might have
some truth in it. Nowadays, a newbie programmer is generally learning
at least 4-5 different things at the same time. Entering any
programming language eco-system can be intimidating at first,
especially if you know how much there is to learn. _Learning_ to use
an editor on top of that could just add more cognitive load.

However, I think that using emcas instead of some IDE at the very
beginning of your career can be extremely rewarding. To reduce the
learning burden, they can use it like a plain vanilla text editor with
some syntax highlighting. Once they get comfortable, they can start
tinkering around with all the wiring. By the time they become
professional hackers, they would be pretty good at it. Being
comfortable with a lisp and its programming paradigm since the
beginning of your career can be invaluable. Being proficient with
emacs automatically translates into having at least a decent
development environment for almost any language you choose. And that
is a benefit worth having because learning new languages is essential
to becoming a good hacker. After spending just a week with emacs[^2],
I am convinced that I would have been really grateful if somebody had
given me this advice when I started learning programming.


IDEs are here to stay, no doubt about that. But good hackers are
always learning new things, especially programming languages, and if
you are one of them, it will pay to learn to use at least one great
text editor really well. I am not saying that go with emacs. You can
choose vim. I have never seen an expert `vim` or `emacs` user working
but I have heard that they can do incredible things. I cannot say much
about `vim`, even though it has been a constant companion. But if you
do learn `emacs`, it would definitely be time well spent [^3].

## Resources

- My own emacs config resides on
  [this github repo][emacs-github]. It's fairly well documented. It's
  a good idea to keep your emacs config in a repo, since it's piece of
  code, and you'll regret it if you lose it [^4]
- [Prelude][prelude] is an emacs *distribution* that makes it easy to
  get started with emacs. I am not using it, but exploring their
  config helped me setup some good defaults. Plus, my required-package
  installation code was inspired by them.
- [Emacs Starter Kit][emacs-starter-kit] is an emacs plugin that
  provides better defaults than the OOTB emacs. Again, I am not using
  it, but exploring their code is a good learning experience.
- [Emacs Live][emacs-live]: The most recent addition to the list of
  curated emacs configs. At this point, it is still going through a
  beta phase. However, the curators are long time emacs users and
  contributors. I am inclined to think that this will end up being the
  config that provides the best starter environment. It is especially
  targeted at clojure development.
- [Spacemacs][spacemacs]: This is another addition to the
  pre-configured emacs distros that make life easier for people who
  want to get started with emacs. If you are a [vi][vim] user, this
  could be your preferred choice since it seems to have gone farthest
  in terms of making it compatible with [vi][vim] key bindings. Even
  if you are not a vi user, this distro has amazing look and
  feel. Polished and with lots of goodies to make emacs experience
  even better [^5].
- [Emacs Documentation][emacs-docs] is amazaingly complete.
- [The Little Schemer][little-schemer] is a good book for basic
  introduction to the Scheme programming language (which is a dialect
  of lisp).
- [ergoemacs.org][ergoemacs] has really good introductory material on
  emacs, as well as several advanced tutorials.
- [masteringemacs.org][matering-emacs] is another resource I found
  very useful.

And, Of course, Google.

## Footnotes

[^1]: I know, eclipse, and almost all the other development tools are
    free, and nobody has a right to complain about the free things
    they get. I am not trying to be ungrateful to the developers of
    these great tools at all. I am extremely grateful, actually. But,
    even with free tools, I think people can still get frustrated.

[^2]: Although, the week I spent with emacs was not a normal 9-to-5, 5
    days a week kind of week. It was a hacker week, in which I spent
    almost all my waking hours digging on the internet and
    experimenting.

[^3]: Of course, you will have some WTF moments when you start using
    it. Like: WTF, who was the ass-hole who created these copy-paste
    short-cuts! Don't give up. I think this is the primary reason
    there aren't more emacs users out there. I gave up on emacs twice
    in the past for that reason. This time, I marched on and the WTF
    phase didn't last longer than two days.

[^4]: There is a [famous story][oreilly-vim] about Tim O'Reilly
    switching to vim when he lost his emacs config file. The situation
    is not that bad now. Out of the box, Emacs is still pretty
    usable. It now has a GUI for configuration and package
    installation as well.

[^5]: I am not using any of the curated emacs configs or the
    customized distros since I want to stay close to the metal, and I
    am enjoying the learning process. That doesn't mean that they are
    not tempting.
    

[emacs-github]: [^3]: https://github.com/Bhashit/emacs-config "Bhashit's emacs config github repo"

[prelude]: https://github.com/bbatsov/prelude

[emacs-starter-kit]: https://github.com/technomancy/emacs-starter-kit

[emacs-docs]: http://www.gnu.org/software/emacs/#Manuals

[little-schemer]: http://www.amazon.com/Little-Schemer-Daniel-P-Friedman/dp/0262560992/ref=sr_1_1?s=books&ie=UTF8&qid=1407818275&sr=1-1&keywords=the+little+schemer

[ergoemacs]: http://ergoemacs.org/emacs/emacs.html

[matering-emacs]: http://www.masteringemacs.org/

[lego]: www.lego.com/

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

[oreilly-vim]: http://www.oreillynet.com/pub/a/oreilly/ask_tim/1999/unix_editor.html

[emacs-live]: https://github.com/overtone/emacs-live

[spacemacs]: https://github.com/syl20bnr/spacemacs

[org-mode]: http://orgmode.org/

[twitter-mode]: https://github.com/hayamiz/twittering-mode
