---
title: "The state pattern" 
layout: post
date: 05-02-2023

usemathjax: true

image: 
  path: tutorials/images/005Machine.png 
  thumbnail: tutorials/images/005Machine.png
  caption: "A slot machine. Another Bad joke."
  categories:
     -intermediate
     -coding
     -pattern
  tags:
     -intermediate
     -coding
     -pattern
---

<h2> How to organize your behaviour without ifs. </h2>

After my last <a href="https://naixgames.github.io/tutorials/003Tutorial/"> post on input mapping</a> you return to work on your game, Purplete. You go and start using your new inputs to create Wadeline’s logic. You start by doing something that looks like:

{% highlight ruby %}
If (InputRight){
   MoveRight();
}
If(InputLeft){
   MoveLeft()
}
{% endhighlight %}

So far so good! You now add a Jump action so you do:

{% highlight ruby %}
If (InputRight){
   MoveRight();
}
If(InputLeft){
   MoveLeft()
}
If (JumpInput and Grounded){
   JumpAction()
}
{% endhighlight %}

Now you add a duck input, so now this turns into:

{% highlight ruby %}
If (InputRight){
   MoveRight();
}
If(InputLeft){
   MoveLeft()
}
If (JumpInput and Grounded){
   JumpAction()
}
if (DuckInput){
   DuckAction()
}
{% endhighlight %}

And that is already looking ugly. Furthermore, can you spot the mistake? Right now you can duck while in the air. Which could create unwanted behaviour. You could add a Grounded check for the DuckAction … but now you can also Jump while ducked!

Suppose now you add an attack that only works when grounded but not while ducking. And now an air attack. And now an attack while ducking. Can you imagine how many ifs you would need for that? How would you spot bugs in that code?  Clearly, there must be a better way. Enter the State pattern!

Note: between the horrible code above and the state pattern there is a middle ground: using Enums that classify the state of your character, and use that to process the logic. In small characters this can actually be good enough (I have used it for some of my game jams game). However, this will quickly get messy for bigger and more complex characters.

<h2> The state pattern </h2>

The idea is as follows: you decouple the behaviour of your “States” into different classes. Then the behaviour of your whole actor is delegated to the State class. That way which action you execute depends solely on the behaviour. So now your Wadeline class can look something like this;

{% highlight ruby %}
class Wadeline{
      private WadelineState mState;

      Update(){
            mState.Execute();
            mState = mState.StateTransition()
      }
}
{% endhighlight %}

Note the action is delegated to the Execute method of the WadalineState class. The change of state can also be delegated to the state class (as is the case). For example, the WadelineJumpingState would transition to the WadelineGroundedState when it becomes grounded. 

Now knowing which State your character is in becomes much easier; you just look at which particular class mState belongs to. This can be helpful for debugging, as you know exactly which state your character is in. Further, it can also help your logic, as you changing an animation state is just looking at the changes in the state of your class.

<h2> Some problems to follow in the next post </h2>

I just talked about the marvellous things this pattern can do, but it also creates some problems. For example; which class create the states? How do we reference the new state in the state transition? How do we pass data between different states? All these questions have a common factor for their solution: creating a finite state machine. 

However, I just shoved the concept of the State pattern down your throat. I will leave you to process that and come back in the next blog post to talk about the finite state machines :)