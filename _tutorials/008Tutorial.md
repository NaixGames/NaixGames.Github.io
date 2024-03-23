---
title: "How to do PowerPoint in Godot" 
layout: post
date: 23-03-2024

usemathjax: true

image: 
  path: tutorials/images/008PowerPoint.png 
  thumbnail: tutorials/images/008PowerPoint.png
  caption: "A pixel art symbol of PowerPoint."
  categories:
     -beginner
     -coding
  tags:
     -beginner
     -coding
---

<h2> The GameDev way of doing presentation </h2>

I like doing presentations. I feel sharing my experience and knowledge is useful for the larger community, while also helping my abilities improve and get tested by other peers. However, I feel that no software 
for doing presentations fits my needs. I have used PowerPoint, latex beamer and IPE, all of which feel too rigid and not reactive enough (although IPE finally allowed me to do pretty pictures easily). I have seen
other presentation frameworks, like Presi, which I think are more reactive and pretty, but annoying to set up and too distracting.

If only there was a piece of software that would allow me to present information, have animations, allow me to control how reactive things are, while also receiving inputs and allow me to be more creative with presentations ...

If I was not being clear enough; game engines fit exactly my needs. It offers control and responsiveness. And I am quite used to working on them. That is why I embarked on reinventing PowerPoint on Godot.

<h2> High level architecture </h2>

First I will explain on a high level how I did this. If you want to attempt to recreate this yourself, without getting spoiled too much, this is the section you can read.

The organisation can be summarized in the following picture:

![System](/tutorials/images/008PresentationSystem.png)

<em>A diagram to represent the presentation system. Not quite an UML, but I guess you can try to do that yourself.</em>

At a glance; we have one component that is PresentationManager, which is the highest-level component. This has a list of PresentationSlides, which as the name suggests, are the different slides or pages of the presentation. 
Each one of these Slides has a list of BulletPoints. These bullet points are the elements that pop up when you click "next" on a presentation, and hide when you click back. 

Going into some more details; the presentation manager has a list of references to all the slides of the presentation, an index to know which slide we are on and a pointer to whatever it has loaded as the "actual slide". 
It has a method that unloads the actual slide and loads the next slide or the prior one. Note this needs some edge cases when we are on the first and last slide. Note also that to add a slide to the presentation,
all we need is to add it to the list of the presentation manager, and this process needs to be done manually (see the last sections for some alternatives to this). Also, this component should initialize the first
slide at the start to kick off the presentation!

The PresentationSlides deal with bullet points in a similar way that the presentation manager deals with slides, but the added advantage is that the Slides are a self-contained prefab/scene and bullet points are
children (so they can be automatically loaded at the start). It has a list of the bullet points, an index to know which bullet point we are in and some methods 
to request the next and prior bullet point to appear. When the last bullet point has already been loaded and we try to load the next one, we instead request the presentation manager to load
the next slide (and do something similar when we are on the first bullet point to load the prior one). Note all the input is managed by the slide, for the reason that dealing with edge cases of bullet
points is easier that way. Note also, that in this case we don't need to add bullet points manually to include them in the slide internal list, but rather we can run through its children and add them to its list.

The bullet points need two main methods; one to deal with requests to appear and one to deal with disappear requests. For appearing you could do anything you want: make the bullet point visible, 
activate a tween, start an animation, etc. Similarly, the disappear request should do anything to undo that (disappear, go back into the tween, reset the animation and hide, etc.). This is
where you can be more flexible and creative and you are not constrained by anything.

Note that the BulletPoint objects can parent anything too. You could have a bullet point that parents a game actor that moves around to explain a point you are making. Hell, technically, the actor
doesn't need to be a BulletPoint of that slide, and in that case, it will always be on the slide (the last point is useful for titles).

To use all of these classes; you create a new "Object" (Prefab for Unity or Scene for Godot) for each Scene. Then you add a PresentationManager script to the top level, and for doing the slide itself
you add all the elements you want and attach a bullet point component to the one that needs it. Then a new "Scene" (Scene for Unity, another Scene for Godot :P) is created that has a PresentationManager
component at the top level. Finally, you add the references to the slides on the presentation manager. With this, you should be good to go! Just play the "game" on the scene with the presentation manager. 


<h2> Some coding details </h2>

In this section, I discuss some small implementation details. In case you want to be spoiled, this is the section to look for! Note I am not going to give a step-by-step setup, but rather go through
how I implemented the classes I discussed above. Note this is all done in Godot with C#. Adapting it to your particular engine and language is up to you :).

Without much more introduction, the PresentationManager:

{% highlight ruby %}
public partial class PresentationManager : Node
	{
		[Export] private string[] mSlidesPaths; //This have the paths of the slides on your project
		private int mSlideIndex=0;
		private Node mActualSlide;

		// --------------------------------------------------------------------

		public override void _Ready()
		{
			mActualSlide = ResourceLoader.Load<PackedScene>(mSlidesPaths[0]).Instantiate(); 
			this.AddChild(mActualSlide);
			PresentationSlide slide = (PresentationSlide) mActualSlide;
			slide.Initiate(this);
		}

		// --------------------------------------------------------------------

		private void LoadIndexSlide(){
			Node newActualScene = ResourceLoader.Load<PackedScene>(mSlidesPaths[mSlideIndex]).Instantiate(); 
			
			PresentationSlide slide = (PresentationSlide) newActualScene;
			slide.Initiate(this);
			
			mActualSlide.QueueFree();
			mActualSlide = newActualScene;

			this.AddChild(mActualSlide);
		}

		// --------------------------------------------------------------------

		public void LoadNextSlide(){
			if (mSlideIndex==mSlidesPaths.Length-1){
				return;
			}
			mSlideIndex++;
			LoadIndexSlide();
		}

		// --------------------------------------------------------------------
		
		public void LoadNextSlide(){
			//Similar as above. Do this on your own ;)
		}

	}
{% endhighlight %}

