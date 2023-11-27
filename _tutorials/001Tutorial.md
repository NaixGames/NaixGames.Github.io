---
title: "Probability and perception" 
layout: post
date: 21-08-2022

image: 
  path: tutorials/images/001DiceOne.png 
  thumbnail: tutorials/images/001DiceOne.png
  caption: "A dice on one"
  categories:
     -beginner
     -math
     -probability
  tags:
     -beginner
     -math
     -probability

---

<h2> Why you think the RNG sucks.</h2>

Randomness and chance are present, in one way or another, in almost every popular game. Card games, dice games, RPGs, Dungeon Crawlers, and so on, all have some sort of randomness in their design. While I could talk about how to incorporate randomness and balance in the design of games (I would like to do that eventually!) in this case I will write about how humans perceive probability, and what problems that can create in your games.

Suppose you are making (or playing) a grid-based war tactics RPG; Ice Badge (Nintendo should call me to name their games). You have 5 warriors attacking an enemy, each attack having a chance of 60% of hitting. How many hits you can expect to hit?

If you answered 3, that is correct. The <a href="https://en.wikipedia.org/wiki/Expected_value"> average number </a> (or expected value if you want a technical name) of hits is 3. However, the average may not always be a good representation. For an example of this, suppose now your warriors have a group skill, under which, they all attack and they all hit with probability 60% or they all miss with probability 40%. What is the average number of hits? An <a href="https://media.makeameme.org/created/and-then-we-5b2da1.jpg"> exercise left to the reader </a> is to show the answer is 3, again!

So yeah, the expected value might not be a good representation. But, how bad can it be? Let's go back to the case where each warrior hits the enemy with probability 60%. What is the probability we actually have 3 hits?

Let us compute this. First let us compute the probability the first 3 warriors hit (and consequently, the last two miss). Each hit has probability 0.6, and each miss probability 0.4. So having 3 hits and then 2 misses has probability (0.6)^3 (0.4)^2, which is, 0.0432.

![Swords](/tutorials/images/001Swords.png)

<em>The swords of the warriors in the setting we are calculating. I hope the colours, ticks and crosses are clear enough.</em>

Now to get the probability of having 3 hits (for any of the warriors) we need to account for the number of permutations of the order of the hits. That is, we multiply the number above by the total possible rearrangement of hit and miss. In other words, we multiply by the combinatorial number, which for 3 hits over a total of 5 possibilities, is 10.

So then the total probability of having 3 hits is 0.432. Note then having 3 hits does not happen even half of the time! Similarly, one can compute that the probability of having all five misses is 0.01024. So about one over a hundred players will get all misses on the situation we are looking at.

<h2> But why do I care about those numbers? </h2>

Imagine you have the following situation. You sell your game Ice Badge with the probabilities we described above. And at one point in the story, every player faces the 5 warriors attacking that lone enemy situation. If your game sells 10,000 copies you will have 100 players angry that all of their warriors miss their attack. They complain on the internet, saying your game is unfair. Maybe some players defend your game ... but not even half of them got the expected value of 3 hits. So not even the majority of your community can agree that your game is fair! Your game fails to sell any more copies because of bad publicity, and a fair number of players ask for a reimbursement.

Yeah, my scenario might be slightly exaggerated, but you only need to go to <a href="https://www.reddit.com/r/Xcom/comments/1wujue/how_badly_has_rng_screwed_you_over/"> X-COM reddit </a> to realize this is not as far from reality as you could think.


<h2> What can we do about this. </h2>

This happens because the perception of chance is different from what chance actually means. So a way of fixing this (not the only one, mind you!) is to show a different probability to the player, than what the number actually means.

In the case of Ice Badge (yeah, I am sticking with the name) we can do the following. We can say that the warriors will all attack and hit with probability 60%. But what we actually do to compute if an attack hits or miss is to use an array. That is, we have an array {1,2,3,4,5} and we randomly rearrange it. Each time a warrior attacks we get one of the numbers. if the number is 3 or smaller the attack hits, if not it misses.

![Swords](/tutorials/images/001HitMiss.png)

<em>How we start with an array of numbers and end with the Hit and Misses of the warrios.</em>

In this situation the first attack has a probability of 60% of hitting, so we are not lying that much! And, furthermore, after the five attacks, we know that EVERY player will get 3 hits. Nobody complains. Your game is saved.

The last technique of using an array is normally known as bagging, and as far as I am aware, it goes back to <a href="https://tetris.fandom.com/wiki/Random_Generator"> Tetris </a>. One can try to change and tweak the system. Maybe having ALWAYS 3 hits is too much. So then throw an extra 1 and 5 in the array to make it less obvious what you are doing. Okay, you do not like the array idea? Then use the approach of Nintendo’s copy of <a href="https://fireemblem.fandom.com/wiki/Random_Number_Generator"> Ice Badge </a> and take two numbers and average them. The point is; that you might need to change what your game does in the background to adjust to the perception the player has of the chance you are showing them.

<h2> Other thoughts for this. </h2>

As I mentioned before, this post was only about how Probability and the perception of it can affect your game. I would like also to write about how one can balance a game that has random elements. Another interesting topic would be how to actually generate random numbers (your computer cannot throw a dice, sadly). I also would like to write about how randomizer works! I hope to go into all those topics in due time.