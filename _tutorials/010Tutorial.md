---
title: "A tale down the archipelago part II: How I got Hades into the multiworld" 
layout: post
date: 07-04-2024

image: 
  path: tutorials/images/009AP.png 
  thumbnail: tutorials/images/009AP.png
  caption: "A pixel art of the symbol of Archipelago."
  categories:
     -intermediate
     -coding
     -probability
  tags:
     -intermediate
     -coding
     -probability
---

The final step into the multiworld.

<h2> Reminders and requirements </h2>

As I said in the last entry, I have been working to add Hades to Archipelago multi-world randomizer. If you read said entry, now you should know 
what those words mean more or less. In this entry, I will write how you could add your favourite game to this same system. Following
the last entry you will know we need to work on 4 elements; Server, Generation, Client and the Game itself. I will explain how to work
on each of these elements from a high-level point of view. For low level, I redirect you to the <a href="https://github.com/ArchipelagoMW/Archipelago/tree/main/docs"> Archipelago documentation </a>.

<h2> A remidner of the multiworld server </h2>

The server, as explained before, is responsible for sending and receiving information related to locations and items between players.
It also stores information related to how the items are randomized.  It will store the locations that players have notified as visited and the items already obtained. 
It can also store information by request, so you could potentially ask the server to store some information (such as keeping track
 of some items that allow the game to be considered finished, or something along those lines).

If you are implementing a game into Archipelago (from now on AP for short), you will not program anything related to the server. Rather, you will 
implement methods that communicate with it on the Client (more information below).

<h2> How to generate the information for your game </h2>

To generate the information for the server, we need to supply information about your generation to the generator that Archipelago uses.
Normally this is done in the .apworld file, which is just a renamed .zip file that contains a bunch of different Python files. Which files are there
and the structure of each one will vary between different games, depending on the needs of each one. But normally, they all have a __init__.py,
 Items.py, Locations.py, Options.py, Regions.py and Rules.py files. __init__.py is the file that is triggered when a multiworld containing this game
is generated and is normally just responsible for calling methods from the other files to generate the information of its particular game.

The Items.py file is responsible for codifying all the items of the game as a number. It needs to be a number because the server stores
each item not as a string (so not by its name), but rather as an integer number. Note each item in ALL the multi-world games should be different to
ensure compatibility because each one is stored on the server when the games are played (Note: there is a series of technical limitations that
make this the standard, but it is outside the scope of these notes. Investigate in Google if you want!). The locations.py file does the same as Items.py
 but with locations. Each location is then encoded as an integer, so it can be stored in the server.

Note at this point items and locations are only a set of integers, without any logic or "real information" or your game attached to them. To 
add more information to the location we get the Regions.py file. This file will aggregate locations to represent "places" in your game. For example,
looking back at the last entry, the game MonPoke has a region the first town, which has 3 locations. The file Rules.py will enforce
certain logical contains between items, locations and regions that are respected. For example, using MonPoke, will enforce that the
location behind the tree can only be accessed after Cut is obtained.

As an example, an implementation of the game MonPoke, an implementation for the first town would look something like this (simplifying some details and
imports):

{% highlight python %}

#Item.py class starts here --------------

class ItemData(typing.NamedTuple):
    code: typing.Optional[int]
    progression: bool #This is used to determine if an item is mandatory for finishing the game.
    event: bool = False

monpoke_base_item_id = 37589470697

#This table would be use by __init__ to store the items.
item_table_pacts: Dict[str, ItemData] = {  
    "Potion": ItemData(monpoke_base_item_id, False),
    "Cut": ItemData(monpoke_base_item_id+1, True),
    "TrainPass": ItemData(monpoke_base_item_id+2, True),
}

#Item.py class ends here --------------


#Locations.py class starts here --------------

monpoke_base_location_id = 37589470697

#This table would be use by __init__ to store locations
location_table = {  
    "PotionOriginalLocation": monpoke_base_location_id,
    "CutOriginalLocation": monpoke_base_location_id+1,
    "TrainPassOriginalLocation": monpoke_base_location_id+2,
}

