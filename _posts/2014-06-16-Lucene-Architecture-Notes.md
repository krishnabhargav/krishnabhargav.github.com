---           
layout: post
title: Lucene Architecture Notes
date: 2014-06-16
updated: 2014-06-16
comments: false
categories: architecture notes, lucene
readmore: true
---

Apache Lucene is a very popular search library/framework used by [100's of companies](http://wiki.apache.org/lucene-java/PoweredBy). Since it is used everywhere and that I keep finding a few positions that really interest me but require search experience, I thought the least I could do is look at Lucene, its architecture. Then there are two additional projects on top of Lucene whcih are also popular - Apache Solr and ElasticSearch. The intention is to look at their architecture as well.

To get started, the following is a simple scala program that indexes a document in-memory & then searches in the same document. 
(There is no idomatic Scala API from Lucene project but one from [Gilt exists here](https://github.com/gilt/lib-lucene-sugar) which by the way doesn't work for me.)

{% gist krishnabhargav/bc40de5630c95d4e5f90 %}

First complaint : the API is a mouthful. I used to see API like this a few years back. Yes, it is all extensible and built around abstractions which enables you to extend the aPi elegantly but I always prefer that over all the abstractions should like a simple straight-forward API that we can use. So i tried with lib-lucene-sugar and it did not work. There is infact an [old issue opened in September 2013]() that isn't addressed yet. May be later, I will try and fix the problem and update the repository.

The main components of the above program are:
- IndexWriter - used to create index and add documents to the index
- Directory - represents a location where index or data is stored
- Analyzer - it is responsible to extracting tokens out of text
- Document - collection of fields with information about the document 
- Field - types are keyword, unindexed, unstored & text

Lucene understands only text so if you want to index PDF or Word documents, you need to convert them into text. 