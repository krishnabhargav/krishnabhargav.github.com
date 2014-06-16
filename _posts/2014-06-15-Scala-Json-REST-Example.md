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
