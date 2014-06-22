---           
layout: post
title: Simple Code!
date: 2014-06-22 8:00 PM
updated: 2014-06-22 8:00 PM
comments: false
categories: rants, essays
readmore: true
---

I lead a small team of engineers at work and I always bring up the topic of "writing good code" to my team during our regular discussions. So what does good code mean? I will try to write down my thoughts on the subject here.

#### Simple code
Good code is simple - simple to write, simple to review, simple to extend and simple to maintain. When I refer to simple, I mean in the context of complexity (not time/space complexity but layman complexity). So simplicity in the context of building blocks of a C# program would be

- A class is simple if it serves one purpose - has one responsbility.
- A method is simple if it does one thing as well - single responsibility and there is a structure to what it does. Reviewing a method should clearly tell the reviewer which portion of the feature the method supports.

But there is more to it than a single responsbility. Let me give this a shot.

When we write a class, 

- we have to do something in the class.
- we need some dependencies to be doing that something.
- we need to expose a few things so that it may be used by some other components.

So in the context of the above, a simple class shall

- Serve one purpose - single responsibility
- Dependencies 
	- received the same way
	- does not modify/mutate the dependencies
- What the class exposes should depend on what purpose it solves. It should not expose anything more than detailed than the abstraction it solves.
- Simple to use

#### Simple Code - Dependencies
It is hard for me to explain these concepts in a blogpost but I will give it a shot. Lets look at the dependencies and what I meant about them.

{%highlight csharp %}
	
	public class SuperSlicer {
		
		private readonly ISlicer _slicer;
		
		private readonly IContainer _container;
		
		public SuperSlicer(IContainer container) {
			_container = container;
			_slicer = container.Resolve<ISlicer>();
		}
		
		public void Slice(){
			var powerTool = _container.Resolve<IPowerTool>();
			_slicer.With(powerTool).Slice();
		}
	}
	
{%endhighlight %}

So the above class is actually simple to read but look at the dependencies. If I give this class for someone to use, sure it is simple to use - just pass in the container and let the class resolve whatever it needs. Right? This is just so wrong on so many ways and the biggest of them is - your code should not annoy the other developer you are expecting to use. 

How the fcuk would he know that the container you pass should have registrations for ISlicer, IPowerTool? 

But it is so simple, look at the code once & you will get it. Duh! 

So lets say for a second, that I get it. I get it that it will need ISlicer and IPowerTool. But that happens to be for now. There is nothing in the code that says in the future, the code you write would still work. So lets say 6 months, later because some moron from the customer team wants to also have the super slicer use ISaw a next generation technology implemented for that realease. So your slicer will change to

{%highlight csharp %}
	
	public void Slice() {
		var powerTool = _container.Resolve<IPowerTool>();
		var sawTool = _container.Resolve<ISawTool>();
		_slicer.With(powerTool).With(sawTool).Slice();
	}
	
{%endhighlight %}

That was a simple change - so our simple class was easy to maintain? The answer is no. It is important to realize that large systems have hundreds of products and a lot of features. So the components that were using your SuperSlicer would not work anymore because that component may not have registered ISawTool to the dependency container.

But then you are supposed to have unit tests to catch situations like these? Right? Again wrong - when you have unit tests, you would unit test the SuperSlicer and your tests will inject the required components properly and the test for SuperSlicer would pass. And then say component X has a class PowerTools that has uses this SuperSlicer. Unit Test for PowerTools may not be using the instance directly and could have been mocked. So the real changes aren't caught in those unit tests either. 

So at runtime, when the PowerTools component is used - it would fail. So if the developer who added the Saw Tool did not know about Power Tools and haven't tested it, you have a defect there! Worse, it could be a crash as most DI containers throw an exception of some form for a failed registration. 

This is exactly how you keep accumulating the debt when something as simple as dependencies are not consumed properly. Remember that the code you write in professional systems (professional - used commercially) gets harder and harder as time passes on - so making changes to is also hard as time progresses and eventually you will end up with a buggy system.

So how do I recommend consuming the dependencies?

{%highlight csharp %}
	public class SuperSlicer {
		
		private readonly ISlicer _slicer;
		
		private readonly IPowerTool _powerTool;
		
		public SuperSlicer(ISlicer slicer, IPowerTool powerTool) {
			_slicer = slicer;
			_powerTool = powerTool;
		}
		
		public void Slice(){
			_slicer.WithPower(_powerTool).Slice();
		}
	}
{%endhighlight %}

So when the SawTool must be integrated, the code would change to:

{%highlight csharp %}
		
		public SuperSlicer(ISlicer slicer, IPowerTool powerTool, ISawTool sawTool) {
			_slicer = slicer;
			_powerTool = powerTool;
			_sawTool = sawTool;
		}
		
		public void Slice(){
			_slicer.With(_powerTool).With(_sawTool).Slice();
		}
{%endhighlight %}

