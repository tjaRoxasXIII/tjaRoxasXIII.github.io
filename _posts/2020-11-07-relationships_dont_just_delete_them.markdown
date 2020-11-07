---
layout: post
title:      "Relationships: Don't just delete them "
date:       2020-11-07 16:02:32 +0000
permalink:  relationships_dont_just_delete_them
---


Project building has always been my favorite part about the Flatiron curriculum.  It's exciting to take an idea and shape it into whatever you'd like it to be using newly acquired knowledge.  Building in Rails, however, has been the hardest task to date.  It's a powerful environment but even a new and empty project comes with so many files and folders compared to previous projects, so it's easy to get a little lost in the madness.  Nevertheless, I set off to build a simple IT Ticketing system to meet my requirements.  I won't explain the entire project, but I do want to break down 3 of my models that helped me realize just how easy it is to break your environment.

Per requirements, here are the relationships between my models:

User Model:
```has_many :tickets
has_many :issue_types, through: :tickets```

IssueType Model:
```has_many :tickets
has_many :users, through: :tickets```

Ticket Model:
```belongs_to :user
belongs_to :superuser, class_name: "User", foreign_key: :user_admin_id, optional: true
belongs_to :issue_type```

After configuring my routes and making sure my objects all behaved individually the way I wanted them to, I built out some 'Show' and 'Index' pages that displayed some basic information about my tickets using the helpful relationships previously defined.  Here's just a simple example I'll use to illustrate the problem I've come to find with relationships:

My ticket object contains a field for an ```issue_type_id``` to help attach it to an IssueType object.  So when we display the ticket on its show page, we can call upon the method created by this relationship like so and specifically pull out information about a specific ticket's issue.

```@ticket.issue_type.name => "Hardware Issue"```

So far everything is normal, right?  But what happens when someone decides they don't necessarily like the options I've provided for my IssueType categories?  I did build in the function to create your own and (here's the dangerous part) a User could delete any of the existing IssueType objects if they wanted to.  But what's the big deal and who even cares?

Well the ticket definitely does.  In fact, once I deleted a category that belonged to one of my tickets and afterwords, nothing functioned properly at all.  Everything within my code now relied on a relationship that no longer existed.  So from there, I was continuously greeted with the ```no object for nil.class``` error and none of my pages would load.  So after dropping the database and getting back to where I was before, some design decisions needed to be made.

These relationships are strong and powerful tools in regards to object associations, but they are also heavily dependant upon each other and depending on how you build them out, they don't like to be destroyed.  I've come across this a few times and have found that there are some ways to work around this to make sure you don't accidentally create a big mess by playing around with your data.  

In a past project, if I were going to destroy one of my Users and still wanted to be able to see their posts on a message board, I would first assign a different object to the post so it could still have something to reference on pages where that would be necessary.  In this particular application, I added a validation to check and see if the category was assigned to any exisiting tickets.  If not, then it was safe to be deleted and a button to complete the action would appear, otherwise, that option was not made available until all existing tickets using the category were switched to something else.

```if issue.tickets.empty?```

There's a lot that goes into building an application and my respect goes out to all of those who seem to do this effortlessly and with sound logic.  As for me, I'm still learning and I may come to find that there's an even better way to handle the exact scenarios listed above.
