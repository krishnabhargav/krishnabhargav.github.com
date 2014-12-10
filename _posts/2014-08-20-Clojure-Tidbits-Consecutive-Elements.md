---           
layout: post
title: Clojure Tidbits - Consecutive elements in a sequence?
date: 2014-08-20 8:00 PM
updated: 2014-08-20 8:00 PM
comments: false
categories: clojure
readmore: true
---

I have recently come across an interesting question that need to be solved using functional programming techniques. So I thought, I shall give it a shot in clojure.

#### Question
How do you determine if all the elements in a given sequence are all consecutive elements?

#### General Idea
1. Sort the sequence
2. Compare each element with the next element & it should differ by 1.

#### Exact clojure
{%gist krishnabhargav/cf4c5a382045a40d2cc8 %}

