---           
layout: post
title: Clojure Tidbits - Learning from reviews
date: 2014-09-13 8:00 AM
updated: 2014-09-13 8:00 AM
comments: false
categories: clojure
group: fp
readmore: true
---

Recently, I have been hanging out at #clojure, talking to experts in clojure about various things and using that to learn something practical about clojure. So far, the experience has been invaluable. I believe I learnt at a much better rate from my two days at #clojure than I did this year through various sources of clojure. Anyway, I wanted to simply model data "the clojure way" & sought out some help from the experts. In this post, I will present the code I came up with and then the comments I received followed by the revised code, etc.

#### What am I trying to model?
Say, I want to model a vacation trip. A trip can be described by having a name, a description, a start date, an end date & a group of people participating in the trip. 

#### How to go about this?
After some discussions and learning a few bits about resisting the urge to model data in an OO way, the gist of the advice which I agree with is - start with basic structures such as lists, maps, vectors & sets. Model your data using those. Use prismatic/schema, if required, to validate the shape of the date you receive. As you progress, you can replace maps with records as all operations on maps are supported on records.

#### Initial code
I took the advice and modeled my trip using maps. But since there is no class definition or anything like that, I started with a "default trip" which is supposed to provide a "shape" for reference along with the default values. Then, I gave helper methods from which you can build your own trip. That looked something like:

{%gist krishnabhargav/2a92da3f5f161c7da197/1655566c434d79353b0d3000650dfe8fb16893e5 %}

##### Comments about the revision
- (assoc trip :people (into (:people trip) persons)) should preferably be replaced with a update-in/assoc-in. So basically, it should change to something like (update-in trip [:people] into persons)
    - If conj has "into" when you have a sequence of elements for adding into a sequence, disj has what? So the answer was to use clojure.set/difference.
- name, description, start-date, end-date helper methods needlessly abstract. Instead, it is preferred to expose the keywords directly as "assoc" already abstracts away the structure for you
- default-trip makes use of a t/today which is a static variable. As an application keeps running for days/months, the value computed by default-trip will be what it was, when originally called. Instead, it should be made a function which makes it clear to that caller that some computation may happen when invoked. 
    - For the above comment, I raised the question about the function call no longer being idempotent because each time you call, the answer may differ. The response was that the t/today itself isn't idempotent so either we provide default dates as nothing or compromise here as it is a decision that I took for making the start date default to today.
- It is better to place usage examples or same file unit tests in a (comment ...) block as this will still allow your editor to syntax highlight and other such support.

#### Revision #1
After I applied the above comments, I have removed the helper methods for start-date, end-date, name & description. 

{%gist krishnabhargav/2a92da3f5f161c7da197/43c1c893a89ad5bc3b72baf593d366ce9b716381 %}

##### Comments about this revision
- assoc is varargs so in the usage example, you dont need to call assoc for each and every key-vaue pair.
- Instead of directly using clojure.set/difference, it is prefered to require clojure.set as set and do a set/difference.

#### Final code
After making above small changes, I ended with:

{%gist krishnabhargav/2a92da3f5f161c7da197/e5f6ed4c9f47da01ea785f357e65a7feb97a4eec %}

As you can see, there is only default data that describes the shape of the data & a couple of helper methods to åssist with nested structures. We have a trip!

#### A few more things
- (macroexpand-1)/(macroexpand) is a good way to understand "weird clojure" such as thread-first macro usages.
- In order to use ->, say on functions that uses a map, the map should be the first parameter. (add-person [map person]) & not (add-person [person map]). In the later case, you may hve to use ->> which is not recommended.
- In order to determine which clojure version, use *clojure-version*.

Overall, its highly recommended for those trying to learn clojure to join the #clojure on webchat.freenode.net & ask away your questions. The folks over there are very helpful (justin_smith, tbaldridge, ToxicFrog and a lot of other very helpful folks). Looking forward to learn more ...
