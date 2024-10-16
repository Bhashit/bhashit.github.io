---
layout: post
title: Futures and async rendering with liftweb
date: 2016-07-17
comments: true
---

Although liftweb is quite good at some things, one of the things it
doesn't do well is dealing with scala/akka Futures in your HTML
rendering code (snippets). If your snippet needs to handle some
Futures before rendering the page, you'd be hard-pressed to find a
_clean_ looking _generic_ solution.

Lift provides something called [lazy-load][lazy-load] that can, not
surprisingly, lazily render something on a different thread. It takes
a normal snippet method, and executes it on a different thread using
an actor, and renders it using a comet when it is finished
executing. However, it doesn't handle a `Future[CssSel]`. So, if you
need to deal with a `Future` in a lazily loaded snippet, you'd still
need to await its completion by blocking. Which is not something I'd
like to do.

Then, there is [this solution][async-render] by one of the most
respected people in Lift community. Here, the initial rendering of
your snippet puts some JS on the client-side, which then repeatedly
polls the server for the result. This example uses lift's own
LAFutures, but it can be easily adapted to use scala Futures.
Although, this is a pretty good solution, I would have liked something
along the lines of LazyLoad snippet without having to do explicit
polling. Especially because lift has great comet support, and that
could be used to render the results when they become available.

So, I put together a [little solution][repo] that uses the idea behind
LazyLoad and allows me to use Futures as easily, and as cleanly, as
the "normal" snippet methods.

The code is built on top of one of the stock sample templates that
come with [lift 2.6 downloads](http://liftweb.net/download). I have
added some comments that should hopefully make the code
self-explanatory. Following is a gist of the basic ideas in this
"solution".

Let's say you have a method named `getDetails:
Future[List[(String, String)]]` that returns some data from a
web-service call, and you need to render it on the page. So, here's
what you might have:

```scala

class HelloWorld {
  
  def render(template: NodeSeq): NodeSeq = {
    AsyncRenderer.render(getAndRenderDetails, template)
  }
  
  
  private def getAndRenderDetails: Future[CssSel] = {
    getDetails.map { renderDetails }
  }
  
  private def getDetails: Future[List[(String, String)]] = ...
}

```

This is taken from the [sample code repo][repo], but with some details
stripped away. The `getAndRenderDetails` returns  a `Future[CssSel]`,
instead of a normal `CssSel` that a lift snippet method is expected to
have. In order to make this work, you provide an extra layer above in
the `render` method. Your HTML will refer to this `render` method.

```html

<div data-lift="helloWorld.render"></div>

```

The `render` method, via the
[`AsyncRenderer`](https://github.com/Bhashit/liftweb-async-snippet-sample/blob/master/src/main/scala/code/lib/AsyncRenderer.scala),
returns a placeholder `NodeSeq` to be displayed on the page while the
Future is pending. The placeholder could be anything you want. In my
sample, it's a spinner image, which goes away when the Future
completes and we can render the data.

When the Future completes, the AsyncRenderer does the following: 

1. Apply the resulting CssSel to the original template, resulting in a
   NodeSeq
2. Create a `JsCmd`  that will set that NodeSeq on the page in its
   intended  place
3. This `JsCmd` is then sent to the client using a comet, which
   results in our data being displayed.

Where did that comet come from? I'm using a basic comet class called
`CommandComet` that can be used for sending JsCmds to the page. If you
already have setup an instance of CommandComet on your page before
your async snippet (`HelloWorld.render`) was called, you can pass its
name to `AsyncRenderer#render`. If not, an instance will be setup
automatically.

The most important bit here is the creation of
[a deferred function](https://github.com/Bhashit/liftweb-async-snippet-sample/blob/master/src/main/scala/code/lib/AsyncRenderer.scala#L47)
that will apply your `Future[CssSel]` to the template. The
`LiftSession.buildDeferredFunction` allows you to execute arbitrary
code inside the context of a request, even when you are not in the
thread that created that request. This basically allows you to access
any RequestVars etc. that you might be needed for rendering. Without
using `buildDeferredFunction`, anything that relied on any kind of
request context would fail. 

The sample code also contains a working example of how
you can handle failed Futures. Also, it's supposed to be a minimal
sample. You can modify/add arguments to the `AsyncRenderer#render`
method to suit your needs and provide more flexibility. For ex. you
can allow the users to pass in an optional `NodeSeq` that can be
displayed while the Future is pending.

Well, this is it. Hope it helps.

[lazy-load]: https://github.com/lift/framework/blob/3.0-RC3-release/web/webkit/src/main/scala/net/liftweb/builtin/snippet/LazyLoad.scala
[async-render]: http://blog.fmpwizard.com/blog/async-snippets-in-lift
[repo]: https://github.com/Bhashit/liftweb-async-snippet-sample
