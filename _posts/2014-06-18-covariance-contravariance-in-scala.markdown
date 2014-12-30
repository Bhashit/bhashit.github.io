---
layout: post
title:  Declaration site covariance and contravariance in Scala for dummies
date:   2014-06-18
comments: true
---

Scala is not a very pretty language. I got to Scala after I learned Clojure and [Scheme][scheme]. So, Scala seems a like a bit of a misfit. Scala is probably to Java what C++ was to C. C++ allowed C programmers to keep their old way of working while giving them an option to use object orientation. Scala allows Java programmers to keep coding the way they are used to, while still allowing them to use the functional features whenever they want. It's a way of going from object oriented to functional. It's an effective strategy to get the developer mindshare. It's very easy for Java developers to pick the basics of Scala. They can keep using their older libraries. And once they taste the syntactic sugar of Scala, they probably won't go back to Java. Scala has almost seamless integration with Java. Unlike Groovy, Scala has static typing without the verbosity. Java developers should feel right at home. So, like Bjarne Stroustrup, Martin Odersky has picked the right strategy and it seems like it's working. Scala is poised to become, or has already become, the next big language on the JVM. But if this analogy to C++ is correct, I suspect that a pure functional language will replace Scala in the years to come; like Java for C++. And right now, it seems like [Haskell][haskell] could be that language. Maybe I'm wrong. We'll see.

For average Java developers (that I know of), it wasn't easy to use Generics effectively. Most of the people only used them with Collections and that was that. I still know several people who would be dumbfounded if they ever had to write something involving bounded types. This post is definitely not for them.

After learning Scala for more than a year, I still cannot say that I am an expert. However, I'm writing more and more idiomatic Scala code, and I have even learned how to handle some [possibly dangerous parts][scala-implicits] of Scala. For now, I'm going to focus on Scala's generics. Scala has much more powerful Generics than Java. One thing that is not quite easy to put into words is the concept of covariance and contravariance. That's what I'm going to attempt to do here.

## Covariance

The textbook-like definition goes something like this:

> If a generic type's subtype relationship is the same as its type parameter T, that generic type is covariant.

I know, I did't get it the first time either. Here's an example that could make this a bit more clear. In Java, you might be used to something like this:

{% highlight java %}

void foo(Iterator<? extends Number> numberIterator) { ... }

{% endhighlight %}

Here, we are saying that we can have an `Iterator` for a `Number` or any subclass thereof. We are declaring `Iterator` to be covariant with respect the class hierarchy of `Number`. The argument `numberIterator` is assignment compatible with an `Iterator` of any subclass of `Number`. That is, you can pass it an `Iterator<Integer>` or an `Iterator<Double>` and it would work. Since `Integer` is a subclass of `Number`, `Iterator<Integer>` is treated as a subtype of `Iterator<Number>`. In some other place, we might have a declaration like this:

{% highlight java %}

public <T extends Object> void doIt(Iterator<T> numberIterator) { ... }

{% endhighlight %}

Here, we are declaring that `Iterator` is covariant with respect to the entire `Object` hierarchy. It can essentially iterate over any `Object`. What we are saying is that

> If a type B is a subtype of A, for `Iterator` to be considered covariant, `Iterator<B>` should be assignable to `Iterator<A>`. In other words, `Iterator<B>`  can be treated as a subclass of `Iterator<A>`

I know, some academics might be ready to kill me now :tired_face:. Reiterating that textbook-like definition now should be fruitful. The generic type `Iterator`'s subtype relationship is in the same direction as its type parameters. That is, since `Integer` is a subtype of `Number`, `Iterator<Integer>` is a subtype of `Integer<Number>`. The types are varying in the same direction.

### Covariance :point_right: producers

Covariance is generally associated with constructs that produce or return values. The `Iterator[_ <: Number]` is a producer for numbers. If you think with that perspective, covariance makes even more sense. An `Iterator[Integer]` is going to produce `Integers`, and `Integers` are `Numbers`, so a producer of `Integer` is also a producer of `Numbers`. The covariance was also applied to method return types in java 5.

