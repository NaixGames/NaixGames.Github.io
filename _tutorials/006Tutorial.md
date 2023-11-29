---
title: "The singleton pattern" 
layout: post
date: 16-03-2023

usemathjax: true

image: 
  path: tutorials/images/006Machine.png 
  thumbnail: tutorials/images/006Machine.png
  caption: "A slot machine, with a 2 intead of a 1."
  categories:
     -intermediate
     -coding
     -pattern
  tags:
     -intermediate
     -coding
     -pattern
---

<h2> Or an easy way to setup your manager </h2>

If you have been following my entries you might remember from the last post that I promised to take about Finite state machines. However, I was halfway during that entry and I realized I wrote the term singleton 3 times. Since I haven’t written about singletons that was incredible confusing. So, the entry for Finite State Machines will need to wait until next month, as it seems there is a more pressing gap in knowledge. Hope you are fine with this :).

Anyway, lets talk about singletons. Singletons are (is?… this is confusing…) a really useful programming pattern. Thankfully they are also one of the easiest to explain. The core idea is: “Ensure there is a single instance of an object and make it global”.

Wait, how do we even to do that? Well there are different ways, which different pros and cons. The one I like is basically this:

{% highlight ruby %}
public class SingletonExample{
    SingletonExample(){
        if (instance≠null){
            BlockHavingAnotherInstance()
            return;
        }
    instance=this;
    }
private static SingletonExample instance;
public static SingletonExample{return TryToGetInstance()}
private SingletonExample TryToGetInstance{
    if (instance=null){
        return HandleNotHavingTheInstance()
    }
    return instance;
}
{% endhighlight %}

Creation should be handled in such a way that you always have a SingletonExample created before it is called, or by a smart implementation of HandleNotHavingTheInstance() (normally created the instance and storing it somewhere). With that, and some precautions, SingletonExample.Instance should always return the only object of the class SingletonExample. If you try to create another one you will be blocked by BlockHavingAnotherInstance(). The fact that Instance and instance are public and static ensures the variable is maintained through every possible implementation of the class. Smart, huh?

<h2> Avoid the singleton pattern </h2>

After me explaining the singleton pattern you might be really confused with the sub-title. I just opened to you a way of having an easy and unified access to a particular class, why would I take that away for you? Well, because I just gave you an easy and unified access to a particular class. This implies that any modification to the singleton class will have incredible ramifications through your whole code.

Let me hit you up with an example for this. Imagine you are doing a single player game and you have a player class. Since you want this to be easily accessible by inputs, data managers, etc. you make this a singleton. That means you never set up a reference to the player, and now in almost every system of your game you have something of the form Player.Instance.

Then comes marketting and gives you a harsh truth: The game is totally unprofitable without a multiplayer mode and you need to add 2player co-op. If you have been following me you will already knows that means the whole code is useless. From the singleton pattern you cannot add another instance (since you are forcing only one). And even if you unforced the singleton pattern from your player there is not a single manual reference to the player itself. So every system needs to be rewritten. Ever heard interviews to developers being asked about multiplayer mode and they answer “no, with our architecture we cant add multiplayer”? Well now you know why :).

<h2> When there is no other choice </h2>

Now that I have told you when NOT to use the singleton pattern I must tell you when you should. And this is normally when there is no other choice. Sadly games are by design in every editor object oriented programs. Meaning you need references to other objects to get information. However, passing every object you might need it not feasible. Imagine how many variables if you would always pass the States of the object, the input, camera, physics, audio .. and so many things! So at one point one needs to cut the cake and put a singleton somewhere.

In general, a good place to add a singleton is on a class you say “if this breaks and needs to be replace, everything in the game still runs … less flashy, but still runs”. An AudioManager is a good example from this.  Any reference to the audio manager calls some sound, and if that break, we can just replace every call by a Mute sound. Not pretty, but runs. An input manager would be another good example; normally an input manager just respond to calls of input assignation. If that breaks; well you can assign the input yourselves. That might stop dynamically handling input requests (ie, if a keyboard/controller gets connect), but the game will still run. A global event dispatcher is another good place in which I would use a singleton (and, surprise, this will be the topic of a couple months in the future :) ).

I really cannot give enough warning to avoid having a system which is based on singletons. A couple of them is fine, but the more you add the more care you need to maintain that code. After all; EVERYTHING in your game could make request to it. So if one thing is not working as expected, bugs will just start creeping in faster than you could realize. Managers (ie, classes that resolve certain global aspects of the game) are normally place where singletons are used, because even if they were not singletons any changes in them would affect the whole game. Hence, you may as well just accept that fact, and make them global. You also get the added bonus you have a single instance of them, which is in general desirable! You would not want two Audio systems working at the same time, would you?

<h2> How to mitigate the problems of the singleton pattern </h2>

If I convinced you about the dangers of the singleton at the start, at this point you might be “Ian, I am afraid to use singletons, is there another way around this?”. The short answer is yes, using a System locator.

A system locator is basically a singleton that works as a bottleneck for requesting unique global elements. Allowing you to work with classes that are not singletons as if they were. So basically, instead of doing something like AudioManager.Instance, you do something like SystemLocator. Instance.GiveManager<AudioManager>(). To ensure there is only one class, in the creator of AudilManager we add a If (SystemLocator. Instance.GiveManage(Audio).) then DontCreateClass. You get the benefits of the singleton without having such big coupling; if you want to change the class AudioManager, you can do another class and then change the class SystemLocator.Instance.GiveManager(Audio) returns. It makes changing classes much more painless than directly having a singleton. The problem with the System Allocator? Indirection. Now to understand what happens when you want to call a Manager, you need to understand what a Manager does AND what the System allocator does. Well, we cannot have everything, for every pattern there is a pro and a con.

In case you are interested; there is a bunch of slight variations of ways to implement the singleton pattern. There is also different flavours for the a Service locator. As always, the bible here is GameProgrammingPatterns. Hope that this post at least has woken up your singleton curiosity ;).