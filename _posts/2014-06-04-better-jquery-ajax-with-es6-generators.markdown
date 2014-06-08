---
layout: post
title:  Eliminating ugliness in jQuery ajax with ECMAScript 6 generators.
date:   2014-06-04
comments: true
---

**Note**: This blog post is inspired by the following two excellent videos. I think it will be more beneficial for you to watch those.

1. [Promises and Generators: control flow utopia -- JSConf EU 2013][forbes-video]
2. [Hanging Up On Callbacks: Using ECMAScript 6 Generators][hangup-callbacks-video]

I have made some changes to deal with the general use-case of multiple [`jQuery`][jQuery] based ajax calls, but for the most part, this blog post is just a learning exercise for me.

I like reading, and learning, a lot. I also recently quit my day job, where I was working overtime (by choice), doing not just my own work, but also helping almost everyone else out with their programming problems. This was fun as long as people came to me with hard problems that they couldn't solve; at least, not in a reasonable amount of time. I enjoyed that part so much that I almost never complained about being underpaid. As long as I could get my hands on some cool problems, money was not an issue.

The bad part was that, since I was so accomodating, people started coming to me with problems that were just plain silly. That was too... hmmm... ah, aggravating. This started happening way too frequently; and that was *the last straw*. Even though I was working overtime by *choice*, I did miss a few things like long learning/programming sessions and leisure reading. Don't get me wrong. I learned a lot on the job. I did learn a lot by solving other people's problems. Sometimes those problems took up my weekends. But I was getting more and more busy. Having 2 days straight just for learning/programming was a luxury. I like learning things in detail. I like getting to know the internals of things. I had been thinking of quitting the job for quite some time, and long story short, I  quit the job.

So, you might be wondering what does this have to do with the `ECMAScript` generators. Well, nothing really. The thing is, after quitting the job, I have been doing some freelance work, which pays way more than my job and I still get more time to learn new things. So, I have been bingeing on all kinds of learning resources: books, articles, videos. That's how I got to [`ES 6`][es6]. I have been working with JavaScript for more than 5 years, from the days when I had to worry about `IE 6` compatibility. It has come a long way from those days.

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
  // Each argument is an array with the following structure:
  // [data, statusText, jqXHR]
  // The arguments are the responses for each request passed to $.when

}, function(xhr) {
  // One of the requests failed, use the xhr object and do the
  // necessary cleanup.
})

```

Again, this is not very hellish; not pretty either. Things are made a bit easy by [`jQuery.when`][jQueryWhen]. However, this is still not very pretty. It's far from the normal flow of code that programmers are generally used to. We don't have `try-catch` and we handle the errors through callbacks. We might need one error handling callback for each asynchronous operation. You might still need additional error handling when dealing with the results of a successful asynchronous operations. For ex. trying to parse the response of a request could throw an exception and you might want to handle that. So, your error handling code would be divided into two parts: the error callbacks and the `try-catch` within the success callbacks.

## Generators to the rescue

To some people, generators might seem like magical beings, especially all you know are few mainstream languages like Java or JavaScript. To *paraphrase* the [MDN][MDNGenerators] documentation

> They have the capability to suspend their execution context and then later re-enter that execution context with variable bindings intact across the re-entrances.

So, if you don't know the basics of generators already, here's a quick rundown of the basics:

```javascript
function *rangeGenerator(lower, upper) {
  var result= lower;
  while(result <= upper) {
    yield result;
    result++;
  }
}

var range = rangeGenerator(1, 3);

console.log(range.next());
// => {value: 1, done: false}
console.log(range.next());
// => {value: 2, done: false}
console.log(range.next());
// => {value: 3, done: false}
console.log(range.next());
// => {value: undefined, done: true}

for(let value of range) {
  console.log(value);
}

// => {value: 1, done: false}
// => {value: 2, done: false}
// => {value: 3, done: false}
// => {value: undefined, done: true}


```

Notice the `*` just before the function name. That's what makes this a generator function. When you call `rangeGenerator()`, it returns you the actual generator object that you can use to get to the generated values. When a `yield` statement is encountered, the result of evaluating the expression on the right-hand side of `yield` is returned and the execution context is suspended. Each call to `range.next()` returns a  JS object with two properties: `value` and `done`. The `value` is the actual value that is returned by the `yield` statement, and `done` is a boolean flag indicating whether the generator is done producing results. You can easily use this flag to get all the values from a generator using some form of a loop. Or, you can directly iterate over them using the new `for...of` loop, as can be seen in the example. You can do all kinds of cool stuff with generators. For ex. you can use them to create infinite sequences.

```javascript
function *getInfiniteSeq() {
  var i = 0 ;
  while(true) {
    yield i;
    i++;
  }
}

