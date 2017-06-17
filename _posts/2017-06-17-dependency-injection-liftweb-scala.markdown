---
layout: post
title: Using built-in dependency injection with Liftweb
date: 2017-06-17
comments: true
---

Dependency injection in liftweb is sort of
a [point of contention][di-liftweb-googlegroup] for a lot of
people. It used to be like that for me as well. And initially, I too
would have preferred using Guice (or some other specific framework)
for my DI needs. However, I think lift's built-in DI mechanisms are
good enough for most of my needs, even with a project that has
literally thousands of scala classes. Thanks to David Pollak's
comments, I (would like to) think that my eyes have been opened
:wink:. The abstraction turtles have to end somewhere.

I am writing this post to enumerate some basic DI utilities that lift
has, since these don't seem to be well-documented.

Lift's DI is based on two main traits [`Injector`][injector]
and [`Maker`][maker]. However, most of the time, you won't need to
deal with them directly. The elements that you would use are:

1. [`Factory`][factory] trait
2. [`FactoryMaker`][factory-maker], which is an abstract class inside
   inside the [`Factory`] trait.
   
Let me just show you an example of how these two are supposed to be
used:

```scala

object DependencyFactory extends Factory {
  
  object emailService extends FactoryMaker(emailServiceImpl)
  
  object sequenceNumberService extends FactoryMaker(seq.incrementAndGet _)
  
  private def emailServiceImpl: EmailService = Props.mode match {
    case Props.RunModes.Production | Props.RunModes.Staging => new RealEmailService
    // A stub for local development
    case _ => new TestEmailService
  }
  
  private val seq = new AtomicLong(0)
}

```

This defines two managed dependencies. In the case of the
`emailService`, the dependency changes based on whether we are running
on production/staging mode or some other mode (e.g. development). The
`emailService` is a singleton dependency. That is, everyone
who injects this dependency will get the same instance throughout the
app. 

Note that `FactoryMaker` constructor takes a stock scala function that
returns an instance of the needed type. A [`Function0`][function0] to
be precise [^1]. If you pass in a pre-created instance, as we are
doing in the case of `emailService`, that instance will always be
returned when this dependency is injected (scala converts that
instance to a `Function0`). This is similar to
the [singleton scope][spring-di-singleton] in spring DI.

However, since this is a normal scala function, you can write whatever
you want within that function. For ex. as the `sequenceNumberService`
illustrates, you can generate a new instance every time it needs to be
injected. Instances could be generated every time, or they could be
generated fresh if some condition is met and so on and so forth.

Here's how you can use these dependencies:

```scala
class SomeClass {
  private val emailService = DependencyFactory.emailService()
  // Or alternatively, if you don't have the FactoryMaker for a given type
  private val someType = 
    DependencyFactory.inject[SomeType]
      .openOrThrowException("No instance of SomeType found")
}
```

You can use the `apply` or `vend` method on the FactoryMaker directly,
which will give you the instance you need. I'll be using `vend` for
further examples just so that it's clear what's happening.

Calling `DependencyFactory.sequenceNumberService.vend` will return a
new number every time, since that's how it's been setup. This is, in
spring terminology, [the prototype scope][spring-di-prototype].

An alternative way is to use `DependencyFactory.inject`. But it
returns `Box` and it's only needed if don't have a FactoryMaker for a
given dependency. Which brings us to the fact that there are other
ways of registering dependencies apart from the FactoryMaker. For ex.

```scala
class SomeOtherClass {
  private val someTypeInstance = new SomeType
  DependencyFactory.registerInjection[SomeType](() => someTypeInstance)
}
```

This can be used by arbitrary code in your app to register injectable
instance. Here again, a singleton is registered. In this case, when
you need to access the registered instance, you have to necessarily
use `DependencyFactory.inject[SomeType]`.

## Overriding dependencies for sessions or requests

If you have a scenario where you need to override a given dependency
for the duration of the current session, you can do something like
following (for ex. in your snippet code):

```
val customEmailService = new CustomEmailService(currentUser)
DependencyFactory.emailService.session(customEmailService)
```

This will set the `emailService` to always return the
`customEmailService` instance for the duration of the current session
for the current user.

Note that is not equivalent to the [session-scoped][spring-di-session]
dependency in spring. This is done explicitly in your application
code. You are overriding a default dependency with some custom
instance for the duration of *this* session. No other user is going to
see it. And as soon as this session is over, it will be gone until you
set it explicitly again.

You can something similar when you need to override a dependency
during a given request.

```
val customEmailService = new CustomEmailService(currentUser)
DependencyFactory.emailService.request(customEmailService)
```

Again, this is not a [request scoped dependency][spring-di-request] as
identified by spring.

## Session or request scoped dependencies

The above examples only set the dependencies for the duration of a
given session or request,and only when the relevant code that sets
those dependencies was executed. 

What if you want to always create a session/request scoped dependency
for all the users. Let's talk
about [session scoped][spring-di-session] dependencies. The discussion
would be identical for
the [request scoped dependencies][spring-di-request]. With session
scoped dependencies, we want a new instance to be created for each
session, for all the users.

### Request scope with request lifecycle hooks

In your Boot.scala, which is used for instantiating and configuring
various stuff in lift:

```scala
class Boot {

  LiftSession.onBeginServicing = ((sess: LiftSession, req: Req) => {
    DependencyFactory.awesomeService.request.set(new AwesomeService {})
  }) :: LiftSession.onBeginServicing

  ...
}

```

This will set a new instance on every request, right at the beginning
of the request servicing. So, calling
`DependencyFactory.awesomeService.vend` will return the instance
created for the particular request.

