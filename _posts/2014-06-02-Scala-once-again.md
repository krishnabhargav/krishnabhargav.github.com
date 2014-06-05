---           
layout: post
title: Scala, once again
date: 2014-06-04 14:46:29 UTC
updated: 2014-06-04 14:46:29 UTC
comments: false
categories: Scala
readmore: true
---

In 2009, when Scala was just getting popular, I tried to build a data-pipeline system based on Scala. At the time, the intention was to build an ETL system where data flow is defined as spring configuration files. The project went very well for a few weeks and then I moved on to better (?) things. But at the time, I really liked Scala.

Later, a month or two ago, I tried my hand at Scala only to dislike it. There is a lot of functionality baked into the Language - implicits, fuction blocks, convention methods such as apply(), etc. I did not enjoy looking at it & add to that there has been "Scala Sucks" articles that I came across. At this time, I did not really like Scala

For the past week or so, I have been closely following **Functional Programming in Scala** on Coursera (well, not really doing the assignments but may be I will start very soon). That renewed my like-ness for Scala. I am also following the **Principles of Reactive Programming** and am into the lectures given by Erik Meijer - which by the way are totally awesome. Erik is a better teacher than Martin :). Anyway, now I am beginning to like Scala (frankly, I am forcing myself to get used to it enough to like it again).

## A brain dump of some things that left an impression
The intention here is to list a bunch of things that I know about scala that are different from regular language such as C#. This is to see how much of the content has made through into my head.

#### Type Alias helps improve readability 

For example, if you have a function which returns another function of type "Int => Boolean" as shown below.

{% highlight scala %}
def scalaFunc(input : Int => Boolean) : Int => Boolean = {...}
{% endhighlight %}
You can change it to:

{% highlight scala %}
type IntToBool = Int => Boolean
def scalaFunc(input: IntToBool) : IntToBool = { ... }
{% endhighlight %}

Effective Scala entry on ["type alias"](http://twitter.github.io/effectivescala/#Types%20and%20Generics-Type%20aliases)

There is also something called [abstract types](http://docs.scala-lang.org/tutorials/tour/abstract-types.html)

For a stackoverflow discussion (lengthy-one) on [abstract types vs generics](http://stackoverflow.com/a/1154727)

#### Traits are "interfaces on crack"

Traits in scala are what most people in the community say "what interfaces should have been". Traits, like abstract classes, allows you to define default implementation for methods. But traits allows you to "mixin".

In the example below ([borrowed](http://docs.scala-lang.org/tutorials/tour/mixin-class-composition.html)), shows an example which uses - abstract type usage, traits, mixin class composition with traits. 

{% highlight scala linenos %}

//Interface 
trait AbsIterator {
  type T //abstract type 
  def hasNext: Boolean
  def next: T
}

class StringIterator(s: String) extends AbsIterator {
  type T = Char
  private var i = 0
  def hasNext = i < s.length()
  def next = { val ch = s charAt i; i += 1; ch }
}

trait RichIterator extends AbsIterator {
  def foreach(f: T => Unit) { while (hasNext) f(next) }
}

object Main extends App {
  //mixin
  var strIter = new StringIterator("Krishna") with RichIterator
  strIter foreach println

  var strIter2 = new StringIterator("Vangapandu")
  //invalid declaration below.
  //var foreached = strIter2 with RichIterator[Char] 

  //create a class that does the mixins
  class StringWithForEach(str: String) 
  		extends StringIterator(str) 
		with RichIterator
		
  //any instance will have foreach
  var foreached = new StringWithForEach("Vangapandu")
  foreached foreach println
}

{% endhighlight %}