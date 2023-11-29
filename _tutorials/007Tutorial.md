---
title: "The finite state machine" 
layout: post
date: 06-06-2023

usemathjax: true

image: 
  path: tutorials/images/007Machine.png 
  thumbnail: tutorials/images/007Machine.png
  caption: "A slot machine, with a bad drawn 3."
  categories:
     -intermediate
     -coding
     -pattern
  tags:
     -intermediate
     -coding
     -pattern
---

<h2> Or how to organize your states </h2>

After postponing this for months I am finally here to write about Finite state machines. Hurray! Now, you will realize I tricked you, and after knowing the state pattern, a Finite State Machine (FSM from now on) is not much.

<h2> Recalling the problem with states </h2>

In one of my previous entries, I wrote about the state pattern. The pattern delegates the problem of detecting the "state" of an object to a class, and then the behaviour is executed by calling methods on that class with an interface that is common to all state classes. The problem is then; how do we manage these classes? How do we make sure these classes actually are called when we want to execute their behaviour?