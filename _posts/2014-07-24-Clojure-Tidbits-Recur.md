---           
layout: post
title: Tail Recursion in Scala and Clojure
date: 2014-07-07 8:00 PM
updated: 2014-07-07 8:00 PM
comments: false
categories: clojure
readmore: true
---

In this post, we will look at tail recursion and how it is supported in Scala and Clojure. We will start with some simple examples in Scala followed by examples on how "recur" - a special form in clojure that is used as a way around the lack of support for Tail Call Optimizations in JVM.

#### What is a tail call optimization?
If the final action of a method is calling a method, the said call is referred to as a tail call. As you may already know each method call will require allocation of space on the stack which shall be used by the code within the method call. So if you have a function that calls another function that calls another and so on ... as the call depth increases, the stack continues to grow and eventually run out of the stack space => this is commonly refered to as stack overflow and such errors are usually manifested as StackoverflowException.

The following Scala function is a very simple implementation for a factorial method.

{%gist krishnabhargav/8e07a7ea4b5be6b789fa %}

Calling the function with large numbers will definitely result in a stackoverflow exception. So how can we support recursion without having to worry about stack overflow? That's where compiler optimizations for tail calls comes into play. Some compilers can detect tail calls & generate instructions such that stack space for the current call can be reclaimed and 

In the line 2, you can see that the last action in the method is not really calling the method itself but the action is 

1. call the method recursively with the number decremented by 1
2. multiply the result to the current number

OR 

1. return the number itself

Since the last action really is multiplying the number with the result of the recursive call, the call is not a tail call. So this demands that the current stack space be present so that the final computation can be performed. So the Tail call Optimization cannot be performed in this case (since there is no tail call).

JVM lacks tail call optimization but the Scala compiler can perform tail call optimization in some cases by converting the function calls into loops. In order for the Scala compiler to do that, there should be a tail call present. So writing the factorial method as a tail-call variant is shown below. The key here is that you will have to eliminate the need to maintain the local accumulator but instead pass the accumulator to the function call itself as a parameter.

{%gist krishnabhargav/74e9f8bbb2f707fd8c78 %}

You can see that the internalFactorial method is implemented in a tail recursive manner. Now when the scala compiler sees the @tailrec on a function definition, it will check to see if the function was tail call optimized and if not, it throws an error for the developer to check the implementation. Note that with or without the @tailrec annotation, the function shall still be optimized.

Support for recursion is a key piece for a language to support functional programming constructs and tail recursion is a must for anything practical. Having looked at the Scala's tail recursion, lets turn our attention to clojure - for the same example.

#### "recur" - special form in Clojure
"recur" is a special form in clojure that can best be understood examples. Let's start with a simple example from the awesome Joy of Clojure - a count down function. The count-down function receives a number and while counting down towards 0, it prints the number. The following example is not right out of the book, but modified to present how recur works.

{%gist krishnabhargav/10c39c276ea0f2c63cdf %}

"recur" in the first example will result in the current method being called again but the arguments will now bind to what was passed to recur. Since the recur takes the execution right back to the beginning of the method, the "Method Started" will be printed n times.

In the second version, we use a loop...recur. Here, recur would not take the execution right back to the beginning of the method instead, it takes the execution back to where loop began. So the "Method Started" message would be printed only once.

The recur is the clojure's way of supporting tail recursive calls. The following shows how factorial may be implemented in Clojure.

{%gist krishnabhargav/ae68e7c49e40b4c616cb %}

I hope these examples shows how to use recur, loop...recur.