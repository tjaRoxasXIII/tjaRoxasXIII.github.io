---
layout: post
title:      "Hey Javascript, Have Some Class"
date:       2020-12-09 18:19:15 +0000
permalink:  hey_javascript_have_some_class
---


Javascript, very similar to most other programming languages, has a way to define, create, and operate on an instance through what we refer to as a Class.  This really helps to focus the scope of the functions we use  to modify data, images, or just about any content within a single page of an application.  I would also consider the ability to organize our data like this adds the benefit of mimicking the style of Ruby for example.  We'll go over just a few ways by using some examples from my own project.

When I first started on my project, I built all kinds of functions that spanned across multiple files and quickly started to find that if I wasn't careful, I could call nearly any function from any file on any object that I wanted.  Javascript is a little bit like a loose cannon in that way.  So even though I had scripts that only pertained to my Users, I could call them without that context and Javascript would go ahead and assume that I knew best and give it a go.  This definitely has its uses and the ability for code to be that accessible is very convenient, it isn't always what we want.  Let's take my a closer look at my User class.

```
class User {
// We'll ignore the constructor class

  savePlayer() {
  //send a fetch POST request to http://localhost:3000/users/${this.id}
	
	fetch(CURRENTPLAYER_URL, {
            method: "PATCH",
            headers: {
                "Content-Type": "application/json",
                "Accept": "application/json"
            },
            body: JSON.stringify({
                username: currentPlayer.username,
                top_score: currentPlayer.top_score
            })
        })
  }
}
```

In the above example, we have the User class and a single function.  The important thing to look at is the URL being posted to.  That last little bit ${this.id} specifically refers to an instance of our User class.  Later on in the body section, Javascript knows that we are dealing with a specific player and can pull that specific id from the given context.  Otherwise, we'd have to explicitly define which user we were dealing with in the URL.

Being able to call methods on an instance of your class can removes the need to pass an object in as an argument for your function and limits the function to only running for the associated instance.  It's a small feature, but an important one for trouble-shooting to make sure the right data is being injected where it needs to be.
