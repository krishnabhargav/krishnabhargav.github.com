---           
layout: post
title: Scala - Working with REST service calls and handling JSON
date: 2014-06-15
updated: 2014-06-15
comments: false
categories: scala, how to
readmore: true
---

In this post, I will try to walk through an example in Scala which shoud cover the following concepts:
- How to make REST service calls in Scala? I am using "[Dispatch]"(http://dispatch.databinder.net/Dispatch.html) library here
- How to convert JSON data to Strongly typed objects?
Please note that the JSON support is done by specifically using the json4s library and not through the internal support that Dispatch library has for JSON (am yet to figure that one out). In order to demonstrate something more real, I will be using the Google Places API, make a server call to it and convert the response back to strongly typed instances.

First things first, if you wish to run this, then you will have to get your own Google API key. The instructions are pretty clear at [Google's documentation site](https://developers.google.com/places/documentation/#Authentication). Once you have an API, you are good to go.

To get started, create a folder for your project. Then place a "build.sbt" whose contents looks as shown below.
{% gist krishnabhargav/6b16db792061b0505abf %}

Some important points, that helps a newbie in Scala:
- The build tool SBT doesnt have a "sbt new my-new-project" but any folder which has atleast one source file can act as a project in Scala
- If you want to "eclipsify" a project to be imported into Eclipse, you will need to follow the instructions here at https://github.com/typesafehub/sbteclipse. Am repeating one way to do that 
	- Open ~/.sbt/0.13/plugins/plugins.sbt on your machine
	- Add the sbt plugin : addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.5.0")
	- Then while you are in the project folder, run "sbt eclipse"
	- Note that anytime you change the build.sbt, you should run "sbt eclipse" again and refresh the project in Eclipse

Our goal here is to make a call to the Google Places service to find some locations around a specific location & that matches a keyword. I am only going to list here the parameters that I know are needed for this call to work. For other details, refer to the Places API documentation.

- key : the api key
- location : lat,long
- radius : search radius in meters
- sensor : if the location set is pulled from a device, its true.
- keyword : whatever you want, I will use starbucks in my example

So what are the steps, in general? The usual steps in any programming environment to make service calls are as follows:

1. Prepare a request object
2. Set the required parameters
3. Make the request using the proper HTTP method
4. wait for the response
5. once the response is back, check if the response is OK or not
6. If the response is OK, read the contents
7. If not, read the error

For an asynchronous way to do thins, the step 4 should be read as "once the response is available, then". So the above 7 steps are shown below. Newbie should also pay attention to some example use case on how to work with Futures (Line #21, #32) , how pattern matching in scala is so powerful (see if the if statement mixed with case at line #24)

{% gist krishnabhargav/fc17f4f7c54e938dd5d4 %}

In order to work with JSON, either you can leave JSON as is and look for specific properties that you are interested or you can "deserialize" JSON to strongly typed objects. I prefer deserializing JSON and working with real objects. This is so much easier than having to deal with RAW JSON all the time. So the steps to convert JSON to objects are:

1. Get some sample JSON output representative enough for the schema.
2. Understand the schema and write classes that matches the schema.
3. Use deserialization framework to deserialize JSON to objects.

First things first, JSON is usually mapped to "case classes" of Scala. And to further simplify the above flow, you can use an online converter that interprets the JSON you give & gives you classes. There exists [one for C#](http://json2csharp.com) and there is one for [Scala](http://json2caseclass.cleverapps.io) as well. Once you have the boiler plate code, you can work on the generated code to fit your needs. 

So get the sample output from the Google Places API and use json2caseclass to create case classes. We will be using json4s to work with JSON deserialization for Scala. And that would not just work directly with the output returned from the json2caseclass because if it does not find a value in the JSON for the case class member, it will be really unhappy. The fix for that is to make the property optional and how do you make that in Scala? The standard way of making anything option in Scala is to use Option[T]. So the corrected case classes are also listed.

{% gist krishnabhargav/7cd67a7ed94d100fb325 %}

When you bring it all together, you will get

{% gist krishnabhargav/762b7f6456bd4c91414b %}

And the output is

	SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
	SLF4J: Defaulting to no-operation (NOP) logger implementation
	SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
	ENTER TO EXIT
	Total Results: 11
	Starbucks at 1364 Centennial Ave, Piscataway Township
	Starbucks at 1 Lincoln Hwy, Edison
	Starbucks at 391 George St, New Brunswick
	Starbucks at 5000 Hadley Center Dr, South Plainfield
	Starbucks at 125 Raritan Center Pkwy, Edison
	Starbucks at 774 Route 1 N, Iselin
	Starbucks at 4 Mt Bethel Rd, Warren
	Starbucks at 250 Woodbridge Center Drive, Woodbridge Center, Woodbridge Township
	Starbucks at 120 Cedar Grove Ln, Somerset
	Starbucks at 45 Central Ave, Clark
	Starbucks @ The Courtyard Bistro at 250 Davidson Ave, Somerset
	
Later, I will try take this and create a REST service that sends the Places results to other callers. 