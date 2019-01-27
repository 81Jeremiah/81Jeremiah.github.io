---
layout: post
title:      "Adding JS to Rails and Breaking a Mental Block"
date:       2019-01-27 01:13:50 +0000
permalink:  adding_js_to_rails_and_breaking_a_mental_block
---


In the 4th project for Flatiron school we are required to add JavaScript to a Rails project that was made previously. Or more specifically the requirement was primarily change a Rails app to incorporate the use of JavaScript to load data asynchronously on various loaded view page. I am not sure how other Flatrion students felt but this whole process was extremely challenging for me. Learning that programing means to improve upon something that you have already built seems like a simple concept. However, breaking  and rebuilding something that already worked was a real struggle. I used several techniques to get on through this project. The highlights were using the Pomodoro technique. This basically means that I set 25 minutes timers and did the whole product in many of 25-minute chunks. From these chunks I was able to stop procrastinating and little by little accomplish the task. I also never fully grasped all the elements of JavaScript, for this I spent a lot of time googling, reading the MDN web docs, and eventually found myself going through the Codecademy course on JavaScript. The last l technique I wan to mention was to go through other Flatiron students code on Github to see how they made these changes. I hope those suggestions can help someone else out because those 3 items are what finally got me to the point where I can submit my project( Hooray!). That being said, I want to highlight a few of the challenges in the code and how I wrote the code to meet all of Flatirons requirements. 


**1. Must render at least one index page (index resource - 'list of things') via JavaScript and an Active Model Serialization JSON Backend.**

Loading a full index page with an ajax request is actually quite simple. It takes a bit to get the hang of it, but as long as you grasp the concept of any http verb, it will make sense.  Ajax at first is super abstract. To me it can appear like some sort of magic is happening as text can load on the page as view stays on the same URL. That is correct, the user is on the same web page but ajax allows us to tap into the server without changing the web page on the view. Say I need to load an index page. To do so in the browser it would be as simple as typing the url /quizzes and sending a get request to load a new page. Ajax allows us to make that GET request by using a jquery/ajax function such as 
```
$.get(/quizzes, (data) =>{
// do something with the data
}
```
The AJAX above has made the GET request. The data variable above is the real magic, as it holds the response of the GET request. There are a few options of how the data can be sent back from the server. It can come in the form of HTML and in that case the HTMl string can be inserted in the DOM. Another  option is for the data be JSON. JSON is JavaScript Object Orientation. This is where the is formatted in nice little chunks set up as JavaScript objects. Objects in javascript are a bit like hashes in Ruby. This JSON structured data is kept in a backend known as an API( which means Application programming interface) . Rails makes converting ruby objects to JSON very simple with some ready go methods. All that has to be done is to tell rails to render a ruby object as JSON. However only rendering JSON can lead to a problem if a user navigates to the url of /quizzes it will show only JSON. This can be fixed with telling rails when to respond with json or html.  Useing the repsong_to method from Rails as noted below accomplishes this task.
```
respond_to do |format|
      format.html { render :show, :layout => false }
      format.json { render json: @quiz, status: 200}
```


**2. Must use your Rails application to render a form for creating a resource that is submitted dynamically through JavaScript. **

Sending a POST request with AJAX is similar to a GET request. The setup is a little different as you have to serialize the data that was entered into the form. To do this, first you have to get data to be serialized.  By hijacking the submit event on the form you will have access to the “this”. In a form situation the “this” is now the information being submitted. By saving this as a variable and attaching the jQuery function of serialize, jQuery will do the heavy lifting and serilaze all the data the from is trying to submit. That serialized sting gets passed into with the AJAX POST request. In rails when a POST request is made the server still reacts in the same way as any POST request and if successful the controller will render the show page which can be returned in as JSON data and created into a JavaScript Model object as described in detail below. 

**3.Must translate the JSON responses into Javascript Model Objects using either ES6 class or constructor syntax.**

Using ES6 class syntax is the moment that I finaly understood JavaScript to work like an object-oriented language. Coming from a few months of Ruby, building classes is one of the great building blocks of the language and was a concept that at this point should be easy for anyone familiar with Ruby. Although the class syntax in JS doesn’t behave exactly like it does in Ruby, it allows us to build visualy appealing classes with methods that can be called once the object is created. When a class is made in this fashion, the methods won’t take up extra memory each time an object is created. Instead the method is actually the same for each object made from the constructor as it actually belongs to the prototype and not the object itself.

For example after I posted data with a AJAX post request the data is returned in JSON. That data can be used to create a JavaScript object as per the below:

```
class Quiz // some nice syntactic sugar allows this to look similr to Ruby classes
  contructor(quizJson) – //quizJson variable comes from the JSON returned after the user input is sent via a post request.

//Bellow builds the attributes to the JS object using the returned JSON. Notice the use of this. I like to think of this similar to the use of self in Ruby

this.name = quizJson.name
    this.id = quizJson.id
    this.userName = quizJson.user.username
    this.userId = quizJson.user_id
```

Within this class I added a few methods to easily create HTML to insert into the DOM. 
Such as..
   ```
createTr() {
     return `<tr> <td> <a href="quizzes/${this.id}">${this.name} </a></td> <td> <a href="users/${this.userId}"> ${this.userName} </td> <td> ${this.highScore} </td> </tr>`
   }
```
Because of how JS class keyword works these methods can all be inside the class function. It is possible to achieve the same goal by using explicitly assigning methods to the prototype of the Object Contructor, but I prefer to use the ES6 syntax to keep it within the class as I think it looks cleaner.
Just like in Ruby the quiz object can now call a method with simple dot notation. For instance, in my JS file I created various functions to generate the html I needed to injext into the DOM. Instead of having to repeat lengthy strings various elements and nodes were dynamically created with functions resulting in much cleaner code. 

The best example of this was I needed to inject 5 questions and answers into the DOM by placing them into the corresponding  Ids. Since I knew that question1 from the original quiz ruby object would always go to the Id 1, my code started off hardcoding each jQuery insertion. This resulted in some very lengthy ultimately wet code. By creating a function in the JS Class for quiz I was able to use a simple forEach loop to go through the question and answer array to insert each question and answer dynamically. By using JavaScript Model objects and methods I was able to create drier and easier to read code. 

JavaScript and AJAX are powerful tools to allow an app to load much faster and increase user interaction. Now, after these implementations and a few others I have a rails app with some nice JavaScript front end. 

