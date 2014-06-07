---           
layout: post
title: Notes from Architecture of Open Source Applications - Asterisk
date: 2014-06-07 14:46:29 UTC
updated: 2014-06-07 14:46:29 UTC
comments: false
categories: Architecture notes
readmore: true
---

There is no other means of getting better at architecting software than reading about existing Software Architecture. And there is an excellent resource in the form of ["The Architecture of Open Source Applications"](http://aosabook.org/en/index.html). In this post, I take some notes from studying each application explained. To start off, the first chapter is about [Asterisk]("http://www.asterisk.org") and here is my notes for future reference.

#### Important points
- Asterisk supports a long of technologies for making/receiving phone calls.
- A phone calls into * system, a channel is established to represent the call.
- Phone A calls Phone B -> two endpoints in the Asterisk system. The channels are said to be briged (connected) for passing media between them.
- Asterisk has to understand all media it is letting through.
- Generic Bridge : media passes through Asterisk
- Native Bridge : if media transport technology is the same, connecting the channels if feasible (and more efficient)
- communication during a call is via frames (ast_frame) - media or signalling frames. Frametypes are statically defined (include/asterisk/frame.h)
- Component Abstractions  
	- highly modular application
	- main hooks up interfaces, concrete implementations are registered at runtime.
	- by default - all modules in modules directory are loaded
	- configuration allows specific modules to be loaded 
	- when loaded, module registers its implementation for an abstraction interface
	- Channel Drivers
		- channel api provides telephony protocol abstraction
		- a driver is required to translate between Asterisk protocol and the specific telephony protocol
		- @interface *ast_channel_tech* 
			- @method *requester* : factory method to create *ast_channel* (abstract channel layer). 
		- ast_channel defers technology specific handling to the driver it holds reference to.
	- Dialplan applications
		- /etc/asterisk/extensions.conf
		- set up call routing using dialplan
		- made of a set of call rules called extensions
		- extensions are list of applications registered as modules are loaded
		- applications can access internal Asterisk API to interact with channels
		- example: Playback, Voicemail
		- applications can work together
		- provides scripting interfaces for those cases where dialplan language is not sufficient
		- dialed extension number will be used to find appropriate application
	- Dialplan functions
		- dialplan apps register these functions that can receive and send information from the dialplan
		- example: adding call to the calllogs 
	- Codec Translators
		- for VOIP calls, each channel may have audio encoded with different technologies
		- Asterisk tries to have the two channels in call use a same codec so that no transcoding is required, which is not always possible
		- sometimes, it has to transcode for some signal processing or likes of that.
		- PSTN may not support codec negotation but multitude of IP protocols does. (eg: SIP)
		- Codec translator modules implement *ast_translator* interface. Knows nothing about a phone call, only knows to convert media from one format to another
- Threads
	- POSIX Threads API wrapped for debugging convenience
	- Threads can be classified as a Network Monitor Thread or a Channel Thread
	- Network Monitor Thread
	   - exists in all major channel drivers
	   - monitors the network for incoming calls or other requests
	   - handles initial connection setup (auth, validation, etc)
	   - upon call setup, creates the ast_channel
	- Channel Thread
	   - created for every inbound channel
	   - dialplan applications execute in the conext of channel thread
	   - lifetime of ast_channel is controlled by channel thread
- The description also gives two scenario which explains the code flow (voicemail and bridged connections).
- Asterisk doesnt scale well but community is working on Scalable Communications Framework to address scalability concerns.

In short, it was a good read. I cannot claim that I now no about Asterisk very well but I can say that I got a good idea on the architecture of Asterisk and I could find some common architecture paradigms such as Main application working as only bootstrapper, proper responsibility assignment such as component within main hooks up the interfaces, codec translators not knowning about anything else but format conversion, loading modules at runtime from a directory (I made a similar decision for its simplicity), etc.

I am looking forward to reading the other chapters and making similar notes. I hope the rest of the book is as awesome read as this chapter on Asterisk. Excellently written by [Russel Bryant](http://blog.russellbryant.net).
