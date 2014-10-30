---
layout: post
title: Hibernate with Scala - with less pain
date: 2014-10-28
comments: true
---

**Note: If you want to stick with JPA only, this post might not be helpful.**

I have been working with the Spring framework and java for the majority of my professional career, so when I need to start working on a new project involving them, the choices are pretty much always clear. Persistence pretty much always means using Hibernate.

For the past year and a half, I have been learning Scala (and Clojure, and Scheme, but that's another story). And 6-8 months ago, I started getting paid for working in Scala. Now, one of the first problems that I ran into was which frameworks to use. With the web frameworks, the choice was between the three leading frameworks, [Play][play-framework], [Lift][lift-web] and [Scalatra][scalatra]. I was facinated with Scalatra, but chossing Play Framework seemed like a better choice since it seemed to have better docs, more books and a lot of momentum. Plus, it was backed by [Typesafe][typesafe].

Then, I needed to choose a reasonably good persistence framework. That's where I was stuck. I am still kinda stuck. Even though I decided to use [Hibernate][hibernate], I have never been satisfied. Hibernate begins to feel a bit clunky when you have to deal with [`Option`][scala-option] types, scala collections or immutability. I am very much inclined to use [JOOQ][jooq] or [Slick][slick]. However, I don't want to give up the convenience of Hibernate altogether. I guess I'll have to come up with a way to combine Hibernate with one of these two[^1] without adding too much boilterplate or increasing the cognitive load. That is still a work in progress. Sticking with Hibernate alone was an extremely hard and frustrating decision. I wanted something more "functional" in nature, without the associated boilerplate.


With that out of the way, I think Hibernate is going to stick around for a while[^2]. This post is an attempt to share a few things that I am doing to make Hibernate work reasonably well with Scala.


## Problem #1: Dealing with Option[X] types

If you use [`AccessType.FIELD`][accesstype-field] for non-option types, hibernate picks them up without any fuss (unless you are using scala collections, of course). However, Hibernate doesn't know how to deal with `Option` types yet. And there is no generic, one-shot way to tell hibernate how to do so.

### First (frustrating) approach

The easiest thing to do is using [`AccessType.PROPERTY`][accesstype-property] to do the following:

{% highlight scala  %}

@Entity
class User extends PersistentEntity {
  // ...

  @org.hibernate.annotations.Type(`type` = "string")
  @Access(AccessType.PROPERTY)
  var website: Option[String] = None
  
  private def getWebsite = website.getOrElse(null)
  private def setWebsite(e: String) = website = Option(e)
  
  // ...
}  

{% endhighlight %}


You end up having to write getter/setter pairs for each property that uses the `Option` type. To prevent your API users from accidentally using these accessors, you make them `private` (Hibernate uses reflection to access them). To be extra careful, you may even declare them as `private[this]`.


Another, similar approach is:

{% highlight scala  %}

@Entity
class User extends PersistentEntity {
  // ...

@Entity
@Access(AccessType.FIELD)
class User extends PersistentEntity {
  // ...

  private var _website: String = _
  
  def website = Option(_website)
  def website_=(e: Option[String]) = _website = e.getOrElse(null)

  // ...
}  

{% endhighlight %}

Things are looking a little better now. This is a bit more scala-like. You use field-access and have a private field that hibernate can deal with. Use your own methods to wrap/unwrap `Option`. If you don't like the generated column names, override the default [`NamingStrategy`][naming-strategy].  This is much less verbose than the previous method.


### A little less tedium with UserTypes

{% highlight scala linenos %}

package persistence.usertypes

import java.sql.{PreparedStatement, ResultSet}
import org.hibernate.`type`.AbstractSingleColumnStandardBasicType
import org.hibernate.`type`.StandardBasicTypes._
import org.hibernate.engine.spi.SessionImplementor
import org.hibernate.usertype.UserType

object AbstractOptionType {
  val scalaToHibernateTypes: Map[Class[_], AbstractSingleColumnStandardBasicType[_]] =
    Map {
      classOf[Integer] -> INTEGER
      classOf[Long] -> LONG
      classOf[Float] -> FLOAT
      classOf[Double] -> DOUBLE
      classOf[Boolean] -> BOOLEAN
    }
}

abstract class AbstractOptionType[T](theClass: Class[T]) extends UserType {

  private val standardBasicType = AbstractOptionType.scalaToHibernateTypes(theClass)

  override def nullSafeGet(rs: ResultSet, names: Array[String], session: SessionImplementor, owner: Object) = {
    val x = standardBasicType.nullSafeGet(rs, names, session, owner)
    if (x == null) None else Some(x)
  }

  override def nullSafeSet(ps: PreparedStatement, value: Object, index: Int, session: SessionImplementor) = {
    standardBasicType.nullSafeSet(ps, value.asInstanceOf[Option[_]].getOrElse(null), index, session)
  }

  override def assemble(cached: java.io.Serializable, owner: Object): Object = cached.asInstanceOf[Object]

  override def deepCopy(value: Object) = value

  override def disassemble(value: Object) = value.asInstanceOf[java.io.Serializable]

  override def replace(original: Object, target: Object, owner: Object) = original

  override def equals(x: Object, y: Object) = x == y

  override def hashCode(x: Object) = x.hashCode

  override def isMutable = false

  override def returnedClass = classOf[Option[T]]

  override def sqlTypes = Array(standardBasicType.sqlType)
}


{% endhighlight %}




## Footnotes

[^1]: I am leaning more towards [JOOQ][jooq] at this point. Expecially after [this little chat][twitter-chat] with [@lukaseder][lukas-eder], one of the developers of the JOOQ framework.

[^2]: And I don't have much against Hibernate. Even with all the hate that ORMs get, Hibernate does its job quite well. I think it's an extremely powerful and highly customizable tool. Of course, if you are just sticking to JPA-only features, you are missing out on a vast array of functionality and knobs that Hibernate provides. Yeah, yeah, I know, you might want to switch your persistence provider in the future. I, on the other hand, don't see that happening very frequenly.


[play-framework]: https://playframework.com/
[lift-web]: http://liftweb.net/
[scalatra]: http://www.scalatra.org/
[typesafe]: https://typesafe.com/
[typesafe-team]: https://typesafe.com/company/team
[hibernate]: http://www.hibernate.org/
[slick]: http://slick.typesafe.com/
[jooq]: http://www.jooq.org/
[activate]: http://activate-framework.org/
[twitter-chat]: https://twitter.com/bhashit/status/527302658260881408
[scala-option]: http://www.scala-lang.org/api/current/scala/Option.html
[slick-relationships]: http://slick.typesafe.com/doc/2.1.0/orm-to-slick.html#relationships
[lukas-eder]: https://twitter.com/lukaseder
[access-annotattion]: http://docs.oracle.com/javaee/7/api/javax/persistence/Access.html
[accesstype-property]: http://docs.oracle.com/javaee/7/api/javax/persistence/AccessType.html#PROPERTY
[accesstype-field]: http://docs.oracle.com/javaee/7/api/javax/persistence/AccessType.html#FIELD
[naming-strategy]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/cfg/NamingStrategy.html
