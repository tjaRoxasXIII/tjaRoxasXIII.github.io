---
layout: post
title:      "Routing - Where do I go from here?"
date:       2020-10-09 16:40:47 +0000
permalink:  routing_-_where_do_i_go_from_here
---


I like to be prepared.  It's nice to know what to expect so you aren't ever caught off guard, so with the Sinatra Project, I was eager to take a look at the requirements and get started a little early on the planning portion.  I really didn't need a ton of time to come up with an idea once I began reading through all of the instructions since a previous job of mine gave me a pretty good idea of what I could work with.  We used to have this website which our company hosted internally.  Employees could log in, share messages, and view basic information about other employees or the company itself.  It wasn't incredibly impressive, but it seemed to be a good place to start and that's what I decided to build.

The actual act of building this project out was very difficult for me at first.   Going through the curriculum, I quickly learned that you either understood concepts or you didn't.  And up until the weekend before starting  the project, I definitely didn't.

So I want to share my thoughts about routing and some of my confusion around their ideas and concepts.  We learned about .erb files and how they contain the HTML formatting and Ruby code to display our desired information on a webpage.  Okay, I was following, but then we started talking about things like GET, POST, PATCH, and other commands and suddenly I was lost.

In my brief prior experience with websites, you would typically build them as individual files and navigate between them using links pointing somewhere within your file structure.  I had to eventually learn that now those routes would be handled within our application controller and that those .erb files could be called from within the controller itself.  Confused yet?  If not, you are already ahead of where I was.

It took a lot of reading and playing around with before the concept finally clicked, but once it finally did, everything became so much simpler and I now realized how dynamic and flexible my webapp could be.  So many checks and routing decisions could be made conditional upon whether a user was signed in or if that user had the proper permission to navigate to a specific location.  Routing decisions made my simple app flow seamlessly and it also provided the security I had never achieved before this point.  

Maybe it wasn't a difficult concept for some, but for me, this project really added clarity on how things are expected to connect to provide a great user experience.  I finally understand the purpose of a Controller within an MVC application.