function take(infiniteSeq, count) {
  let result = [];
  for(let i = 0; i < count; i++) {
    result.push(infiniteSeq.next().value);
  }
  return result;
}


var inifiniteSeq = getInfiniteSeq();
console.log(take(inifiniteSeq, 7));
// => [0, 1, 2, 3, 4, 5, 6]
console.log(take(inifiniteSeq, 5));
// => [7, 8, 9, 10, 11]

```

You can go even a step further and define operations like `map` to be lazy. Alright, so, as far as using generators with async operations is concerned, there are two important things to keep in mind about generator.

1. You can inject values into a generator by passing an argument to the `next()` method call. If you do so, the value passed to the method call would be the return value of the last `yield` statement that resulted in a value.
2. You can call `throw()` method on a generator. The exception object passed to the `throw` will be thrown from the current suspended context of the generator, as if the yield statement that is currently suspended were a `throw` statement.

The following sample should probably explain it a bit more clearly.

```javascript
function *generatorFunc() {
  var injectedVal = yield 1;
  console.log(injectedVal == 0);
  return 10;
}

var generator = generatorFunc();
console.log(generator.next());
// => {value: 1, done: false}
generator.next(0);
// => true
console.log(generator.next());
// => {value: undefined, done: true}
generator.throw("Some Error");
// "Some Error"

```

Keeping these two things in mind, a generic utility method could be created that would allow us to handle the jQuery ajax code in a much more natural, and easy to manage manner. We would make use of the `async` function defined below. This function is inspired by the contents of the two videos mentioned at the beginning of this post. Of course, I have made a few changes, and corrected a mistake.

```javascript
function async(genertorFactory) {
  var generator = genertorFactory.apply(this, arguments);
  var handleResult = function(result) {
    if(result.done) return result.value;
      // In our example, the result.value would be a jqXHR object, which has a
      // then() method that is similar in its contract to the Promise objects
      // specified in A+ promises (for ex. https://www.promisejs.org/)
      return result.value.then(function(nextResult) {
        // Push the result back to the generator. This will be the
        // return value of the corresponding yield operation.
        return handleResult(generator.next(nextResult));
    }, function(error) {
      // Propagate the error back to the generator. This exception will be
      // thrown from the current suspended context of the generator, as if
      // the yield statement that is currently suspended were a `throw`
      // statement.
      generator.throw(error);
    })
  };
  return handleResult(generator.next());
}
```

So, how do we use this. Well, pretty simple. If you want to have multiple async calls executed one by one:

```javascript
async(function *() {
  try {
    // With jQuery, the XHR objects returned by the $.ajax method calls are
    // somewhat like a Promise object (they have the then() method, which is all
    // we require for this code. If you want a full Promise implementation with
    // jQuery, you can call promise() on any jQuery object.)
    var result1 = yield $.ajax("/request1");
    var result2 = yield $.ajax("/request2");
    var result3 = yield $.ajax("/request3");
    // Do something with the results
    console.log(result1, result2, result3);
  } catch(xhr) {
    console.log("Error: " + xhr);
  }
});
```

This is much more readable then any callback based mechanism we might come up with. It also has the advantage that the errors can be handled in a more natural way. No multiple `try-catch` blocks, no error-callbacks. Just a straightforward `try-catch`.

If we want to execute these calls in parallel, we can do that as well.

```javascript
async(function *() {
  try {
    var resultPromise1 = $.ajax("/request1");
    var resultPromise2 = $.ajax("/request2");
    var resultPromise3 = $.ajax("/request3");
    // Do something with the results
    let results = {"1": yield resultPromise1, "2": yield resultPromise2, "3": yield resultPromise3}
    console.log(results);
  } catch(xhr) {
    console.log("Error: " + xhr);
  }
});
```

Notice that all the ajax calls start in parallel. Notice especially the position of the `yield` keyword.

Generators do look like a great feature. As more developers become faimiliar with it, I think they will come up with even more usages for them.

## resources

- [ECMAScript spec][es6]. Still in the draft stage.
- [ECMAScript Generators Spec][generators]
- [MDN documentation for generators][MDNGenerators]
- [`jQuery.when`][jQueryWhen]
- [jQuery promises][jQueryPromise]
- [Summary of new features in ES 6][es6-features]
- [Summary of ES6 compatibility in various browsers][es6-compatibility]

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
[MDNGenerators]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*
