---
author: jsr
comments: true
date: 2011-05-25 22:16:28
layout: post
slug: getting-started-with-play-framework-scala-and-casbah
title: Getting started with Play Framework, Scala and Casbah
wordpress_id: 5
categories:
- Casbah
- MongoDB
- Scala
---

I've been dorking around with the [Play Framework](http://scala.playframework.org), [Scala](http://www.scala-lang.org/) and [Casbah](http://api.mongodb.org/scala/casbah/) recently and I wanted to share my recent progress. I'm new to scala and play, so I am probably running afoul of some of the best practices out there. If you've got any advice for me after reading this post, please do share.

Here's what we're going to do in this tutorial:




  
  1. Set up a new play framework project using scala

  
  2. Add the dependencies so you can use Casbah

  
  3. Create a controller that lists / posts messages to MongoDB

  
  4. Create templates for the app

  
  5. Create routes for our new controller

  
  6. You win!





## Set up a new play framework project using scala




    
    
    bash$ play install scala 
    bash$ play new myCasbahDemo --with scala 
    



Now you should have a new directory called "myCasbahDemo" populated with the template for a play app. 



## Add the dependencies so you can use Casbah



We need to modify the conf/dependencies.yml file to tell play how to load the casbah dependencies. Here's my conf/dependencies.yml file


    
     
    # Application dependencies
    require:
        - play
        - play -> scala 0.9
        - com.mongodb.casbah -> casbah_2.8.1 2.1.2
    
    repositories:
      - scalatools:
         type: iBiblio
         root: http://scala-tools.org/repo-releases/
         contains:
           - com.mongodb.casbah -> *
           - org.scalaj -> *
    



Now we can run 

    
    
    bash$ play dependencies
    



Play will fetch the casbah dependencies and install them in the local project. 



## Create a controller that lists / posts messages to MongoDB



I created a new controller in app/controllers/Messages.scala with the following content: 


    
    
    package controllers;
    
    import play.mvc._;
    import com.mongodb.casbah.Imports._
    import scala.collection.JavaConverters._
    
    object Messages extends Controller {
    
      val _mongoConn = MongoConnection()
    
      def index = {
    
        val msgs = _mongoConn("casbah_test")("test_data").find( "msg" $exists true $ne "" )
        val msgStrings = msgs.map( (obj: DBObject) => obj.getOrElse("msg","") )
        Template( 'msgStrings -> msgStrings.asJava )
      }
    
      def save(msg:String) = {
        val doc = MongoDBObject("msg" -> msg)
        _mongoConn("casbah_test")("test_data").save( doc )
        Redirect("/messages")
      }
    }
    



Our controller has two methods: "index" and "save". 

Index grabs a list of messages from mongo, extracts the text, and renders them in the corresponding template (we'll look at those in a moment). 

There's a bit of magic going on here. Play uses Groovy as its template language, but many Scala types don't directly translate to groovy types. So you'll notice these lines of code: 


    
    
        val msgStrings = msgs.map( (obj: DBObject) => obj.getOrElse("msg","") )
        Template( 'msgStrings -> msgStrings.asJava )
    



First, we got back a cursor from our mongo query. So we use map to extract the "msg" attribute from each of the docs returned. This gives us a scala `List`. But the Groovy language currently used by Play templates do not know how to iterate over a Scala collection. In order to help programmers working with other JVM languages, Scala 2.8.1 + provide the `scala.collection.JavaConverters` library, which adds the `asJava` method to Scala Collections (And an equivalent `asScala` method to Java collections).  By calling `asJava`, we wrap our `List` in a Java compatible object that can be iterated. 

We are working to provide an add on to Casbah or a patch to Play to make this easier in the future. In the mean time, be sure to include `scala.collection.JavaConverters` and convert your scala types to java types before calling your template. 

Save accepts a new string message from the client and saves it to the collection before redirecting the user back to the message list template. 

Note that I'm opening a connection to MongoDB in the controller. This isn't a great design choice as this connection can be re-used by other controllers. In a future post, i'm going to add a real model layer and abstract the connection to mongoDB out. But for the time being, this will suffice. 



## Create templates for the app



There's just one template for my app, a simple page with a form field for submitting a new message, followed by a list of the existing messages. My template is in app/views/Messages/index.html, following the play pattern. 



    
     
    #{extends 'main.html' /}
    #{set title:'Home' /}
    
    <form action="@{Messages.save()}" method="POST"/>
      <input type="text" name="msg"/>
      <input type="submit" value="Add message" />
    </form>
    
    <ul>
      #{list items:msgStrings, as:'mess' }
      <li>${ mess }</li>
      #{/list}
    </ul>
    





## Create a route for our new controller


in conf/routes, we need to add a route to our new controller. Here's what mine looks like: 


    
     
    # Routes
    # This file defines all application routes (Higher priority routes first)
    # ~~~~
    
    # Home page
    GET     /                                       Messages.index
    POST    /                                       Messages.save
    





## Try it out!



First off, make sure that mongod is running on your localhost (we used the default constructor of the MongoDB connection which means it will look for mongod on localhost on the default ports. 


    
    
    bash$ play run 
    



Wait for the app to startup, then point your browser at localhost:9000

And bam! You've got your first play + scala + casbah app up and running! 

You can also check out the [full source code for my demo app](http://github.com/jsr/casbah_play_tutorial).