{% highlight java %}

class Producer {
  Object getSomething(){ ... }
}

class IntegerProducer extends Producer {
  @override
  Integer getSomething() { ... }
}

{% endhighlight %}

Since the method `getSomething` is supposed to return (produce) an `Object`, it should be okay to return an `Integer` from the overridden method. Since `Integer` *is* an `Object`, a method producing `Integer` is producing an `Object`. So, to reiterate:
> covariance is associated with constructs that produce or return values.


## contravariance

Contravariance is much less intuitive than covariance. To again begin with a textbook-like definition again:

> If a generic type's subtype relationship is the reverse direction as that of its type parameter T, that generic type is contravariant.

I'll go with a pretty contrived example here to illustrate my point. Suppose you have the following kind of a method

{% highlight java %}
void printSomeInteger(Function<Integer, String> formattedStringProducer) {
  Integer value = getSomeInteger();
  String formattedString = formattedStringProducer.apply(value);
  System.out.println(formattedString);
}

Function<Number, String> numberFormatterFunction = ...
Function<Integer, String> integerFormatterFunction = ...

{% endhighlight %}

This method `printSomeInteger` expects a [`Function`][java8_function] that can take an Integer and produce a formatted string. Now, with this definition, I will be able to pass `integerFormatterFunction` to `printSomeInteger`, but not `numberFormatterFunction`. That's where I can use contravariance. If I modify the signature of `printSomeInteger` like this:

{% highlight java %}
void printSomeInteger(Function<? super Integer, String> formattedStringProducer)
{% endhighlight %}

I will be able to pass both `numberFormatterFunction` and `integerFormatterFunction` to it. The argument `formattedStringProducer` is now declared to be contravariant. What it is actually saying is "any Function that can take an Integer or some supertype thereof and produce a String". The argument is a [`Function`][java8_function] that can consume an Integer. That is the essence of it.

### Contravariance :point_right: consumers

Contravariance is generally associated with constructs that consume values, or take them as an argument. Since we want a Function that can consumer an Integer, it should generally be okay to pass a Function that can consume some supertype of Integer. A Function that can consume (that is, take as an argument), a Number, can consumer an Integer (that is, take an Integer as an argument) because all Integers are Numbers. So, with contravariance, a `Function<Number, T>` is assignment compatible to a `Function<Integer, T>`. The second type parameter is not important at the moment, so ignore it for now. This leads us to

> If a type B is a subtype of A, for `Function` to be considered contravariant, `Function<A, SomeType>` should be assignable to `Function<B, SomeType>`. In other words, `Function<A, SomeType>`  can be treated as a subclass of `Function<B, SomeType>`

This stands true for all consumers, not just `Function`. For ex. the signature of [`Collections.addAll()`][collections_addAll] stands like this

{% highlight java %}
public static <T> boolean addAll(Collection<? super T> collectionToFill,
        T... elements)

{% endhighlight %}

Here, the method is supposed to add the elements of type T to the Collection. The `collectionToFill` is a consumer here since it's going to consume the items in the `elements` array. Here, we can pass a `Collection<Number>` as the first argument and a few Integers as the trailing vararg list and it will work, even though the compiler knows through type inference that the the type parameter `T`,in this case, is evaluated to Integer. That is because, in this case, the Collection argument is *declared* to be contravariant. So, a Collection<Number> is assigment compatible to a `Collection<Integer>`. To reiterate on the definition, with contravariance, the Collection<Number> can be treated as a sub-type of Collection<Integer> *because* Integer is a subtype of Number. The type-subtype relationship is inverted. This makes sense when you think about it in terms of consumers. A Consumer that can take Numbers, should be able take Integers, precisely because an Integer *is* a Number. So, any place that expects a consumer of Integers, should be able to work with a consumer of Numbers, because a consumer of Numbers is a consumer is Integers.