nothing new to comment on besides what was said in the last section. Now, the PresentationSlide is more interesting:

{% highlight ruby %}
public partial class PresentationSlide : Node
	{
		private PresentationManager mPresentationManager;
		private InputReaderAbstract mInput;
		private List<BulletPoint> mBulletPoints = new List<BulletPoint>();
		private int mBulletPointIndex = 0;

		// --------------------------------------------------------------------

		// Called when the node enters the scene tree for the first time.
		public void Initiate(PresentationManager presentationManager)
		{
			//This automatically loads all bullet points to then go through them on the presentation.
			foreach (Node node in this.GetChildren(false)){
				if (node is BulletPoint){
					BulletPoint bulletPoint = (BulletPoint) node;
					mBulletPoints.Add(bulletPoint);
				}
			}

			mInput = InputManager.Instance.GiveInput();
			mPresentationManager = presentationManager;
		}

		// --------------------------------------------------------------------

		public override void _Process(double delta)
		{
			if (mInput.IsButtonJustPressedInput("Next")){
				if (mBulletPointIndex == -1){
					mBulletPointIndex++;
					return;
				}
				if (mBulletPointIndex<mBulletPoints.Count){
					mBulletPoints[mBulletPointIndex].ShowBulletPoint();
					mBulletPointIndex ++;
					return;
				}
				mPresentationManager.LoadNextSlide();
			}
			if (mInput.IsButtonJustPressedInput("Back")){
				if (mBulletPointIndex == mBulletPoints.Count){
					mBulletPointIndex--;
					return;
				}
				if (mBulletPointIndex>-1){
					mBulletPoints[mBulletPointIndex].HideBulletPoint();
					mBulletPointIndex --;
					return;
				}
				mPresentationManager.LoadPreviousSlide();
			}
		}

		// --------------------------------------------------------------------

	}
{% endhighlight %}

The input part needs to be adapted for your system and to how you deal with inputs. As you can see, this is pretty similar to the PresentationManager, with some caveats to adjust to bullet points. Also, with this implementation when you go back
to a previous slide, no BulletPoint will be loaded. This is by design; I have seen too many presentation in which the speaker wants to go back and need to scroll through every single bullet point. This to allow a faster way of doing this.
Now, for the bullet point class itself, the simplest version for this:

{% highlight ruby %}
public partial class BulletPoint : Control
	{

		// --------------------------------------------------------------------
		public override void _Ready()
		{
			this.Visible = false;
		}

		// --------------------------------------------------------------------

		public virtual void ShowBulletPoint(){
			this.Visible = true;
		}

		// --------------------------------------------------------------------

		public virtual void HideBulletPoint(){
			this.Visible = false;
		}

		// --------------------------------------------------------------------
	}
{% endhighlight %}

As you can see, the ShowBulletPoint() and HideBulletPoint() are marked as abstract, so inheriting classes can do another method if they like. You can go wild with what you do in ShowBulletPoint and HideBulletPoint. The very simplistic
implementation above should at least give you powerpoint.

<h2> Some changes that could be made </h2>

<h3> Have the presentation manager load all the slides from the start. </h3>

As I have mentioned before, the PresentationManager object has references to the slide "objects" in memory. An alternative is to have it so the PresentationManager does not instantiate the Slides,
but rather have a reference to the Slide "game objects" and you add these objects manually to the PresentationManager game object.

There is a reason why I didn't go this route. First, if done this way, that would mean the game would load all the slides at the start-up. That means a bigger strain on the computer running the presentation
(the memory needs to hold all that data!) and also a bigger start-up time. The way it is done, the start-up is faster and in general the memory usage is lower. The downside is that loading each new slide is a bit
slower, but the slides shouldn't be SO big that this issue even becomes noticeable.

Anyway, there is not a clear answer here, you can decide what approach fits you the best.


<h3> Have the PresentationManager deal with input. </h3>

One dirty thing about my implementation is that the Slide deals with input. A cleaner way would be for the PresentationManager to deal with inputs, and give requests to the Slide go to the next bullet point or back. 
This is cleaner because then the component that is the most "top-level" one deals with communicating with the input it its own, and just pass on request "down" the hierarchy.

The only reason I didn't do it was because of laziness. In this implementation you need to deal with some edge cases on the slide, and then pass back to the PresentationManager requests to deal with those. While certainly
this can be done nicely, I just found it plain cleaner for the Slides to deal with all of this, which meant being the one that received the input.

This is one of the "this is not about function, but about cleanliness" discussions you can have that may drag on forever. If you only care about function you can ignore the last point. If you care about form; I just
found happiness in my code being smaller using this setup. Let me be happy :)