So why is this way good? I am not sure if it is a good way that solves all problems but it definitely solves the two major problems - usability of the class and modifications induced defects.

As for the usability - if you think the previous usage was simple, this would be even better. You know exactly what you need to pass in order to use a superslicer - don't you?

As for the modifications - you can see that adding a dependency required modification to the constructor. So all those consumers who instantiates SuperSlicer will no longer compile unless the usage is fixed. So the issue is caught at compile time than at runtime.

But I am arguing that the purpose of passing the container was that the tool was registered transient (new instance created for each resolve call) and each Slice() should use a new instance of Power Tool and new instance of the Saw Tool. 

Well, my friend, that argument is not valid either. Firstly, it is great that DI containers suport lifetime during registrations but one single change to the lifetime may induce defects to your system. And you will be hunting down weird problems. Secondly, I prefer the lifetime to be evident from my usage. So I propose the code to change to something like

{%highlight csharp %}
		
		public SuperSlicer(ISlicerFactory slicerFactory, IPowerToolFactory powerToolFactory) {
			_slicerFactory = slicerFactory;
			_powerToolFactory = powerToolFactory;
		}
		
		public void Slice(){
			_slicer.With(_powerToolFactory.NewTool())
				   .With(_slicerFactory.NewTool()).Slice();
		}
{%endhighlight}

Now you can pass the appropriate factory. It can still meet our previous requirements and solves your previous argument about new tool per invocation.

But if you think I am not qualified enough for you to list, I will quote an apt principle that you can read up. But remember that there are so many ways all these amazing pricinples that OO pundits have come up with can be applied. For example, [Law of Demeter or principle of least knowledge](http://en.wikipedia.org/wiki/Law_of_Demeter). I can argue that you should know as minimal as possible about other units. So know about a IContainer breaks/violates/rapes the Law of Demeter. 

#### Simple Code - Abstractions
A simple class does not leak its internals. A simple class provides functionality through abstractions and the abstractions can be as simple as some methods. Again there can be a lot of rules about writing good methods but my point here is that 

- you should contribute to enable Law of Demeter. By that, I mean you should not expose more information than you need to. If you dont expose more information than you need, the consumer of you class cannot access it - so in a way you contribute to help maintain the Law of Demeter.
- you should not leak abstractions. Lets say, you work with 4 types and for sake of convenience you wrapped these types into a struct. So do you really need to change your method call to receive a struct of this type as opposed to the 4 arguments? 
	- The common argument that I hear about this subject is that it is easier to have a data type that wraps these 4,5,6 or 10 arguments and have the user pass it. Firstly, this is wrong because you lost compile time safety. You are basically allowing the user to miss not passing some arguments. And by the way, if you have 10 argument to your method - you are doing it wrong. Your function is not simple anymore and I dont even want to talk to you - you are not my friend & if you are going to argue for this - you are dead to me.
	- [The original leaky abstractions talks about your abstraction unable to hide the underlying details even though that was the intention of your abstraction](http://www.joelonsoftware.com/articles/LeakyAbstractions.html).
	- The one I am refering to here is, do not model classes or data types to "abstract" the inputs or outputs. Simple inputs and outputs have their benefits and proper care should be taken before such radical refactorings.

#### Simple Code - Ownership
Are you that kind of person who borrows a friends car and shows off to the world as if it is your car? What a fking show-off! Are you going to take my car to the service center? Are you going to pay for my insurance? Get your own fking car and show off, not with my car?

The same thing applies to the dependencies you receive. The dependencies you receive are like a car that you borrow from your friend. It is impolite to show off as if you own it, it is impolite to scratch the car or destroy it and it is definitely impolite to lend it to others.

Ownership of components largely depends on how the system is designed. One tip for this - think of this way - each and every component should have an owner. It is the owner who can

- pass around the component to some other components to use it
- make modifications to the component
- destory the component

Everyone else can only use that component. And remember, when you borrow my car, it does not matter how you gave it to others - you may wrap it in a tin foil and lend it - it is still my car. I am saying this now because, you should can always create your own class and have a property return the dependency you consumed. So your new class is the Tin foil here. Like i said, it doesnt matter.

So long story short - ownership should clearly be defined at design level and attention should be paid when reviewing the code as one assignment state is more than enough to lose ownership. So there is another lesson here - just because you have an awesome car doesn't mean you can just lend it to anyone. Samething with the components you own - if you dont pass it on to others, then the chances of it leaking will also be reduced. 

I am sure this article does not have the proper structure that great authors can provide but I believe the points I made are totally valid and violating any of the points (and probably more that I have missed) implies your code is not simple anymore and to me it is not good code.

