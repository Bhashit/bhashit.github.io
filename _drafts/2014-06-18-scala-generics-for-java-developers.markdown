---
layout: post
title:  Generics in Scala- covariance and contravariance for dummies (AKA Java developers)
date:   2014-06-18
comments: true
---

Were you offended the I called java developers dummies? Have you been following [twitter][twitter] lately? Or, any place where the geeks hang out? Java is still the leading enterprise language, but it is no longer considered cool to call yourself a Java developer (was it ever?). I got "enlightened" by functional programming pretty late in my career. By that time, I had already begun working with Scala and Clojure on a regular basis. And then Java 8 was introduced, which took a *lot* of the pain away. For most of my career, [Spring][spring] made life a lot easier. That being said, I no longer prefer to work in Java if I have an option to work with, say, Scala.

Scala itself is not a very pretty language. I got to Scala after I learned Clojure and [Scheme][scheme]. So, Scala seems a like a bit of a misfit. Scala is to Java what C++ was to C. C++ allowed C programmers to keep their old way of working while giving them an option to use object orientation. Scala allows Java programmers to keep coding the way they are used to, while still allowing them to use the functional features whenever they want. It's a way of going from object oriented to functional. It's an effective strategy to get the developer mindshare. It's very easy for Java developers to pick the basics of Scala. They can keep using their older libraries. And once they taste the syntactic sugar of Scala, they probably won't go back to Java. Scala has almost seamless integration with Java. Unlike Groovy, Scala has static typing without the verbosity. Java developers should feel right at home. So, like Bjarne Stroustrup, Martin Odersky has picked the right strategy and it seems like it's working. Scala is poised to become, or has already become, the next big language on the JVM. But if this analogy to C++ is correct, I suspect that a pure functional language will replace Scala in the years to come; like Java for C++. And right now, it seems like [Haskell][haskell] could be that language. Maybe I'm wrong. We'll see.

For average Java developers, it wasn't easy to use Generics effectively. Most of the people only used them with Collections and that was that. I still know several people who would be dumbfounded if they ever had to write something involving bounded types. Well, this post is definitely not for them.

After learning Scala for more than a year, I still cannot say that I am an expert. However, I'm writing more and more idiomatic Scala code, and I have even learned when to stay away from some [ugly parts][scala-implicits] of Scala. For now, I'm going to focus on Scala's generics. Scala has much more powerful Generics than Java. One thing that is not quite easy to put into words is the concept of cavariance and contravariance. That's what I'm going to attempt to do here.

## Covariance

The textbook definition goes something like this:

> If a generic type's subtype relationship is the same as its type parameter T, that generic type is covariant.

I know, I did't get it the first time either. Here's an example that could make this a bit more clear. In Java, you might be used to something like this:

```java
void foo(Iterator<? extends Number> numberIterator) { ... }
```

Here, we are saying that we can have an `Iterator` for a `Number` or any subclass thereof. We are declaring `Iterator` to be covariant with respect the class hierarchy of `Number`. The argument `numberIterator` is assignment compatible with an `Iterator` of any subclass of `Number`. That is, you can pass it an `Iterator<Integer>` or an `Iterator<Double>` and it would work. Since `Integer` is a subclass of `Number`, `Iterator<Integer>` is subclass of `Iterator<Number>`. In some other place, we might have a declaration like this:

```java
 public <T extends Object> void doIt(Iterator<T> numberIterator) { ... }
```

Here, we are declaring that `Iterator` is covariant with respect to the entire `Object` hierarchy. It can essentially iterate over any `Object`. What we are saying is that

> If a type B is a subtype of A, for `Iterator` to be considered covariant, `Iterator<B>` should be assignable to `Iterator<A>`. In other words, `Iterator<B>`  can be treated as a subclass of `Iterator<A>`

I know, some academics might be ready to kill me now :tired_face:. Reiterating that academic definition now should be fruitful. The generic type `Iterator`'s subtype relationship is in the same direction as its type parameters. That is, since `Integer` is a subtype of `Number`, `Iterator<Integer>` is a subtype of `Integer<Number>`. The types are varying in the same direction.

### Covariance :point_right: producers

Covariance is generally associated with constructs that produce or return values. The `Iterator[_ <: Number]` is a producer for numbers. If you think with that perspective, covariance makes even more sense. An `Iterator[Integer]` is going to produce `Integers`, and `Integers` are `Numbers`, so a producer of `Integer` is also a producer of `Numbers`. The covariance was also applied to method return types in java 5.

```java

class Producer {
  Object getSomething(){ ... }
}

class IntegerProducer extends Producer {
  @override
  Integer getSomething() { ... }
}
```

Since the method `getSomething` is supposed to return (produce) an `Object`, it should be okay to return an `Integer` from the overridden method. Since `Integer` *is* an `Object`, a method producing `Integer` is producing an `Object`. So, to reiterate:
> covariance is associated with constructs that produce or return values.


## contravariance

Contravariance is much less intuitive than covariance. To again begin with a textbook-like definition:

> If a generic type's subtype relationship is the reverse direction as that of its type parameter T, that generic type is contravariant.



[twitter]: https://twitter.com/
[spring]: http://spring.io/
[scheme]: https://en.wikipedia.org/wiki/Scheme_%28programming_language%29
[haskell]: http://www.haskell.org/haskellwiki/Haskell
[scala-implicits]: http://docs.scala-lang.org/overviews/core/implicit-classes.html
