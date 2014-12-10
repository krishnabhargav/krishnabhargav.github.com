---           
layout: post
title: Windbg Tip - Cannot "break" in Windbg when attached to live process
date: 2014-12-10 8:00 AM
updated: 2014-12-10 8:00 AM
comments: false
categories: windbg, tips
readmore: true
---

I am sure some of you have encountered the issue where Windbg doesn’t break when you attach to a live process (I saw this more on the 64-bit one than the 32-bit one) & the only way seem to be to kill the debugger.

Well, I found something that works for me (and hopefully for you). 

The next time you see the issue, just don’t kill Windbg. Instead go to the Debug menu and uncheck the “Resolve Unqualified Symbols”. 

If that doesn’t help, try unchecking the “Source Mode” as well.  

And then try to break. 
