---
layout: post
title: Alexa Development with ASP.Net WebAPI
description: "Developing Alexa Skills with ASP.Net WebAPI"
modified: 2016-12-22
tags: [alexa, .net, c#, asp.net, WebAPI]
image:
  feature: EchoDotNet.jpg
---

Like most devs new to Alexa development, I created my first Alexa Skill, [C# Quiz](http://bit.ly/CSharpQuiz "C# Quiz Alexa Skill on Amazon"), by copying and modifying the [Trivia Game Template](https://developer.amazon.com/blogs/post/TxDJWS16KUPVKO/new-alexa-skills-kit-template-build-a-trivia-skill-in-under-an-hour "New Alexa Skills Kit Template: Build a Trivia Skill in under an Hour"). I created an AWS Lambda function to host the back-end, and I chose to write it in node.js, because Amazon has so graciously created a [node.js SDK for Alexa Skills Kit](https://github.com/amzn/alexa-skills-kit-js "GitHub Page for node.js ASK SDK"), with sample code to get you started. This approach was perfect to get me started developing Alexa Skills, and I highly recommend you do the same thing for your first Alexa Skill. It will allow you to create and host your skill quickly, and it will force you to learn your way around the [Developer Portal](https://developer.amazon.com/alexa "Amazon Developer Portal"). You will learn about custom and built-in intents. You will get an idea about the structure of the requests and responses that you will have to handle in your code. Most importantly, it will familiarize you with the certification process, and allow you to get your first skill in the store. For some reason that is, at least mentally, one of the hardest parts. Once you have a skill in the store, you will likely be totally excited about working on the next one. You know, the one that will make the world a better place!

[freebusy.io](https://freebusy.io/blog/getting-started-with-alexa-app-development-for-amazon-echo-using-dot-net "Getting started with Alexa App development for Amazon Echo using .NET on Windows")
[serverfault.com](http://serverfault.com/questions/301655/how-to-create-a-self-signed-ssl-certificate-with-subject-alternate-names-san-f "How to create a self-signed SSL certificate with subject alternate names (SAN) for IIS websites")
[Bob Lautenbach's Template](https://github.com/boblautenbach/AlexaASKNetTemplate "GitHub page for Alexa ASK Template for .Net")
[Walter Quesada's Pluralsight Video](https://www.pluralsight.com/courses/amazon-echo-developing-alexa-skills "Developing Alexa Skills for Amazon Echo")
