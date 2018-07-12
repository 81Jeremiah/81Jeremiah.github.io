---
layout: post
title:      "Active Record, and My Sigh of Relief "
date:       2018-07-12 04:10:22 +0000
permalink:  active_record_and_my_sigh_of_relief
---


With a successful passing of the CLI gem project I was ready and eager to get started on the Sinatra section. With excitement I open up the first SQLite lesson and soon my excitement begins to fade as I learn I'm about to make digital spread sheets, a lot of them. Now, I'm not downplaying the importance to ways to store data, in fact this blog post is ultimately going to praise the ability to store data, I'm just simply stating that the switch from OO Ruby to SQLite to me felt a little, well......boring. 

Ultimately SQLite is a form of SQL. SQL, from my little research is the main way almost all data is logged on the internet. Ultimately, the process is very similar to building a spreadsheet. The basic steps are(see below for the acronym CRUD):
1. You CREATE a table with columns
2. You insert the data you've CREATED and store in your table 
3. Through a process of various queries you can, READ the data, UPDATE the data or DELETE the data, or the table entirely. 

Pretty basic process but can often feel a little tedious, and a lot of the labs seemed to have 'code smell' as I kept repeating a lot of my code. As I entered learning about ORM's (Object Relational Mapping) it seemed like the code was endless. ORM's take the principles of SQLite and use OO Ruby to turn your data into objects rather than raw data so it can be used like objects. 

Through the ORM process we learn how to dynamically create methods that preform the CRUD method that not only create objects but also persist them in our SQLite database. The ORM process is significantly more advanced than what was done with SQLite as we are now dealing with objects rather than raw data. The most important part of the ORM methods that ae being built is they are dynamic which means they don't have be rebuilt for each object. Pretty cool, but once again a lot of work. Wouldn’t it be great if these ORM methods were stored somewhere, perhaps a gem where they could be accessed without have to write the code each time?

Enter the magic of Active Record.  In Active Record's Readme there is line in the philosophy section that actually says, 'Magic is not inherently a bad word" Which, yes Active Record is magic, but on the other hand it's perfectly logical. I would say Active Record is more of a gift than magic. Active Record does all of the heavy lifting to allow many CRUD methods to be inherited by a class simply by adding the Active Record gem to an application, with the proper configuration. With a simple table migration and building a class to inherit from ActiveRecord::Base the new object now has 387 methods that can aid in every step of the CRUD process. This is all done with just a few lines of code!

My ultimate takeaway from the awe I was in upon my first experience with Active Record is ultimately gratitude. It amazes me that the coding community is built around the concept that there is always a better way to write code. I am grateful for all of the programmers before me that have contributed to improve the coding world, and I only hope one day I can do the same.





