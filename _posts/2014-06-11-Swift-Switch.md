---           
layout: post
title: Swift language - Switch statement
date: 2014-06-11
updated: 2014-06-11
comments: false
categories: OSX, Swift, iOS development
readmore: true
---

In my marathon effort to catch up with all the things that interest me, I am looking at iOS development and now that Apple has announced Swift language, I was playing with it in the XCode Playground. In this post, I will capture some notes associated with Swift's switch statement. Hopefully, it will be a breathing document that I will add code examples as I see.

Shown below is a simple enum declared in Swift. Notice that you have a "case" for each value in the Enum. 

{%highlight csharp %}
enum FriendType {
    case Acquantance
    case None
    case Best
}
{%endhighlight %}

Now we are looking at the usage of the enum in a switch statement.

{%highlight csharp %}
var myFriend = FriendType.Best

switch myFriend {
    case .Acquantance:
        println("Whats up!!")
    case .None:
        println("Hello")
    case .Best:
        println("Whats up bro!")
}
{%endhighlight %}

If you try the above code, it would just work and the compiler would be just happy. But let us add one more case to the enum and leave the code as is.

{%highlight csharp %}
enum FriendType {
    case Acquantance
    case None
    case Best
	case Good
}
{%endhighlight %}

Then your usage program will error out as:
	error: switch must be exhaustive, consider adding a default clause

So what is the exhaustiveness of the switch. I will explain as I understood - your switch should cover all the possible cases. Previously, before you added "case Good", your switch had all the cases covered but not any more. So either you make your switch cover all cases or add a "default:" case.

{%highlight csharp %}
switch myFriend {
    case .Acquantance:
        println("Whats up!!")
    case .None:
        println("Hello")
    case .Best:
        println("Whats up bro!")
    default:
        println("Whatever!");
}
{%endhighlight %}

So that is pretty good. But "default" has its disadvantanges in a real application. Adding a default will make you unaware of any changes to the enums that you are consuming & at times it may be good to catch these at compile time. 

SideNote: If this were in C#, we did have more confusion. For example, I have some switch code in A.Dll and enum in B.DLL. For some patch, I changed the contents of B.Dll and added a new value to the enumeration I was using. So would the switch case still work or throw error during JIT?

I think iOS/swift code is statically linked so it might not be a problem. But at the sametime, I dont know if there exists a notion of "patches" as in only those DLL that have changed. As I dig more into the environment, I will find more and I will be sure to take some notes. Now back to switch.

enum supports range values. The example for the WWDC'14 talk on basic swift.

{%highlight csharp %}
enum Status {
    case OnTime
    case Delayed(minutes:Int)
}
{%endhighlight %}

The following usage will result in the same exhaustiveness error.

{%highlight csharp %}
func doSomething(status: Status) {
    switch (status) {
    case .OnTime:
        println("Train is ontime")
    case .Delayed(1..10):
        println("Just a few minutes more")
    }
}

doSomething(Status.Delayed(minutes:15))
{%endhighlight %}

So how to fix this? Make this switch exhaustive. So add a default or just add a case for .Delayed

{%highlight csharp %}
func doSomething(status: Status) {
    switch (status) {
    case .OnTime:
        println("Train is ontime")
    case .Delayed(1..10):
        println("Just a few minutes more")
	//Add base one
    case .Delayed:
        println("Just delayed")
    }
}
{%endhighlight %}

So whether to fix it this way, or to add a default case should be used upon your best judgement and experience. If it were me, I would try to be as specific as possible in my code and there will be times when you do not have an option but to use default. For example,

{%highlight csharp %}

let myname = "Krishna"

switch myname {
case "Krishna":
    println("hey, Krishna!");
//case _:
default:
    println("hello, \(myname)");
}
{%endhighlight %}

So you can use "case _:" or a "default:". From what I experimented, there doesnt seem to be a difference. "_" is a wildcard or anonymous variable in Swift.