### Session scope with session lifecycle hooks

Similarly, you can do it for sessions in `Boot.scala`:

```scala
class Boot {

  LiftSession.afterSessionCreate = ((_: LiftSession, req: Req) => {
    DependencyFactory.awesomeService.session.set(new AwesomeService {})
  }) :: LiftSession.afterSessionCreate  
  
  ...
}

```

That's pretty much it.

## Overriding dependency factory for tests and mocking dependencies

The above cases handle most of the stuff you will need. When testing,
all of your tests might need to mock some of the services, without
affecting other tests. Doing this manually would be a nightmarish,
extremely prone to errors. The way to do it is to have isolated
dependency graph for your tests. The key is realizing that the
`DependencyFactory` could be just a normal scala instance that itself
can be injected as needed. I'm sure David Pollak is smiling right
now. This is what he keeps saying repeatedly (at least I hope
so). It's just scala code. There is no magic here.

A trait that represents your `DependencyFactory`

```scala
trait DependencyFactory extends Factory {

 implicit object cardService extends FactoryMaker(cardServiceVendor)
 ...
 
 
 // the default implementation of card-service
 // this is the method you override when needed
 protected def cardServiceVendor: Vendor[CardService] = new PaymentCardService

 // other such vendors
 ...
}

object DependencyFactory extends Factory {
  // the default instance that will be used unless overridden
  private val DefaultInstance = new DependencyFactory {}
  
  object instance extends FactoryMaker[DependencyFactory](DefaultInstance)

  // Allow making calls directly on DependencyFactory companion object instead of
  // having to use DependencyFactory.instance
  implicit def dependencyFactoryToInstance(dft: DependencyFactory.type): DependencyFactory = instance.vend

  // you shouldn't write code that needs this, this is just an example
  def resetDefault = instance.default.set(DefaultInstance)
}

```

Now, when you do `DependencyFactory.cardService.vend`, it will using
the `DefaultInstance`. Your call will be implicitly translated to
`DependencyFactory.instance.vend.cardService.vend`. This is the part
that allows you to completely override everything you need in your
dependency graph. For ex. you could do this in your tests:


```scala
class SomeSpec {

  override def beforeAll = DependencyFactory.instance.default.set({
    new DependencyFactory {
      override def cardServiceVendor: util.Vendor[CardService] = mock[CardService]
    }
  })

  override def afterAll: Unit = DependencyFactory.resetDefault
}

```

However, there is a possible problem with this approach (I haven't
tested it). If your test suites are running in parallel, this
set/reset of the default instance will be problematic. I don't
recommend this approach unless you know what you are doing.

One safe way of doing this is to use the stackable nature of the
`Makers`:

```scala
private val customDepFactory = new DependencyFactory {
  override def cardServiceVendor: util.Vendor[CardService] = mock[CardService]
}

DependencyFactory.instance.doWith(customDepFactory) {
  // write all your tests here
}

```

And this would work as expected. You can try to come up with variation
on how to do this without the added indentation though. For ex. you
can do following with [scalatest][http://www.scalatest.org/]:

```scala
trait DependencyOverrides extends SuiteMixin { self: Suite =>

  // Just override this and your tests will be executed with that overridden DependencyFactory instance.
  protected def dependencyFactory: Vendor[DependencyFactory] = DependencyFactory.instance

  // Run the tests with the given dependency-factory instance.
  abstract override def withFixture(test: NoArgTest): Outcome = {
    DependencyFactory.instance.doWith(dependencyFactory.vend) {
      super.withFixture(test)
    }
  }
  
}

```

Scalatest has something called [fixtures][fixtures] that comes in
really handy here. Any test where you need to provide a custom
`DependencyFactory` instance should override this trait and just
override with the custom implementation. For ex.

```scala
class SomeSpec extends ... with DependencyOverrides {
  override val dependencyFactory: Vendor[DependencyFactory] = new DependencyFactory {
    override def cardServiceVendor: util.Vendor[CardService] = mock[CardService]
    // other overrides
    ...
  }
}

```

## Conclusion

Most of the time, you should be able to do away with any specialized
DI framework. But I'd be curious to find out if there are some
specific cases that can't be handled by lift's built-in DI.


[di-liftweb-googlegroup]: https://groups.google.com/forum/#!topic/liftweb/_lleL2xpCFU
[injector]: https://liftweb.net/api/26/api/index.html#net.liftweb.util.Injector
[maker]: https://liftweb.net/api/26/api/index.html#net.liftweb.util.Maker
[factory]: https://liftweb.net/api/26/api/index.html#net.liftweb.http.Factory
[factory-maker]: https://github.com/lift/framework/blob/5033c8798d4444f81996199c10ea330770e47fbc/web/webkit/src/main/scala/net/liftweb/http/Factory.scala#L37
[function0]: http://www.scala-lang.org/api/current/scala/Function0.html
[sess-var]: https://liftweb.net/api/26/api/index.html#net.liftweb.http.SessionVar
[req-var]: https://liftweb.net/api/26/api/index.html#net.liftweb.http.RequestVar
[sess-lifecycle-callbacks]:http://chimera.labs.oreilly.com/books/1234000000030/ch06.html#_problem_54
[spring-di-prototype]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-prototype
[spring-di-singleton]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-singleton
[spring-di-session]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-session
[spring-di-request]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-request
[fixtures]: http://www.scalatest.org/user_guide/sharing_fixtures

[^1]: It's also an instance of the `Vendor` trait, but that's not important here 
