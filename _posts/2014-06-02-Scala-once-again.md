# Scala, once again

In 2009, when Scala was just getting popular, I tried to build a data-pipeline system based on Scala. At the time, the intention was to build an ETL system where data flow is defined as spring configuration files. The project went very well for a few weeks and then I moved on to better (?) things. But at the time, I really liked Scala.

Later, a month or two ago, I tried my hand at Scala only to dislike it. There is a lot of functionality baked into the Language - implicits, fuction blocks, convention methods such as apply(), etc. I did not enjoy looking at it & add to that there has been "Scala Sucks" articles that I came across. At this time, I did not really like Scala

For the past week or so, I have been closely following **Functional Programming in Scala** on Coursera (well, not really doing the assignments but may be I will start very soon). That renewed my like-ness for Scala. I am also following the **Principles of Reactive Programming** and am into the lectures given by Erik Meijer - which by the way are totally awesome. Erik is a better teacher than Martin :). Anyway, now I am beginning to like Scala (frankly, I am forcing myself to get used to it enough to like it again).

## A brain dump of some things that left an impression
The intention here is to list a bunch of things that I know about scala that are different from regular language such as C#. This is to see how much of the content has made through into my head.

#### Type Alias helps improve readability 

For example, if you have a function which returns another function of type "Int => Boolean" as shown below.

	def scalaFunc(input : Int => Boolean) : Int => Boolean = {...}

You can change it to:

	type IntToBool = Int=>Boolean
	def scalaFunc(input: IntToBool) : IntToBool = { ... }

Effective Scala entry on ["type alias"](http://twitter.github.io/effectivescala/#Types%20and%20Generics-Type%20aliases)

#### Traits are "interfaces on crack"