With this background, I'll finally come to the point.

## Use site variance and declaration site variance

In Java, the covariance and contravariance are specified on the use site. That is, none of the Generic types are variant by definition. You can never assign `List<Integer>` to a `List<Number>`. Whenever you need variance, you need to specify it explicitly, wherever you *use* it. That's why it's called user-site variance. And that makes sense with mutable collections. For ex. Arrays in Java are covariant by definition, and that has been a problem. See this example:

{% highlight java %}
Number[] nums = new Integer[10];
nums[0] = new Double(10);
{% endhighlight %}

The array assigned to `nums` is an arrays of Integers. Trying to put a double in it will throw an `ArrayStoreException` at runtime. The error won't be caught at compile time. That's why the collections are not covariant by definition. If they were, we would be facing the same problem. When covariance is achieved by using wildcards with the `extends` keyword, this problem doesn't occur since the compiler won't let you put anything into a collections whose exact type parameter is not known or inferred. For ex.

{% highlight java %}
List<? extends Number> numbers = new ArrayList<>();
numbers.add(10);

{% endhighlight %}

would result in a compile time error. However, the problem with in-built covariance only exists for mutable collections.

### Declaration site covariance

In Scala, immutable collections are much more common. And with immutable collections, in-built covariance doesn't cause any problems. It rather eliminates the boilerplate of specifying variance at use site. In Scala, `scala.collection.immutable.List` is declared as

{% highlight scala %}
class List[+A]
{% endhighlight %}

The `+` means that the List class is covariant. So a `List[Integer]` is a subtype of `List[Number]`. The covariance is built into the definition of the class. This is called *declaration site variance*. The type is *declared* with the intended variance. The problem mentioned in the example with arrays is simply absent here since the List is immutable. You cannot modify the list, and hence, cannot store elements of incorrect type. Scala does have some mutable collections and they are invariant, like java collections. For ex. the mutable version of `Seq` is declared as

{% highlight scala %}
package scala.collection.mutable

trait Seq[A] ...

{% endhighlight %}

And the immutable counterpart of this trait is declared as

{% highlight scala %}
package scala.collection.mutable

trait Seq[+A]
{% endhighlight %}

The declaration-site covariance is available for almost all the types where having it makes a lot of sense. For ex.

{% highlight scala %}
trait Iterator[+A] ...

trait Traversable[+A]

{% endhighlight %}

### Declaration site contravariance

Like producers with covariance, there are some obvious candidates for contravariance. For ex. Scala has a type called [`Function1`][scala_function1] that denotes a function that takes a single argument and produces a value.

{% highlight scala %}
trait Function1[-T1, +R]
{% endhighlight %}

Notice the first type parameter. The `-` says that the type `Function1` is contravariant in its argument type, which is the type that is consumes. To put the signature in words: "It is a function that can take an argument of type T1 or some supertype thereof and produce a value of type R or a subtype of R". So, everywhere a `Function1[Integer, R]` is expected, we can pass a `Function1[Number, R]` since the latter is a subtype of the former. This is such an obvious case for having declaration-site variance.

Java doesn't have the concept of declaration-site variance, and hence, if you are using java, you'll have to specify the variance wherever you use it. Scala does have support for use-site variance as well. However, if you are using immutable collections with declaration-site variance, using use-site variance with those could lead to very confusing code.

## A few quirks

### Sets are not covariant

Even though other collections like `List`, `Seq` and `Vector` are covariant, the Sets are not. This, [according to Martin Odersky][sets_odersky], was a wrong-headed decision. This was done to accomodate the definition of Sets as Functions. For ex. the definition of `Set` looks like this

{% highlight scala %}
trait Set[A] extends (A => Boolean)
{% endhighlight %}

The set-as-function construct is used to determine whether an element exists in the set or not.