#Locations.py class ends here --------------


#Regions.py class starts here --------------


#This function would be used by AP to create the regions
def create_regions(ctx, location_database):
    #the following method creates a region and add it to AP database.
    #We create a "Menu" location because the AP always assume you start in a regions with that name
    ctx.multiworld.regions += [create_region(ctx.multiworld, ctx.player, "Menu", None, ["Menu"])]   

    #This create a region, first town, that has an exit, TrainStation
    ctx.multiworld.regions += [create_region(ctx.multiworld, ctx.player, "FirstTown", None, ["TrainStation"])] 

    #This says to the generator that you can get from the Menu to the FirstTown
    ctx.multiworld.get_entrance("Menu", ctx.player).connect(ctx.multiworld.get_region("FirstTown", ctx.player))  

#Regions.py class ends here --------------

#Rules.py class starts here ---------------

#This is a function AP uses to determine the logic rules in the game. How this exactly looks for you will vary a lot.
#Recommend looking at the documentation to see how this works
def set_rules(world, player):
    #This codifies that the player can only get the item behind the tree in the first town if they get cut.
    set_rule(world.get_location("TrainPassOriginalLocation",player), lambda state: state._has_("Cut", player))


#Rules.py class ends here -----------------

{% endhighlight %}

there are a couple of methods I didn't explain, but I hope their names are explicit enough for you to deduce what they do. I hope
my comments also help to bridge some details that are missing there.

Before moving to the next section, I must talk about the forgotten file Options.py. This one will contain all the options you might
want to tweak for your game. It is normally quite specific and something you will normally start thinking about after gaining a bit more
understanding about how the multi-world randomizer works, so I redirect to the documentation in case you want to know more.

<h2> Communicating the game and the server; the client </h2>

Now the glue that communicates your game with the server; the Client. As I already said before, the idea of the Client is that it serves
as a bridge between the server and the game, sending information between each other.

First, you might ask yourselves in which language you will code the client ... and well, it depends. It depends on what your game
modding scene looks like and what tools it can give you. The Client needs to receive and send information from the game, which
means you need to be able to add some code that makes those two talk. 

In the case of Unity games, normally you can inject
mods using Bepinex. Using that you can already instal a client template for C# games (the .NET client available
 <a href="https://github.com/ArchipelagoMW/Archipelago/blob/main/docs/network%20protocol.md"> here </a>). That is the best-case scenario;
in which you can easily inject an existing client template without complications. Some other cases will be a bit more involved. In the case of Hades,
I couldn't inject a Client directly in the game, due to the version of the Lua language of the game not accepting ejernal library injections. 
The community found a way around using StyxScribe, which in lose terms, allows the the game to communicate with external
Python programs. From there I could communicate the game with the Python client template. Again, what works for you is going
to be specific to the game you are working on, so I push you to talk to the game modding scene and see what is available to you.

Once you get your Client working and communicating with your games, you might ask; how do I communicate this with the server?
Well, you normally send packages. This is not different from sending strings in a certain format to the server. For example,
to unlock a location, you use something like this (on a Python Client).

{% highlight python %}

async def send_location_check_to_server(self, locations):
    payload_message = []
    #the next should convert locations from name to ids, which is what the server needs.
    sendingLocationsId += [self.location_name_to_id[locations]] 
    payload_message += [{"cmd": "LocationChecks", "locations": sendingLocationsId}]
    asyncio.create_task(self.send_msgs(payload_message))

{% endhighlight %}

You can also request information from the server. The server allows certain tags to request information. For example, if you want to request
the items that are in a certain location (to show them displayed in a store, for example), you could do something like this:

{% highlight python %}

def request_location_to_item_dictionary(self, request):
    asyncio.create_task(self.send_msgs([{"cmd": "LocationScouts", "locations": request, "create_as_hint": 0}]))
{% endhighlight %}

If you want more detail I refer to the <a href="https://github.com/ArchipelagoMW/Archipelago/blob/main/docs/network%20protocol.md"> network API documentation </a>.

