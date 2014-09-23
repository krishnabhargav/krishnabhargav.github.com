---           
layout: post
title: Clojure - Getting started with Web Development
date: 2014-09-22 8:00 AM
updated: 2014-09-22 8:00 AM
comments: false
categories: clojure
readmore: true
---

There are several leiningen templates to create a basic web application in Clojure. But in this post, I will describe how you can create a basic web application that uses Clojure on the backend and ClojureScript on the front-end. The goal is to help understand all the pieces that are involved and how they need to come together so that you can better understand what some of the templates are actually doing it under-the-hood.

Web Development in Clojure is more about using several libraries - so basically pick your liking => too many things to worry about but it is how it is. There are a few single frameworks but this is more about the former. 

So what do we need on the server? On the server-side, for a basic web application, we will need a server to serve static/dynamic content based on the URI for the specific resource. In clojure, web servers are usually "ring-compliant" => which basically means they respect and work with Ring specification. So in all, we will use the following:

- (Ring-based server backend & middleware)[https://github.com/ring-clojure/ring]
- (Http-Kit)[http://http-kit.org] as the server library
- (Compojure)[https://github.com/weavejester/compojure] for routing

Oh, we will use (Cursive)[https://cursiveclojure.com] as our IDE - which is totally awesome.So lets get started. 

We will create a leiningen application called groupie.

{%highlight clojure %}
lein new groupie
{%endhighlight %}

Once the project is created, (import the project into Cursive)[https://cursiveclojure.com/userguide/leiningen.html]. If you look at the project.clj; you will find it to be a very simple clojure application. So lets add dependencies to project.clj for Ring, Http-kit and compojure. You can visit the links to get the latest version numbers. After this, my project.clj looks like

{% gist krishnabhargav/7919baa593b38f0bc8f8/3bae796c5138f31fd4253b02072cc6c638679673 %}

As you modify project.clj, you can refresh it within the IDE by View->Tool Windows->Leiningen and then "refresh" icon or you can associate a keymap to the specific action. This, I believe, is equivalent to run "lein deps" from the command line.

Once the project.clj is updated to include the dependencies, lets add a server to our application. 

The server.clj added to the source folder could be something like shown below.

{% gist krishnabhargav/7a5ef05bcba877099436/6cd5760bd906e8cffc82caf516159fca5580e894 %}

There are four methods in the server.clj

1. a "handler" defined according to the ring spec. The ring spec states a handler receives a ring request and sends back a response which are both simply maps.
2. an "app" which is our real handler, that gets passed to the ring compliant server. Think of this app as a place where we will hook all the required ring middleware. For example, in this app handler, we defined two middleware which basically allows us to serve static content from the "resources\public" directory
3. a dev-main function which uses adds an optional middleware to the "app" that allows the server to reload the clojure files upon changes. This is very handy for us during development (hence the dev-main). Then the resultant handler gets passed to the run-server method of http-kit which launches a web server.
4. an actual -main method which gets executed when you do a "lein run". For this to really work, you will have to define add :main groupie.core to the defproject inside project.clj. Then you can run "lein run 8080 true" where 8080 and true gets passed in as values for port and auto? parameters in the definition. The main either calls run-server without auto reload or delegates the responsibility to -dev-main for auto reload based on the auto? value. 

Add an index.html to "resources\public" folder in the project (create public folder if unavailable). Then lein run passing the parameters. Test if everything works. For index.html, the file will be server and for any other request - you will simply see Hello World message.

So far, we made use ring middleware, a ring compliant web-server using http-kit & we can serve one request and static content. Now we will make use of compojure to define routes & update the app middleware pipeline.

{%gist krishnabhargav/7a5ef05bcba877099436/de2121411036bac2cc55cad5870fc1b74a016706 %}

Now, lets add clojurescript to the project. To do that, we will add clojurescript to the dependencies & add cljsbuild to the :plugins. Then, we define instructions/configuration of the cljsbuild within the defproject. The updated project.clj would look like:

{%gist krishnabhargav/7919baa593b38f0bc8f8/7e2660bb5554402a3eb03c02d87bfc79c833802e %}

Create folder called "cljs" within src (so as to match what we defined in the project.clj). A very simple client.cljs file would be added.

{%gist krishnabhargav/762b8ef59b60056bb8dc/ac7669b3ca00080585d0b1d6b43645429386257a %}

If you remember, earlier we added the middleware "wrap-reload" in the app to support reload of the changes on the server without rebooting the server. It would be nice if we can make clojurescript changes & the browser gets reloaded without having to manually refresh. This exact same behavior is provided via the "figwheel" plugin. Adding support to figwheel is straight-forward.

- We add [figwheel "0.1.4-SNAPSHOT"] to the dependencies (or whatever is current version)
- We added [lein-figwheel "0.1.4-SNAPSHOT"] to the :plugins
- Then we define :figwheel in defproject where you can list the server root to serve static content, the ring handler you want it to use (it uses http-kit, by the way)

Apart from the above, we also adjust the project's :output-dir to indicate that is where we want it to output the generated js files. The project.clj would look like:

{%gist krishnabhargav/7919baa593b38f0bc8f8/45759c4c9e758ba629aa7d43ff751bc9728a8978 %}

Now, to finish the integration of figwheel, we will update the cljs file to include a call to figwheel's watch-and-reload.

{%gist krishnabhargav/762b8ef59b60056bb8dc/a10658dd6ac14cd9026ac138e0557cabbfee3c1e %}

The index html would be updated as shown below. You will have to require the project cljs file in the script snippet.

{%gist krishnabhargav/66a66ed0d7663b4d874f/a5c6088fe31eef6a576e2f7ed3f7564a1d8d61e1 %}



