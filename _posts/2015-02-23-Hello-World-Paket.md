---           
layout: post
title: Using Paket - a very simple introduction
date: 2015-02-23 8:00 AM
updated: 2015-02-23 8:00 AM
comments: false
categories: paket
readmore: true
---

Paket is a dependency manager for .NET and Mono projects. Instead of managing references to assemblies (irrespective of the source - nuget, local or whatever) in the project files; paket provides a (in my opinion) better way to organize these references. More information and documentation for Paket is available at [here](http://fsprojects.github.io/Paket/).

In this post, I will present how I managed to make paket work for me. The purpose of this post is to act as a simple "get started" guide with some more details than what the documentation presents. Please note that I am currently using Mono on OSX but the steps should work as is when using Windows as well.

1. As a first step, you need to get hold of paket.bootstrapper.exe which simply put is that application which gets the latest version of paket.exe. You can get this from the [github release page](https://github.com/fsprojects/Paket/releases/)
2. Make sure the paket.bootstrapper.exe is accessible from the command line. In my case, I created a folder for my project called "hello-paket" and downloaded the paket.bootstrapper.exe in that folder.
3. Create a basic C# project using Xamarin Studio or Visual Studio. I used Xamarin Studio to create a new Console Project. The console output so far is shown below (inspect the output for ls; to be clear)

{%gist krishnabhargav/688e7a2d399edffedde6}

4. Now run "mono paket.bootstrapper.exe" if you are using Mono or just "paket.bootstrapper.exe" if you are on Windows using .NET. This gets the latest paket.exe and saves it locally.
5. Paket needs a paket.dependencies file which contains list of dependencies. In this example project, I want to add xunit as a dependency.
6. In order to add a dependency, you will need a paket.dependencies file to be present. Paket documentation states that you can have one created for you by "mono paket.exe init" or "paket init". It does not work for me on Mono/OSX but its not a big deal. Just create an empty file called "paket.dependencies" using the command "touch paket.dependencies"
7. Assuming the project files, the paket files so far (.exe and .dependencies) are all in the same folder and the command prompt set to the same; now execute the command "mono paket.exe add nuget xunit --interactive". 
8. The "add nuget xunit" options basically makes paket pull xunit from nuget and add it as a dependency. The "--interactive" flag at the end of the command prompts the user for each project in the folder asking if the user wishes to add the reference to the project or not. Enter "Y" to confirm.
9. If you look at the csproj file in text editor; you will notice the csproj be modified to include the xunit as a dependency. So you can start using XUnit classes in the newly created project. Note: I haven't checked within Visual Studio but Xamarin Studio will not show these references in the references view.
10. You can remove a dependency using the same above command but instead of add, it must be remove - "mono paket.exe remove nuget xunit --interactive"

{%gist krishnabhargav/688e7a2d399edffedde6}

That is pretty much it. Refer to the documentation for paket on how it can further help manage dependencies. I like paket and I think it really makes dependency management very simple - but I just thought the "hello world" kind of tutorial was lacking and I hope this simple steps above can guide you through and remove any confusion that you may have (I was initially confused on how exactly to use it).