<h2> Moding the game so it talks to the client </h2>

Once you get the client working, you will need the messages from the Client to the game do something. That is, you will
need to mod the game so you execute code to unlock items, or change the effect of checking a location.

Note that, here, you need to be realistic about what you can do. If the modding scene of your game does not exist,
then it is more likely there is a reason for this. It might be that the game uses a custom engine, so it is impossible
to inject code into it (at least for now!). I urge you to do your research on this one.

Once you get a really simple mod working in your game (an equivalent to "hello world") you will need to do some 
code that allows intercepting location checks and sending this to the server. For example, in Hades, I do the 
following to change the behaviour of unlocking a Keepsake check:

{% highlight lua %}

--this is a mod that allows intercepting a function and replacing it with something else
--we intercept the function that increases the gift meter of gods, unlocking a keepsake
ModUtil.Path.Wrap("IncrementGiftMeter", function (baseFunc, npcName, amount)
    --if the option for keepsakes is off, then return the normal behaviour
    if GameState.ClientGameSettings["KeepsakeSanity"] == 0 then
        return baseFunc(npcName, amount)
    end

    --Give the location check for unlocking the keepsake
    --this normally just sends a message to the Client that the location check should be processed
    PolycosmosEvents.ProcessLocationCheck(cacheNPCName, true)

    -- If the keepsake item is unlocked, return the base behaviour.
    if (GameState.Gift[npcName].Value>0) then
        return baseFunc(npcName, amount)
    end
end)
{% endhighlight %}

With that, we can avoid giving a keepsake, and give a request to the server. In the case we receive an item from the
server we need to unlock the item in the game. In Hades, I do something like this to unlock a Keepsake when it is
obtained as an item from the server:

{% highlight lua %}

function PolycosmosKeepsakeManager.GiveKeepsakeItem(item)
    --we parse the item from the server to the format expected in-game
    gameNPCName = KeepsakeDataTable[item].HadesName

    --this can only be if we already got the keepsake
    if (GameState.Gift[gameNPCName].Value>0) then
        return
    end
	 
    --set the friendship value to 1. This unlocks the keepsake.
    GameState.Gift[gameNPCName].Value = 1
    
    --say to the player they got the keepsake
    PolycosmosMessages.PrintToPlayer("Received keepsake "..item)
    --save to avoid losing the item.
    SaveCheckpoint({ SaveName = "_Temp", DevSaveName = CreateDevSaveName( CurrentRun, { PostReward = true } ) })
    ValidateCheckpoint({ Valid = true })

    Save()
end
{% endhighlight %}

Again, what this code looks for you will depend on how your game works. I hope at least this gives you an idea
of what you should think about doing.

<h2> Some personal advice </h2>


As a closing remark, I will give you some personal advice. Doing a mod like this is game development,
meaning it is hard, long and sometimes really stressful work. But it can also be really fun and rewarding.
The next points are just intended to get more of the latter and less of the former.

First, in terms of software architecture, try to research a bit about good software architecture if you haven't.
Your mod doesn't need to be perfect in this regard, but it will help to keep different systems on different
files, and avoid mixing your Client and Game systems too much. That way, when a bug occurs, it will be much easier
to debug and will imply rewriting less of your systems.

Having collaborators and help from the community can do wonders for your progress. I have the help of Bo,
who has researched the Hades base codebase much faster than I have, which has helped me write code for the mod much faster.
He even did some parts of the mod himself. I also had the help of Silviris, who rewrote some code of the .apworld to get it to a much
better format and order. Without people like this helping, I wouldn't have made the progress I have done today.

Test your mod. And not when it is finished, test it constantly. Doing all the hardcore without getting any 
feedback can be incredibly depressing. Having people give feedback and participate in the construction of the
mod turns it into a much better experience. They can also be an incredibly good bug testing unit. Just make sure
to give them tools to report bugs to you (like a way for them to send you bug logs).

Finally, good luck! Doing this can be an incredibly good learning experience. Enjoy it :)