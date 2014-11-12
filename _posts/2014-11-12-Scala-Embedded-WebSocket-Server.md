---           
layout: post
title: Scala - hosting a websocket server using Atmosphere/Nettosphere
date: 2014-11-12 8:00 AM
updated: 2014-11-12 8:00 AM
comments: false
categories: scala, websockets
readmore: true
---

Websockets are one of the means to push data from a backend application to a web application running in the browser. In this post, we will look at hosting a web socket server within a java/scala application using [Atmosphere/Nettosphere](http://async-io.org). Atmosphere is an asynchronous application development framework that allows development of realtime applications.

Shown below is the scala code that launches a websocket server on localhost at 8080. When the application starts, Atmosphere scans the classpath to locate relevant resources that could be loaded as handlers. In this case, it finds the WebSocketChat class which has the attribute "WebSocketHandlerService". This handler is made to handle incoming websocket requests at the path "/echo".

{%gist krishnabhargav/e0965b0e1ebb5c8067ec %}

For this to work, the build.sbt should be modified to include the dependencies.

{%gist krishnabhargav/b2abf711718c56cb3c5a %}

Once the application is up and running, you will be able to connect to it using regular WebSocket clients. For example, shown below is a simple html file that connects to the websocket we just launched and pings to it.

{%gist krishnabhargav/ac7a5af593a20ce50b86 %}

You can look at the javascript console in the development tools of the browser to look at the output. Note that you have to wait for the socket to be connected before you can actually make the "send()" call. So in our example, I make a send() call from the "onopen" handler of the websocket.

I have some plans on using this to build some webparts based system that you can use to monitor observables remotely....more on it later.