---
layout: post
title: Using Hibernate with Scala Options - with less pain
date: 2014-10-28
comments: true
---

**Note: If you want to stick with JPA only, this post might not be helpful.**

I have been working with the Spring framework and java for the majority of my professional career, so when I need to start working on a new project involving them, the choices are pretty much always clear. Persistence pretty much always means using Hibernate.

For the past year and a half, I have been learning Scala (and Clojure, and Scheme, but that's another story). And 6-8 months ago, I started getting paid for working in Scala. Now, one of the first problems that I ran into was which frameworks to use. With the web frameworks, the choice was between the three leading frameworks, [Play][play-framework], [Lift][lift-web] and [Scalatra][scalatra]. I was fascinated with Scalatra, but choosing Play Framework seemed like a better choice since it seemed to have better docs, more books and a lot of momentum. Plus, it was backed by [Typesafe][typesafe].

Then, I needed to choose a reasonably good persistence framework. That's where I was stuck. I am still kinda stuck. Even though I decided to use [Hibernate][hibernate], I have never been satisfied. Hibernate begins to feel a bit clunky when you have to deal with [`Option`][scala-option] types, scala collections or immutability. I am very much inclined to use [JOOQ][jooq] or [Slick][slick]. However, I don't want to give up the convenience of Hibernate altogether. I guess I'll have to come up with a way to combine Hibernate with one of these two[^1] without adding too much boilterplate or increasing the cognitive load. That is still a work in progress. Sticking with Hibernate alone was an extremely hard and frustrating decision. I wanted something more "functional" in nature, without the associated boilerplate.


With that out of the way, I think Hibernate is going to stick around for a while[^2]. This post is an attempt to share a few things that I am doing to make Hibernate work reasonably well with Scala.

## Dealing with Option[X] types

If you use [`AccessType.FIELD`][accesstype-field] for non-option types, hibernate picks them up without any fuss (unless you are using scala collections, of course). However, Hibernate doesn't know how to deal with `Option` types yet. And there is no generic, one-shot way to tell hibernate how to do so.

### First (frustrating) approach

The easiest thing to do is using [`AccessType.PROPERTY`][accesstype-property] to do the following:

{% highlight scala %}

@Entity
class User extends PersistentEntity {
  ...
  @org.hibernate.annotations.Type(`type` = "string")
  @Access(AccessType.PROPERTY)
  var website: Option[String] = None
  
  private def getWebsite = website.getOrElse(null)
  private def setWebsite(e: String) = website = Option(e)
  ...
}  

{% endhighlight %}


You end up having to write getter/setter pairs for each property that uses the `Option` type. To prevent your API users from accidentally using these accessors, you make them `private` (Hibernate uses reflection to access them). To be extra careful, you may even declare them as `private[this]`.


Another, similar approach is:

{% highlight scala %}

@Entity
@Access(AccessType.FIELD)
class User extends PersistentEntity {
  ...
  private var _website: String = _
  
  def website = Option(_website)
  def website_=(e: Option[String]) = _website = e.getOrElse(null)
  ...
}

{% endhighlight %}

Things are looking a little better now. This is a bit more scala-like. You use field-access and have a private field that hibernate can deal with. Use your own methods to wrap/unwrap `Option`. If you don't like the generated column names, override the default [`NamingStrategy`][naming-strategy].  This is much less verbose than the previous method.

### A partial solution to target scalar types

There is a very easy way to make hibernate work with optional
scalars. You can implement the JPA
[`AttributeConverter`][attribute-converter] with the
[`@Converter`][converter] annotation to convert between your scala
types and the their database representations. It's pretty
straightforward, so I am not providing a code sample. The
[`Converter`][converter] annotation comes with an `autoApply`
attribute and when it is set to `true`, the conversion is supposed to
be applied automatically to the target types. 


### A partial solution using UserTypes (scalar types only)

You can easily define [`UserType`][usertype] implementations for all your scalar types. 

{% highlight scala %}

package persistence.usertypes

import java.sql.{PreparedStatement, ResultSet}
import org.hibernate.`type`.AbstractSingleColumnStandardBasicType
import org.hibernate.engine.spi.SessionImplementor
import org.hibernate.usertype.UserType

abstract class AbstractOptionType[T](theClass: Class[T],
    standardBasicType: AbstractSingleColumnStandardBasicType[_]) extends UserType {

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

This class is just either wrapping the scalar values inside `Option` or unwrapping them from it. Once you have this, defining custom types is extremely easy.

{% highlight scala %}

import org.hibernate.`type`.StandardBasicTypes

class OptionInt extends AbstractOptionType(classOf[Int], StandardBasicTypes.INTEGER)

class OptionLong extends AbstractOptionType(classOf[Long], StandardBasicTypes.LONG)

class OptionBigDecimal extends AbstractOptionType(classOf[BigDecimal], StandardBasicTypes.BIG_DECIMAL)

class OptionString extends AbstractOptionType(classOf[Boolean], StandardBasicTypes.STRING)

// and so on...

{% endhighlight %}

Once the custom types are defined, use them with the [`Type`][hibernate-type] to annotate your fields.

{% highlight scala %}

@Type(`type` = "your.path.OptionLong")
var id: Option[Long] = None

{% endhighlight %}

The [`StandardBasicTypes`][standardbasictypes] class defines several other scalar tyeps, so you can use as many as you like to define your own `Option` wrapper types. Plus, if you don't want to put in full paths in the [`Type`][hibernate-type] annotation every time you use them, you can give your custom types names using [`@TypeDef`][typedef] annotations and use those names instead. Either way, you still need to use the [`Type`][hibernate-type] annotation everywhere[^3].

### The best way (that works with all types)

Even if the [`AttributeConverter`][attribute-converter] and the related annotations worked, they still can't deal with the composite types. You can try using the [`CompositeUserType`][composite-usertype]; Good luck and have fun. However, what if I told you there is a better, and much less frustrating solution? Well, there is. This one involves extending [`PropertyAccessor`][propertyaccessor] and [`PojoEntityTuplizer`][pojo-entitytuplizer]. Don't worry though, it really isn't as bad as it sounds. In fact, it is quite easy.

If you have worked long enough with Hibernate, you might be aware that it provides all kinds of pluggability. One of those pluggable thing is how hibernate will access the individual properties of a class. You can provide a custom strategy for accessing the properties. If you have been using JPA's [`@Access`][access-annotattion] or Hibernate's [`@AccessType`][hibernate-accesstype] annotations, you have been doing that already. I am just going to take it bit further.

### Define a PropertyAccessor that works with scala methods

A [`PropertyAccessor`][propertyaccessor] is supposed to return a [`Getter`][getter] and a [`Setter`][setter] implementations. Which are exactly what they sound like. They are an abstraction over how those properties of a class are accessed. Your implementation will be given the class-name and the property-name that hibernate needs to access, and you return instances that allow hibernate to do that. The methods are very appropriately named `getGetter` and `getSetter`.

When you declare a `var` in scala, the scala compiler actually creates a field by that name and creates the getter and setter. The getter has the same name as the property, whereas the setter is named like `propertyName_=`. Since java identifers cannot have an equal sign in them, internally, this setter is represented as `propertyName_$eq`.

So, once you have these two pieces of information, all you need to do get the [`java.lang.Method`][java-lang-method] instance that represent the getter and the setter for the given property-name; and that is easy, right. one word: reflection. In my code, I am using the scala reflection api, but the Java API would work just as well.

A [`Getter`][getter] has two responsibilites:

1. Get the return-type of property (the getter method)
2. fetch the value of the property from the given instance.

A [`Setter`][setter] has only one responsibility:

1. Set the given value on the given object.


Do you see it now! Well, here are the three facts

1. You are given the class-name and the property-name
2. When hibernate needs the value of a property, it asks your [`Getter`], passing it the target object.
3. When hibernate needs to set the value of a property, it asks your [`Setter`], passing it the target object and the value to set.

All done. This is it. When Hibernate asks you to set the value, wrap in an `Option`. When it asks you to fetch a value, and if you see that it an `Option`, unwrap it before returning.

The implementation is pretty simple. Just copy over the code from [`BasicPropertyAccessor`][basic-propertyaccessor] and convert it to Scala. Just for the completeness' sake, I am including my implementation of the PropertyAccessor. It has warts. I know it can be improved, and I am working on it, but I am eager to share this.

{% highlight scala %}
package persistence.property

import scala.reflect.runtime.{universe => ru}
import scala.reflect.api._

import java.beans.Introspector
import java.lang.reflect.InvocationTargetException
import java.lang.reflect.Method
import java.util.Map
import _root_.org.hibernate.{ PropertyAccessException, PropertyNotFoundException, PropertySetterAccessException }
import _root_.org.hibernate.engine.spi.{ SessionFactoryImplementor, SessionImplementor }
import _root_.org.hibernate.property.{ Getter, PropertyAccessor, Setter, BasicPropertyAccessor }
import _root_.org.slf4j.LoggerFactory

// TODO this class might need some work to make it look better. Keep an eye on it.
class ScalaPropertyAccessor extends BasicPropertyAccessor {
  import ScalaPropertyAccessor._
  
  override def getSetter(theClass: Class[_], propertyName: String): Setter = {
    createSetter(theClass, propertyName);
  }

  override def getGetter(theClass: Class[_], propertyName: String): Getter = {
    createGetter(theClass, propertyName);
  }  
}


object ScalaPropertyAccessor {

  var playApplication: Option[play.api.Application] = None

  // In scala, the setter for the vars is named like <property-name>_$eq
  private def setterMethod(theClass: Class[_], propertyName: String) = {
    val getter = getGetterOrNull(theClass, propertyName)
    val returnType = if (getter == null) null else getter.getReturnType
    val methods = theClass.getDeclaredMethods()
    // TODO add check for the argument type. Also, once the scala reflection API is
    // table enough, move all code to use it.
    methods.
      filter(_.getParameterTypes.length == 1).
      find(_.getName == (propertyName + "_$eq")).getOrElse(null)
  }

  private final class BasicGetter(private val clazz: Class[_], private val method: Method, private val propertyName: String) extends Getter {
    override def get(target: AnyRef): java.lang.Object = {
      val returned = method.invoke(unwrappedTarget)
      if(returned.isInstanceOf[Option[_]]) {
        returned.asInstanceOf[Option[_]].getOrElse(null).asInstanceOf[Object]
      } else {
        returned
      }
    }

    override def getForInsert(target: AnyRef,
        mergeMap: java.util.Map[_, _],
        session: SessionImplementor) = {
      get(target)
    }

    override val getReturnType: Class[_] = {
      val returnType = getReturnTypeForGetter(clazz, propertyName)
      val className = if(isReturnTypeOption(returnType)) {
        returnType.typeArgs(0).typeSymbol.fullName
      } else {
        returnType.typeSymbol.fullName
      }

      if(playApplication.isDefined) {
        playApplication.get.classloader.loadClass(className)
      } else {
        throw new IllegalStateException("The Play Application ClassLoader must be set before building the SessionFactory")
      }
    }

    override def getMember() = method

    // This is an optional method in the Getter interface. The documented approach is to
    // return null when you don't want to implement this method. Keeping it unimplemented
    // so that the method isn't exposed and the getting/setting is always intercepted by
    // us.
    override def getMethod() = null
    override def getMethodName() = method.getName
    override def toString() = "BasicGetter(" + clazz.getName() + '.' + propertyName + ')'
    def readResolve(): AnyRef = createGetter(clazz, propertyName)
  }

  private def isReturnTypeOption(returnType: ru.Type) = {
    val mirror = ru.runtimeMirror(getClass.getClassLoader)
    val optionTypeSymbol = mirror.classSymbol(classOf[Option[_]])
    returnType.typeSymbol == optionTypeSymbol
  }

  private def getReturnTypeForGetter(clazz: Class[_], propertyName: String): ru.Type = {
    val mirror = ru.runtimeMirror(getClass.getClassLoader)
    val classSymbol = mirror.classSymbol(clazz)
    val getter = classSymbol.selfType.member(ru.TermName(propertyName)).asMethod.getter
    getter.info.resultType
  }

  private final class BasicSetter(val clazz: Class[_], val method: Method, val propertyName: String, val isOptionType: Boolean) extends Setter {
    override def set(target: AnyRef, value: AnyRef, factory: SessionFactoryImplementor) = {
      val valueToSet = if(isOptionType) Option(value) else value
      method.invoke(target, valueToSet)
    }

    // see the comment on BasicGetter#getMethod
    override def getMethod() = null
    override def getMethodName() = method.getName
    def readResolve() = createSetter(clazz, propertyName)
    override def toString() = "BasicSetter(" + clazz.getName() + '.' + propertyName + ')'
  }

  private def createSetter(theClass: Class[_], propertyName: String): Setter = {
    val result: BasicSetter = getSetterOrNull(theClass, propertyName);
    if (result == null) {
      throw new PropertyNotFoundException("Could not find a setter for property " +
          propertyName + " in class " + theClass.getName())
    }
    return result
  }

  private def getSetterOrNull(theClass: Class[_], propertyName: String): BasicSetter = {
    if (theClass == classOf[Object] || theClass == null) null
    else {
      val method = setterMethod(theClass, propertyName)
      if (method != null) {
        method.setAccessible(true)
        val returnType = getReturnTypeForGetter(theClass, propertyName)
        new BasicSetter(theClass, method, propertyName, isReturnTypeOption(returnType))
      } else {
        var setter = getSetterOrNull(theClass.getSuperclass(), propertyName)
        if (setter == null) {
          val interfaces = theClass.getInterfaces()
          var i = 0
          while (i < interfaces.length && setter == null) {
            setter = getSetterOrNull(interfaces(i), propertyName)
            i += 1
          }

        }
        setter
      }
    }
  }


  private def createGetter(theClass: Class[_], propertyName: String): BasicGetter = {
    val result = getGetterOrNull(theClass, propertyName);
    if (result == null) {
      throw new PropertyNotFoundException(s"Could not find a getter for $propertyName in class ${theClass.getName()}")
    } else {
      result
    }
  }

  private def getGetterOrNull(theClass: Class[_], propertyName: String): BasicGetter = {
    if (theClass == classOf[Object] || theClass == null) {
      return null;
    }

    val method = getterMethod(theClass, propertyName);

    if (method != null) {
      method.setAccessible(true);
      return new BasicGetter(theClass, method, propertyName);
    } else {
      var getter = getGetterOrNull(theClass.getSuperclass(), propertyName);
      if (getter == null) {
        val interfaces = theClass.getInterfaces();
        var i = 0
        while (i < interfaces.length && getter == null) {
          getter = getGetterOrNull(interfaces(i), propertyName);
          i += 1
        }
      }
      getter;
    }
  }

  private def getterMethod(theClass: Class[_], propertyName: String): Method = {
    val methods = theClass.getDeclaredMethods()
    methods.
      filter(_.getParameterTypes().length == 0).
      filter(!_.isBridge).
      find(_.getName == propertyName).getOrElse(null)
  }
}


{% endhighlight %}

This class needs access to the play framework's [Application][play-application] object. It's needed for getting java Class objects. If you are not using play, replace this with appropriate code. If you are using play, see [this later sections](#load-time-weaving-in-play-dev-mode).

### Telling Hibernate to use our PropertyAccessor

An easy way to make Hibernate use our `PropertyAccessors` is to use Hibernate's [`AccessType`][hibernate-accesstype][^4] annotation and specify the class-name of your `PropertyAccessor` as its value. Unfortunately, there is a [bug in Hibernate][accesstype-bug] that makes it ignore our custom property-accessors[^5]. What happens internally is that if your property-access strategy name doesn't match one of "field" or "property", it is defaulted to one of them. It's implemented as an enum and there is nothing anyone can do about it. No pluggability anywhere around.

Hibernate also provides an annotation called [`@AttributeAccessor`][attribute-accessor], which is supposed to replace their, now deprecated, `@AccessType` annotation. However, at the moment of writing this, this annotation doesn't work at all. There are no references to that annotation in the entire hibernate source code (version 4.3.6).

After wading through a bunch of Hibernate code, I finally found a way to work around that. Hibernate provides another pluggable feature called [EntityTuplizer][entity-tuplizer] where you can customize the way an entity or a compoent is tuplized. There is also a default implementation called [PojoEntityTuplizer][pojo-entity-tuplizer] that is used for POJOs. In that class, there is a pair of methods called `buildPropertyGetter` and `buildPropertySetter`. These methods are used for building the [Getter][getter] and [Setter][setter] implementations for all properties of a pojo. We already have a class that does this for us. All we need to do is hook it in. The aforementioned methods are passed instances of [`Property`][mapped-property]. These instances have a field named "propertyAccessorName". The value of this field is a string representing our property-access strategy. Yeah, the same strategy that we would have used with the [`AccessType`][hibernate-accesstype] annotation. So now:

1. We extend [`PojoEntityTuplizer`][pojo-entity-tuplizer]
2. override the `buildPropertyGetter` and `buildPropertySetter`
3. when those two methods are called with a [`Property`][mapped-property] instance, we set its `propertyAccessorName` field to the name of our custom property-accessor class

There is another little snag: the class that overrides [`PojoEntityTuplizer`][pojo-entity-tuplizer] must be written in Java. Since the entity-tuplizers need to have two very specific constructors. If those constructors are not present, Hibernate doesn't accept our tuplizer. In scala, there is no way to call multiple super-class constructors. So, we write a bit of Java.

{% highlight java %}

import org.hibernate.mapping.*;
import org.hibernate.tuple.entity.*;
import org.hibernate.metamodel.binding.*;
import org.hibernate.property.*;

public class CustomPojoEntityTuplizer extends PojoEntityTuplizer {
  public CustomPojoEntityTuplizer(EntityMetamodel emm, EntityBinding eb) {
    super(emm, eb);
  }

  public CustomPojoEntityTuplizer(EntityMetamodel emm, PersistentClass pc) {
    super(emm, pc);
  }

  @Override
  protected Getter buildPropertyGetter(Property mappedProperty, PersistentClass mappedEntity) {
    mappedProperty.setPropertyAccessorName("persistence.property.ScalaPropertyAccessor");
    return super.buildPropertyGetter(mappedProperty, mappedEntity);
  }

  @Override
  protected Setter buildPropertySetter(Property mappedProperty, PersistentClass mappedEntity) {
    mappedProperty.setPropertyAccessorName("persistence.property.ScalaPropertyAccessor");
    return super.buildPropertySetter(mappedProperty, mappedEntity);
  }
}

{% endhighlight %}

Once we have that in place, give this class to Hibernate while building the session-factory.

{% highlight scala %}

def buildSessionFactory() = {
  val configuration = new Configuration

  // Our custom Tuplizer to take care of custom property access.
  configuration.getEntityTuplizerFactory.
    registerDefaultTuplizerClass(EntityMode.POJO, classOf[CustomPojoEntityTuplizer])
    
  ...
  
  val srb = new StandardServiceRegistryBuilder().applySettings(configuration.getProperties);
  configuration.buildSessionFactory(srb.build());
}

{% endhighlight %}

### Dealing with Embeddables

Even though you are setting your own property-access stragety for all the properties of all your classes, it still doesn't work with [`Embeddable`][embeddable] types. That is, if your embeddable components themselves contain `Option` fields. For that, you need to use another knob. This one is called [`ComponentTuplizer`][component-tuplizer]. Specifically, you need to extend the in-build [`PojoComponentTuplizer`][pojo-component-tuplizer] and basically do the same thing that we did earlier with entity-tuplizer: Override the relevant methods to specify our own property-access strategy.

{% highlight scala %}


import org.hibernate.mapping.Property
import org.hibernate.property.Getter
import org.hibernate.property.Setter
import org.hibernate.tuple.component.PojoComponentTuplizer
import org.hibernate.mapping.Component

class CustomPojoComponentTuplizer(component: Component) extends PojoComponentTuplizer(component) {
  override def buildGetter(component: Component, prop: Property): Getter = {
    prop.setPropertyAccessorName("persistence.property.ScalaPropertyAccessor")
    prop.getGetter(component.getComponentClass())
  }

  override def buildSetter(component: Component, prop: Property): Setter = {
    prop.setPropertyAccessorName("persistence.property.ScalaPropertyAccessor")
    prop.getSetter(component.getComponentClass())
  }
}

{% endhighlight %}

This has ony one required constructor, so you don't need to write Java. Okay, so how do you hook this in? Turns out that like entities, there is no global way to plug-in our own component-tuplizer. We need to do it on each individual property.

### The broken approach with @Tuplizer

Use the [`@Tuplizer`][tuplizer] annotation on each [`@Embeddable`][embeddable] property.

{% highlight scala %}

@Entity
class User extends PersistentEntity {

  @Embedded
  @Tuplizer(impl = classOf[CustomPojoComponentTuplizer])
  var mainProfile: UserProfile = _

  @Embedded @org.hibernate.annotations.Target(classOf[Address])
  @Tuplizer(impl = classOf[CustomPojoComponentTuplizer])
  var address: Option[Address] = None

  ...

}

{% endhighlight %}


If your embeddable itself is optional, then you need to specify its type explicitly using [`@Target`][target-annotation]. Extremely annoying.

And no, putting [`@Tuplizer`][tuplizer] on the embeddable class itself does not work. That is fairly annoying if you need to use the same embeddable in multiple places; especially because the docs for the annotation say:

> Define a tuplizer for an entity or a component.

Even after doing all that, hibernate keeps refusing to behave correctly. A combination of [`@ElementCollection`][element-collection] of [`@Embeddable`][embeddable] elements with nested embeddables and option types doesn't work with our custom property-access strategy. This is a [bug in hibernate][HHH-9089][^6]. I was so frustrated by this titime, I decided to pull out the big guns.

### Using AspectJ to inject our property-access strategy

Yeah baby, I am going to mess with the bytecodes. Submitting patches to hibernate code and waiting for the issues to be resolved was not an option. I didn't know how long the entire process would take. Some of the issues that I linked in this post have been open for more than 2 years. Also, some of things I wanted to do weren't even bugs. They might not even be accepted by the hibernate community[^7]. So, yeah, I am going to use aspects *with scala*[^8].

Still here? Okay. In the entire hibernate source code, there is only one place where the [`PojoComponentTuplizer`][pojo-component-tuplizer] is being used. I wrapped some code around that to replace it with my `CustomPojoComponentTuplizer` described previouly. It was surprisingly simple.

{% highlight scala %}

@Aspect
public class ComponentTuplizerAspect {

  private Logger logger = LoggerFactory.getLogger(getClass());

  @Around("execution (private static * org.hibernate.tuple.component.ComponentTuplizerFactory.buildBaseMapping())")
  public Object setupDefaultComponentTuplizer(ProceedingJoinPoint pjp) throws Throwable  {
    logger.debug("Applying aspect around ComponentTuplizerFactory.buildBaseMapping()");
    java.util.Map<EntityMode, Class<? extends ComponentTuplizer>> map
      = (java.util.Map<EntityMode, Class<? extends ComponentTuplizer>>) pjp.proceed();
    map.put(EntityMode.POJO, CustomPojoComponentTuplizer.class);
    return map;
  }
}

{% endhighlight %}

Take a look at the [source code of `buildBaseMapping()`][build-base-mapping-code] in [`ComponentTuplizerFactory`][component-tuplizer-factory] and you will understand what is happening here. I am just replacing the default component-tuplizer with my own. This is done via [load time weaving][load-time-weaving] of aspects. Compile time weaving was not really an option since I needed to weave into already-compiled, third-party classes.

I am aware that the solution with aspects might break when the new version of Hibernate comes out. It won't be that big of a problem and they might have fixed some of the issues by then.

### Are we there yet? I am getting annoyed.

That's it. We are done. And if this sounds a bit hacky, that's because it is. And I am okay with that. One of the advantages of working with open-source frameworks is that you can look under the hood and rearrage the wiring if needed. Of course, even though I am fairly familiar with Hibernate source code, I've never had so many WTF moments as I had while working on this. The code around the tuplizers, property-accessors and session-factory building is in a flux right now. There are several comments including the words "yucky" in that source code. So, I guess they know it too.

## A few minor annoyances

This solution works really well. However there are still a few things you need to keep in mind:

### You still need to provide explicit type info

Whenever you are using the option types, you need to provide explicit type info. That's because hibernate will try to verify the mappings for properties before it starts dealing with tuplizers. Type info can be provided using one of the following.

1. [`@Type`][hibernate-type] with optional scalars
2. [`@Target`][target-annotation] with optional embeddables
3. Specify `targetEntity` attribute with the `@OneToX` and `@ManyToX` annotations.

### Load time weaving in Play dev mode

Getting aspects to work with play framework in the development mode is a bit of a pain in the ass. The kind of aspects I needed to use could only be done via [load time weaving][load-time-weaving]. Which, as the name suggests, is related to class-loading. In dev-mode, Play has multiple classloaders to allow dynamic reloading of source code. So, the whole load-time weaving with third-party classes gets messed up. Note that this problem only occurs in development mode. Class loading in play is pretty simple in the production mode. I will be writing a short companion post about it soon.


## Conclusion

All of this exercise is required because Hibernate wasn't designed to work with Scala. However, it doesn't help that some of the things that Hibernate provides do not work. Like the custom strategy values for [`@AccessType`][hibernate-accesstype], or the [`@AttributeAccessor`][attribute-accessor] annotation. However, I am hoping that the latter will start working soon. Once it starts working, this entire post can be reduced to a quarter of its current size. 

## Footnotes

[^1]: I am leaning more towards [JOOQ][jooq] at this point. The whole API seems really cool.

[^2]: And I don't have much against Hibernate. Even with all the hate that ORMs get, Hibernate does its job quite well. I think it's an extremely powerful and highly customizable tool. Of course, if you are just sticking to JPA-only features, you are missing out on a vast array of functionality and knobs that Hibernate provides. Yeah, yeah, I know, you might want to switch your persistence provider in the future. I, on the other hand, don't see that happening very frequenly.

[^3]: Although the [`Type`][hibernate-type] has a field called `defaultForType`, it won't be very useful since it can't work with generic types due to java's erasure. The type-erasure keeps getting in the way, all the time.

[^4]: It's deprecated in favor of the [`@Access`][access-annotattion], I know. However, the Hibernate docs still point out that it is still useful if you want to provide a custom property-access strategy.

[^5]: It used to work, but the Hibernate guys are trying to make the session-factory and configuration related classes more streamlined. And this bug was probably a side-effect of that. I can see several TODO comments in the Hibernate code which indicate that this too is something they are working on. However, this could have been a low-priority issue, because, up until now, rarely anybody needed to provide custom property-accessors.

[^6]: Some relevant discussion at [this link](http://www.manning-sandbox.com/message.jspa?messageID=147449#147452), and [this stack-overflow answer](http://stackoverflow.com/questions/18441222/issue-with-jpa-mapping-for-two-nested-embeddable)

[^7]: Although, I am going to try that. One of the things I want to do is open up more hooks for customization.

[^8]: If you think aspects are evil, and against functional programming principles, you might be right. However, there are certain scenarios where using aspects does make sense. See [this post][why-aspects] for an interesting summary of when it is okay to use aspects.

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
[usertype]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/usertype/UserType.html
[standardbasictypes]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/type/StandardBasicTypes.html
[hibernate-type]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/annotations/Type.html
[typedef]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/annotations/TypeDef.html
[attribute-converter]: http://docs.oracle.com/javaee/7/api/javax/persistence/AttributeConverter.html
[attribute-converter-bug]: https://hibernate.atlassian.net/browse/HHH-8804
[converter]: http://docs.oracle.com/javaee/7/api/javax/persistence/Converter.html
[composite-usertype]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/usertype/CompositeUserType.html
[basic-propertyaccessor]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/property/BasicPropertyAccessor.html
[pojo-entitytuplizer]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/tuple/entity/PojoEntityTuplizer.html
[hibernate-accesstype]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/annotations/AccessType.html
[propertyaccessor]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/property/PropertyAccessor.html
[getter]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/property/Getter.html
[setter]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/property/Setter.html
[java-lang-method]: http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html
[accesstype-bug]: https://hibernate.atlassian.net/browse/HCANN-48
[entity-tuplizer]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/tuple/entity/EntityTuplizer.html
[pojo-entity-tuplizer]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/tuple/entity/PojoEntityTuplizer.html
[mapped-property]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/mapping/Property.html
[embeddable]: http://docs.oracle.com/javaee/7/api/javax/persistence/Embeddable.html
[component-tuplizer]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/tuple/component/ComponentTuplizer.html
[pojo-component-tuplizer]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/tuple/component/PojoComponentTuplizer.html
[tuplizer]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/annotations/Tuplizer.html
[embedded]: http://docs.oracle.com/javaee/7/api/javax/persistence/Embedded.html
[target-annotation]: org.hibernate.annotations.Target
[element-collection]: http://docs.oracle.com/javaee/7/api/javax/persistence/ElementCollection.html
[HHH-9089]: https://hibernate.atlassian.net/browse/HHH-9089
[why-aspects]: http://scala-programming-language.1934581.n4.nabble.com/Welcome-Jonas-Boner-to-the-Lift-committers-tp1969804p1969815.html
[component-tuplizer-factory]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/tuple/component/ComponentTuplizerFactory.html
[load-time-weaving]: https://www.eclipse.org/aspectj/doc/next/devguide/ltw.html
[attribute-accessor]: http://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/annotations/AttributeAccessor.html
[play-application]: https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.Application
[build-base-mapping-code]: https://github.com/hibernate/hibernate-orm/blob/b943525c80b411c9fa1f66b44f8a5b14f928bf34/hibernate-core/src/main/java/org/hibernate/tuple/component/ComponentTuplizerFactory.java#L151
