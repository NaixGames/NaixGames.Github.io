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

In one of my <a href="https://naixgames.github.io/tutorials/005Tutorial/"> previous entries</a>, I wrote about the state pattern. The pattern delegates the problem of detecting the "state" of an object to a class, and then the behaviour is executed by calling methods on that class with an interface that is common to all state classes. The problem is then; how do we manage these classes? How do we make sure these classes actually are called when we want to execute their behaviour?

<h2> A first approach </h2>

If you only read my last entry about states you might not realize the problem. You might say "Just have a variable assigned to a state, and replace that in memory as required". So, you would do something like:

{% highlight ruby %}
public class PlayerStateMove(){
     public Execute(){
          //code for movement
     }	
     public CheckTransition(){
           if Gounded() { return this; }
           else { return new PlayerStateJump() }
     }
}
public class PlayerStateJump(){
      public Execute(){
            //code for jumping
     }	
     public CheckTransition(){
           if Gounded() { return new StateMoving(); }
           else { return this; }	
      }
}
public class Player{
      private PlayerState mState;
      Update(){
            mState.Execute();
            mState = mState.CheckTransition();	
      }
}
{% endhighlight %}

The code above, if you complete the relevant details, actually works. However, if you read my <a href="https://naixgames.github.io/tutorials/004Tutorial/"> Object Pooling entry </a>, your spider sense might be one fire. In short; on each change of state we are creating a new state, which requests new memory. Having thousand of entities moving around and having memory problems because of fragmentation can get really problematic. Also, just requesting memory for storing the new states each time is expensive. Again, in the case where we have a bunch of actors running around in the scene, you are going to see the stutter. Clearly, we need to do better. 

<h2> Enter the FSM </h2>

Here is where our friend comes to save us. The FSM is what allows us to do better when managing states.  An FSM is a container for states so they are managed smoothly. That is all there is. We start by creating the state at the start, storing them on a FSM and then using that to execute the code in the state. So, in the code above, we could add;

{% highlight ruby %}
public class FSM{
      public PlayerState mState;
      public PlayerState[] mStateArray;
      
      public void Execute(){
            mState.Execute();
            mState.CheckTransition();
      }
	public void CheckTransition(){
              mState = mStateArray[mState.CheckTransition];
       }
}
{% endhighlight %}

And the player code is slightly modified to be 

{% highlight ruby %}
public class Player{
      private FSM mFSM;
      Init(){
            mFSM.mStateArray = {new PlayerMoving(), new PlayerJumping()}
      }
      Update(){
            mState.Execute();
            mState = mState.CheckTransition();	
      }
}
{% endhighlight %}

where again I omit some details of implementation. The main thing is; states are now created at initialization, and then the FSM just manages those states. There is no high memory management, just moving the pointer mState to the state that corresponds to.

<h2> Where do we store the data? </h2>

A big problem in this is: where do we store data and pass it to the states? If you have any decent game object, you might want to store data on it (for example, maximum speeds and accelerations!). Then you want to pass this data to the states so they can use them in their code! There is a couple of ways of dealing with this. One possibility is storing the data in the states. That has the advantage that is clear where the state is going to execute. However, if the data is going to be shared between different states, then it can be incredibly messy to synchronize data in different states.

A nicer approach is to have a dictionary in the game object or FSM that has all the relevant variables, and then the state just casts the value of these variables to get information about the object. That is incredibly neat, as then the States don't contain any data at all. In that case ... you may as well not even instantiate them, and just have them be static classes. So then states can just be a case of the <a href="https://naixgames.github.io/tutorials/006Tutorial/"> singleton </a>! Which actually make saves you all the heartache of memory management. Not only that, you will be sharing a SINGLE state for ALL instances of a particular actor! HIGH EFFICIENCY BABY!

However, the most likely case is that the use of the singleton pattern and having no data AT ALL on the states makes your head spin. You might want to cache SOME data on the states to avoid consulting the dictionary repeatedly. In that case, having those static elements might not be possible. However, the principle is the same; you might be able to share some states between different actors, saving you even more time and memory. A lot of optimization can be done in this case. This is the reason why I thought it important to know the existence of the singleton pattern before doing this post: you need to be aware of how far deep these implementations can go in complex scenarios. A big bunch of game optimization of changing how many of these states we have created and running in different scenarios.

<h2> The world of finite state machines </h2>

With this post, you can have fun trying to implement a FSM in a small project. However, in any real game, you might want the FSM to have more features. A simple variation includes executing code on each transition (one for exiting a state and another one for entering a state). You might also want to be able to execute repeated sub-routines in a FSM ... in that case, hierarchical/sub-FSM might help! You might also want to be able to exit the current logic you are following on certain conditions, and in that case, storing a "blip" state might be useful. Also, it is common for AI to be FSM themselves, so having a variation that is also an input interface might provide useful for allowing control of actors in a really modular way.

FSM are most of the basis of the organization and logic of game objects since ... forever! Every company might have their own FSM, and each programmer might have a different take on what is best. Be sure to come back and argue about that in a couple of months! (Please don't)

Also, an earlier version of this post had much more discussion on having singletons states, but I decided to cut it down. Guess I didn't mention singletons as much as I expected ... might have done this post months ago if I had noticed! Oh well :)
