---
title: "Some thoughts for input mapping" 
layout: post
date: 29-12-2022

usemathjax: true

image: 
  path: tutorials/images/003InputButton.png 
  thumbnail: tutorials/images/003InputButton.png
  caption: "A multiplication"
  categories:
     -beginner
     -coding
     -input
  tags:
     -beginner
     -coding
     -input
---

<h2> Or why Space should not be Jump. </h2>

Note: This entry is intended to be a high-level discussion on some do and don’t when structuring the input of a game. Every code here will be (really bad) pseudocode, but I hope the ideas come through better that way.

Imagine you are creating a new platformer, Purplete (Yes that is a colour, no I will not elaborate further). You start the creation of your game by working on the main character Wadeline. You create a square at first (for simplicity) and you start configuring the main input. You do something along the lines:

{% highlight ruby %}
If ("Press Left Key") { GoLeft();}
If ("Press Right Key") {GoRight();}
If ("Press Space"){ Jump(); }
{% endhighlight %}


You can go and create the functions I am missing in my example. You test it. Everything works fine. You forget about this code … until you have some trouble with it.

<h2> Why that code will create trouble </h2>

You continue to create your game. You eventually get to playtesting and you notice the first tester tries to move with WASD. Hence you go and change your code to:

{% highlight ruby %}
If ("Press Left Key" or "Press A") { GoLeft()}
If ("Press Right Key" or "Press D") {GoRight()}
If ("Press Space" or "Press W"){ Jump() }
{% endhighlight %}

Confident with your code is better, you go and test with a second person. Your confidence melt when you see the person arriving with a controller. You quickly go and change your code to:


{% highlight ruby %}
If ("Press Left Key" or "Press A" or "Stick In Controller Left") { GoLeft()}
If ("Press Right Key" or "Press D" or "Stick In Controller Right") {GoRight()}
If ("Press Space" or "Press W" or "Stick In Controller Up"){ Jump() }
{% endhighlight %}

Okay, you come back and make the player start testing the game. However, you notice he wants to move with the D-pad, not the stick! So you go and change this to:

{% highlight ruby %}
If ("Press Left Key" or "Press A" or "Stick In Controller Left" or "D-Pad left") { GoLeft()}
If ("Press Right Key" or "Press D" or "Stick In Controller Right" or  "D-Pad right") {GoRight()}
If ("Press Space" or "Press W" or "Stick In Controller Up" or  "D-Pad up"){ Jump() }
{% endhighlight %}

While this works you are already sick of working on the input of the main character on every test. This needs to change.

<h2> What is wrong and how it could get worse </h2>

If you are a beginner coder you might still not know what is wrong in the example above. The problem can be formulated as the main character is coupled to how you get your input. That means in its core it is impossible to split the main character to how you read what the player does. This is screaming that we need to split the input reading from the character.

Now, you might be stubborn. You might say “I don’t need to split this, that will only create more problems!”. My answer to that is that the problems can get worse. Much worse.

Imagine you need to do console ports. You would need to add in those ifs one more statement for every console you want to port, to check for the new inputs. Imagine if you want to do multiplayer. How would you try to split the character inputs? Would you do a switch depending on the player and then have two sets of gigantic ifs depending on which character is for which player? Finally, imagine you want to do online multiplayer. How you will differentiate the input in Computer A and the input in Computer B if they are (potentially) using the same keys to play?

All these issues can be solved by separating how we read the input from how we process it. (And the online multiplayer with some more networking knowledge magic).

<h2> The Input reader class </h2>

A simple way of splitting the input and player would be to create an Input reader class. Then, your player code would be reduced to


{% highlight ruby %}
If (mInput.IsPressingLeft()) { GoLeft()}
If (mInput.IsPressingRight()) {GoRight()}
If (mInput.IsPressingJump()){ Jump() }
{% endhighlight %}

Now note that once you write that code for the player, no matter how you change the input, you do not need to change it again. Now for the input reader, you can do something along the lines of:


{% highlight ruby %}
private Keys[] mLeftKeys;
private Keys[] mRightKeys;
private Keys[] mJumpKeys;
IsPressingLeft(){
   for (key in mLeftKeys){ 
   	if (key.IsPressed()){ return True; }
   }
}
//I guess you can figure out IsPressingLeft() and IsPressingRight()
{% endhighlight %}

Want to add another input for jumping? Well, just added it to the mJumpKeys array. If you are using any commercial engine you should be able to set the keys in the inspector, without any extra line of code! If you want to deal with different inputs for multiplayer; just create different instances of InputReaders which access different inputs … or better, different systems!

<h2> Some final notes </h2>

If we are using commercial engines, instead of doing that yourself, you might also be able to get libraries that do that for you. The Rewired package in Unity works more or less as I explained above. I hope that these notes, at least might help you understand why you would need to use them. And why you do not want to hardwire your input to other parts of your code.

Now I am missing so much outside this little note. One neat thing one could do is to make the InputReader an abstract class, and so allow even the AI to control the main character. But that might be the topic of another entry.