{% highlight scala %}
val set = Set(1, 2, 3)
println(set(2)) // prints "true"
{% endhighlight %}

However in that definition, the `(A => Boolean)` part is `Function1`. And `Function1` is contravariant in its argument type, remember? So, if a `Set` is a `Function1`, it is contravariant; and it can't be covariant and contravariant at the same time. We can't declare set to be contravariant as well, since that would create problems with covariant traits like `Iterable` that Set extends. So, there is a conflict with either kind of variance and it was decided to make Sets invariant. Scala people are learning to live with that mistake. See [this link][so_discussion] for some interesting discussion.

### Scala compiler does get confused sometimes

With java, we are pretty much used to seeing this definition:

{% highlight java %}
static <T extends Comparable<? super T>> T max(Collection<? extends T> coll)
{% endhighlight %}

Here, we are expecting a Collection of some type that implements Comparable either directly, or through one of its super-classes. Translating this declaration directly into Scala doesn't work as of now (Scala version 2.11.0-M8)

{% highlight scala %}
def max[T <: Comparable[_ >: T]](p: Iterable[T]) = ???
{% endhighlight %}

The compiler complains about an "illegal cyclic reference involving type T". Well, fortunately, there is a simple workaround for this.

{% highlight scala %}
type ContravariantComparable[T] = Comparable[_ >: T]
def max[T <: ContravariantComparable[T]](p: Iterable[T]) = ???
{% endhighlight %}


## Resources

Eric Lippert has written excellent series of articles on convariance and contravariance. The articles are directed at C#, but should be easily understandable for people who know Java, Scala or any other Java like language.

1. [Covariance and Contravariance in C#, Part One][eric_lipper_1]
2. [Covariance and Contravariance in C#, Part Two: Array Covariance][eric_lipper_2]
3. [Covariance and Contravariance in C#, Part Three: Method Group Conversion Variance][eric_lipper_3]
4. [Covariance and Contravariance in C#, Part Four: Real Delegate Variance][eric_lipper_4]
5. [Covariance and Contravariance In C#, Part Five: Higher Order Functions Hurt My Brain][eric_lipper_5]

Of course, there are several more parts in that series. But these are the most essential. If you are inclined, you will be able to search that blog for the rest of the articles in the series very easily.


[twitter]: https://twitter.com/
[spring]: http://spring.io/
[scheme]: https://en.wikipedia.org/wiki/Scheme_%28programming_language%29
[haskell]: http://www.haskell.org/haskellwiki/Haskell
[scala-implicits]: http://docs.scala-lang.org/overviews/core/implicit-classes.html
[java_util_function]: http://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html
[java8_function]: http://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html
[collections_addAll]: http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#addAll-java.util.Collection-T...-
[scala_function1]: http://www.scala-lang.org/api/current/scala/Function1.html
[sets_odersky]: http://www.scala-lang.org/old/node/9764
[eric_lipper_1]: http://blogs.msdn.com/b/ericlippert/archive/2007/10/16/covariance-and-contravariance-in-c-part-one.aspx
[eric_lipper_2]: http://blogs.msdn.com/b/ericlippert/archive/2007/10/17/covariance-and-contravariance-in-c-part-two-array-covariance.aspx
[eric_lipper_3]: http://blogs.msdn.com/b/ericlippert/archive/2007/10/19/covariance-and-contravariance-in-c-part-three-member-group-conversion-variance.aspx
[eric_lipper_4]: http://blogs.msdn.com/b/ericlippert/archive/2007/10/22/covariance-and-contravariance-in-c-part-four-real-delegate-variance.aspx
[eric_lipper_5]: http://blogs.msdn.com/b/ericlippert/archive/2007/10/24/covariance-and-contravariance-in-c-part-five-higher-order-functions-hurt-my-brain.aspx
[so_discussion]: http://stackoverflow.com/questions/676615/why-is-scalas-immutable-set-not-covariant-in-its-type
