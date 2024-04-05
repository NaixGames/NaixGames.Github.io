---
title: "A tale down the archipelago part II: How I got Hades into the multiworld" 
layout: post
date: 10-04-2024

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

<h2> Reminders and requirements </h2>

As I said in the last entry, I have been working to add Hades to Archipelago multi-world randomizer. If you read said entry, now you should know 
what those words mean more or less. In this entry, I am going to write how you could add your own favourite game to this same system. Following
the last entry you will know we need to work on 4 elements; Server, Generation, Client and the Game itself. I will explain how to work
on each of these elements from a high-level point of view. For low level, I redirect you to the  <a href="https://github.com/ArchipelagoMW/Archipelago/tree/main/docs"> Archipelago documentation </a>.

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
is generated and is normally just responsible of calling methods from the other files to generate the information of its particular game.

The Items.py file is responsible for codifying all the items of the game as a number. It needs to be a number because the server stores
each item not as a string (so not by its name), but rather as an integer number. Note each item in ALL the multi-world games should be different to
ensure compatibility because each one is stored on the server when the games are played (Note: there is a series of technical limitations that
make this the standard, but it is outside the scope of these notes. Investigate in Google if you want!). The locations.py file does the same as Items.py
 but with locations. Each location is then encoded as an integer, so it can be stored in the server.

Note at this point items and locations are only a set of integers, without any logic or "real information" or your game attached to them. To 
add more information to the location we get the Regions.py file. This file will aggregate locations to represent "places" in your game. For example,
looking back at the last entry, the game MonPoke has a region the first town, which has 3 locations. The file Rules.py will enforce
certain logical contains between items, locations and regions that are respected. For example, using MonPoke again, it will enforce that the
location behind the tree can only be accessed after Cut is obtained.

As an example, an implementation of the game MonPoke, an implementation for the first town would look something like this (simplifying some details and
imports):

{% highlight ruby %}

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

Como mandar paquetes de datos.
Como recibir paquetes de datos.
Dar de ejemplo como lo hice en hades.

<h2> Connecting the client and your game </h2>

"Observer pattern" 
Recomendar tener cliente y "juego" separado.
Dar de ejemplo como lo hice en hades.


<h2> Some personal advice </h2>


Separar sistemas en partes pequeñas.
Pedir ayuda / colaborar (Bo + Silviris).
Testear con la comunidad (los tkms)