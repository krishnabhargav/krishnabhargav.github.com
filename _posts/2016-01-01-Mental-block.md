---           
layout: post
title: Mental block on personal projects & lessons learned
date: 2016-01-01 8:00 AM
updated: 2015-01-01 8:00 AM
comments: false
categories: personal
readmore: true
---

I have a big question for which I did try to search around and get some guidance. The question is something like ..

```
I am a software engineer by profession. And I would like to think a good one. I managed teams in the past that designed and built some incredible systems 
under very tight schedules. Not just as a leader, but as an individual contributor I wrote some really cool software.

Now I want to take the same level of expertise and apply it to my own ideas. And I do have some free time to apply myself.. but I cannot bring myself to do it.
There is always something unknown that prevents me from applying myself.
```

Now this is a serious enough problem for me. I do not really have an issue with starting new projects/tasks or seeing activities to completion at work. But when it comes to my
own personal startup ideas or just some web apps, I am not productive - heck, I don't even get started well.

Last year, I did have one successful start though. I wanted to build a cricket scoring application while enabling a social network of cricket lovers.
For something that I stopped half-way, it went considerably well. I had real progress around it and if not a successful platform that came out of it, I have some lessons to share.

1. Screw perfection and get your hands dirty.
    - Nothing is perfect from the beginning. So do not wait to start it "perfectly".
    - In the past project ideas, I spent a lot of time thinking through - so much time just thinking that I usually do not do anything else but think!
    - With the cricketty app, I had a rough idea of what I want and just created a gitlab project, drew some sketches of expectations/requirements and got things rolling.
2. Do not approach this project as how you do it at work.
    - Projects we tackle at our job has different driving factors and more importantly has excellent peer-support. There is usually a team of people vested in making it work and ofcourse its their job.
    - For personal projects, you will not have such a support system. What I observed is that it is often better to just have a rough idea on what you want, get started working on it.
3. Be clear of your goals.
    - Learning a different technology and applying it to execute your idea is not going to work. A lot of my projects failed for the same reason. 
    In the past, my reasoning was that I will pick some technology I want to learn and use that for this project. So if my startup idea fails, I will still end up with new skills.
    - Based on my own experience, this is not realistic goal. And yes, I did learn Elixir, Clojure, Go and few other things while I attempted to execute my ideas - none of the ideas were ever executed properly.
    - So in the future, I would rather just make the working project itself as a goal and not learning new skills.
4. Just get started
    - A lot of times, all you need is to just get started. Once you get started, things will flow naturally and you shall find your rythm.
    - So do not waste time overdoing the initial idea refinement, splitting into tasks or long term discussions on where you can take your idea. 
    Just understand what you want for your MVP and get your things going.
5. But do plan ahead
    - While most of the advice so far has been to just get things going - I still think there is some value around what you want to accomplish in your first attempt or a prototype phase.
    - For example: for a new idea that I want to explore, I still find it really helpful to breakdown what I want into smaller goals - all when accomplished gives me an MVP.
    - So if I were to build a new web application, I list down all the pages that will provide a flow, all the backend actions to be supported, any additional infrastructure support required?
6. Understand your limits
    - A lot of time, I get distracted trying to accomplish or visualize what my application would look like - a clean UI with professional look-and-feel. 
    But guess what, I am not a UI designer and I am fairly certain I am not going to end up with polished apps that can compete with a Dropbox.com or Medium.com
    - Same deal with your architecture. Just build something that will deliver some initial value - then you can worry about architecture, scalability and even security. 
    - Most of the times, when you are working by yourself or with a couple of friends and desire to get the best possible application done - you will not be able to get that.
    - For example, in order to get my cricketty social app working as I "desired", it would take me atleast 6-8 months assuming I work 4 hrs/day. 
    For something that is done part-time and with no visible benefits, it is really hard to stay focused and dedicated for that timeframe. So I reduced the scope of the project, came to an understanding that I cannot
    build a webapp, a mobile app and a full backend all-by-myself.
    - So understand your limits, first move fast to get a working prototype. Once it helps validate your idea, you could take it to the next stage or rearchitect it again.
7. Have a team with similar aspiration levels
    - I love my friends but they are an awful folk to collaborate with. They have the desire and the passion to get something of our own going but lack the drive to prioritize our project over social life.
    - And it isn't always the others; I myself have prioritized hanging out with friends, going to movies over our "planned/committed" work.
    - But more often that not, having a team with the same level of expectations around contributions and priorities will really help. 
8. Pick a technology that you know the best. It will help you get going much faster, you will be more productive in a comfortable setting than new environments.
9. Set short goals. "MVP done by next month last Friday" isn't really a goal that will help you. Instead, set daily or semi-weekly goals for specific tasks. As you execute a goal, you will find yourself learning new things, wanting to add features. Just make a note of those and focus on the task at hand. Once a goal is accomplished, you can revisit your notes to add new goals, reprioritize or just move to the next goal (which is almost always better!) 
    
Again, these lessons are not from my past success. Rather, they are from my failures to get my ideas implemented and running. And it isn't like I learnt my lesson and have successfully applied these to create something wonderful.
I am just making an attempt to log these, albeit a little unorganized to teach myself to get rid of this mental block of getting things done.

So for any upcoming project that I have planned for this year, this is what I shall be doing.

1. I have a general idea of what I want to provide - a mission statement. 
```
Want to provide a web publishing platform focused purely on Telugu content. We want to provide tools for people to author content in Telugu and provide editorial support for those in need of it,
and once the content passes our quality board, we will publish it to a social network of Telugu language enthusiasts.
```
2. Now I identify what makes a prototype - MVP. So basically, I believe there are a few flows such as registration, user login, submit a post, commit a post, display the posts, etc.
Based on these flows, I list all the pages/views, their corresponding backend actions that will be needed to stitch these flows together. Each of these views themselves will be one task.
So I have say around 15-16 tasks to get things rolling.
3. Create a gitlab repository. (Gitlab is like Github but permits private projects). Create all these tasks with some descriptions inside Gitlab.
4. For this project, I haven't really decided what my data model is and may most likely reuse what I built for cricketty as it is generic enough. 
5. Technology choices - I have two ways to go about this. 
    1. An SPA model where frontend becomes the key and backend is a set of services. 
    2. Regular website where frontend is driven by server-generated HTML. Backend becomes the key factor.
   Backend choices: Elixir + Phoenix, F# + Suave/Nancy/MVC, Clojure + Ring.
   Frontend choices: I want to use Elm if required.
   But in light of the above lessons - I will stick to using Elixir + Phoenix and make a regular website. I have some pieces of backend that I wrote for cricketty that I can reuse.
6. Once I have the tasks, I want to estimate each task and set a deadline. That way, I can predict when I will have something working.

Again, a lot of what I wrote here is just for my own notes. If you are someone like me, who finds yourself to be stuck at getting your personal projects done, feel free to share.


    