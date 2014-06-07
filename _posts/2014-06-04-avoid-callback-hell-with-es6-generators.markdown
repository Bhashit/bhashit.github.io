---
layout: post
title:  Breaking out of the callbak hell with ECMAScript 6 generators.
date:   2014-06-04
categories: programming JavaScript
comments: true
---

**Note**: This blog post is inspired by the following two excellent videos. I think it will be more beneficial for you to watch those.

1. [Promises and Generators: control flow utopia -- JSConf EU 2013][forbes-video]
2. [Hanging Up On Callbacks: Using ECMAScript 6 Generators][hangup-callbacks-video]

I have made some changes to deal with the general use-case of multiple [`jQuery`][jQuery] based ajax calls, but for the most part, this blog post is just a learning exercise for me.

I like reading, and learning, a lot. I also recently quit my day job, where I was working overtime (by choice), doing not just my own work, but also helping almost everyone else out with their programming problems. This was fun as long as people came to me with hard problems that they couldn't solve; at least, not in a reasonable amount of time. I enjoyed that part so much that I almost never complained about being underpaid. As long as I could get my hands on some cool problems, money was not an issue. The bad part was that, since I was so accomodating, people started coming to me with problems that were just plain silly. That was too... hmmm... ah, aggravating. This started happening way too frequently; and that was *the last straw*. Even though I was working overtime by *choice*, I did miss a few things like long learning/programming sessions and leisure reading. Don't get me wrong. I learned a lot on the job. I did learn a lot by solving other people's problems. Sometimes those problems took up my weekends. But I was getting more and more busy. Having 2 days straight just for learning/programming was a luxury. I like learning things in detail. I like getting to know the internals of things. I had been thinking of quitting the job for quite some time, and long story short, I  quit the job. So, you might be wondering what does this have to do with the `ECMAScript` generators. Well, nothing really. The thing is, after quitting the job, I have been doing some freelance work, which pays way more than my job and I still get more time to learn new things. So, I have been bingeing on all kinds of learning resources: books, articles, videos. That's how I got to [`ES 6`][es6]. I have been working with JavaScript for more than 5 years, from the days when I had to worry about `IE 6` compatibility. It has come a long way from those days.

`ES 6` is going to come out with a [lot of features][es6-features]. A lot of those features actually look quite promising. Kind of coming of age for JavaScript. Of all those features, I am going to talk about [`generators`][generators]. The other features are going to make our lives easier, and I am looking forward to having them, but they are not going to change the way we fundamentally program in JavaScript.

`ECMAScript 6` is still not very widely implemented yet. If you want to execute the code, the latest versions of Firefox or Chrome would be obvious choices. If you want to checkout the implementation status of `ES 6` for various browsers, check out this [compatibility index][es6-compatibility].

## The control flow in asynchronous coding
In jQuery, now that we have [`deferred objects`][jQueryDeferred] and the so called [`promises`][jQueryPromise], this is how I could nest my ajax calls:

```javascript
$.ajax("/request1").then(function(data1) {
  console.log("Request 1 done");
	$.ajax("/request2").then(function(data2) {
	  console.log("Request 2 done");
  }, function() {
    // handle request 2 failure
  });
}, function() {
  // handle request 1 failure
});

```

Okay, this is not yet very ugly. However, add one more nested ajax call, and you enter the proverbial [callback hell][callbackHell]. Instead of nesting, if you want to execute those ajax calls in parallel, you could do something like this:

```javascript
$.when($.ajax("/request1"), $.ajax("/request2")).then(function(response1, response2) {
  // Each argument is an array with the following structure: [data, statusText, jqXHR]
  // The arguments are the responses for each request passed to $.when

}, function(xhr) {
  // One of the requests failed, use the xhr object and do the necessaey cleanup.
})

```

Again, this is not very hellish; not pretty either. Things are made a bit easy by [`jQuery.when`][jQueryWhen]. However, this is still not very pretty. It's far from the normal flow of code that programmers are generally used to. We don't have `try-catch` and we handle the errors through callbacks. We might need one error handling callback for each asynchronous operation. You might still need additional error handling when dealing with the results of a successful asynchronous operations. For ex. trying to parse the response of a request could throw an exception and you might want to handle that. So, your error handling code would be divided into two parts: the error callbacks and the `try-catch` within the success callbacks.

## Generators to the rescue




[es6]: http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts
[es6-features]: http://espadrine.github.io/New-In-A-Spec/es6/
[generators]: http://wiki.ecmascript.org/doku.php?id=harmony:generators
[es6-compatibility]: http://kangax.github.io/compat-table/es6/
[jQuery]:http://jquery.com/
[forbes-video]: https://www.youtube.com/watch?v=qbKWsbJ76-s
[hangup-callbacks-video]: https://www.youtube.com/watch?v=OYdP1tQ9Rnw
[jQueryWhen]: http://api.jquery.com/jQuery.when/
[jQueryDeferred]: http://api.jquery.com/category/deferred-object/
[jQueryPromise]: http://api.jquery.com/promise/
[callbackHell]: http://callbackhell.com/
