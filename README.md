# Teleportation Arrow

In this course, we will explore how we can edit the outcome of an arrow being fired from a bow. In this case to teleport the player to the location of the arrow when it lands.

## You will need

* A text editor - such as Visual Studio Code
* A copy of Minecraft: Java Edition

## Prerequisites

You should already be familiar with the idea of resource packs, in-game commands and ticks. You should also know how to navigate to your Minecraft installation's working directory.

## Part 1: Creating a Data Pack

In Minecraft: Java edition, we work with 'data packs'.  
We will be using 'datapacks' to add content and override functions, loot tables, and much more without creating disturbances to the code of Minecraft.

So, to begin we need to create our world and a data pack which will be the space in which all of our code will be contained.

- Open Minecraft Java Edition and 'Create New World' - we can call it 'Teleport' for consistency, but you can call it whatever you like.
- The world needs to be made with `Allow Cheats: ON` & `Game Mode: Creative`.
- `Create New World` when all of these settings have been made.
Once the world has been created, we need to exit the world and go back to our world and access its world folder.
- Left Click on our world - `Teleport` and then select `Edit`.
- Select `Open World Folder`. This will open a folder list for our world. 
- We need to open the `data packs` folder which should be empty at this stage.
- Create a new folder, by right clicking in the blank space and call it `TPArrow`.
 Once we have created the 'data pack' we need to fill it with folders to operate the Teleportation Arrow.
- Inside `TPArrow` create a new folder called `data`.
 A data pack is identified by Minecraft with a `pack.mcmeta` file that is stored in the pack.
- With our text editor we need to create a new file called `pack.mcmeta` inside which we need the following:

```json
{
    "pack": {
        "pack_format": 5,
        "description": "Add arrows that teleport the player."
    }
}
```

**Be sure to save your progress to the correct location**

There are two properties within `pack.mcmeta` that warrant definition:

* 'pack_format: 5' describes the version of Minecraft that we are using, which for this example should be Minecraft 1.15.2.
* 'description' merely provides an overveiw of what our code is intended for.

Moving forward, we need to understand *namespaces*.
A namespace is a domain for a particular set of contents, which prevent things with the same name from interfering with each
other. For example, if a mod adds a new type of furnace with the block ID *furnace*, Minecraft would find a conflict between
the default furnace and our new one with the same name — and the game breaks. When we use different namespaces for the mod
and the vanilla furnace, the blocks become *minecraft:furnace* and *mod:furnace*, which no longer conflicts.

Lastly, we need to setup two more folders before we move on to part 2.

- In `data` we need to create a new folder called `minecraft`.
- Then, also within `data` create a new folder called `tp_arrow`. Using our namespace formula as above.
**Now we have set up our 'data pack' to begin inputting the particulars of our mod.**
 
## Part 2: Functions and Tags

We have created the framework for our Teleportation Arrow, now we need to fill out the body of the code to tell the
program what we need it to do.

Firstly, we will fill out the `minecraft`.

- In `minecraft` we have to create a new folder named `tags`.
 *Tags* are used by Minecraft to group like things together. They can group items, blocks and more. 
- In `tags`, we then create a new folder called `functions`. In Minecraft, *functions* are a way to group several commands
 together and run them all at once. Instead of typing each command into the chat window or chaining them together using
 command blocks, we can write each command as a line in a text file and add them to our game using data packs.
- Lastly we need to create a new File using our text editor, we will call it `tick.json`. Inside `tick.json` input the following code.

 ```json
 {
    "values": [
    "tp_arrow:tick"
    ]
}
```

**Be sure to save your progress to the correct location**

We need to work on the contents of `tp_arrow`, so navigate back to `data` and then `tp_arrow`.

- Create a new folder inside `tp_arrow` called `functions`.
- In our new functions folder we need to create a new file using our text editor called `tick.mcfunction`. This file will contain the majority of our code.

## Part 3: Coding

In `tick.mcfunction` we need to start typing our code. 
Note that functions do not use slashes at the start of commands.
Type:

```
execute
```

We want to run our command *as* a landed arrow, so we can add the `at` keyword next. **Add `at` to your command.

```
execute at
```

We want to target arrows that are in the ground.
To do this, we use `@e`, which is a *target selector* meaning *all entities*.
'Entities' are any object in Minecraft that isn't a block. For example, items, players, and animals are all entities.

Add `@e` to your command.

```
execute at @e
```

We can limit our selection by using *target selector arguments. 
We add these inside square brackets after the selector. 
The `type` selector can target entities like `zombie`, `ender_dragon` or `item`. 
We want to target spectral arrows. So add the following to your code.

```
[type=minecraft:spectral_arrow]
```

Your code should look like this:

```
execute at @e[type=minecraft:spectral_arrow]
```

We can also look at the *NBT data* of entities.
This stores extra information about the entity.
For example, chests use NBT data to store what items they hold and the direction they're facing.
For arrows, the `inGround` NBT tag stores wether or not the arrow is in the ground.

So we need to edit our code by adding:

```
,nbt={inGround:1b}
```

**Make sure that this is inside the close `]`.**

0 means that the arrow is in the air, while 1 means that the arrow is in the ground.
The `b` indicates that this is a *binary number*, meaning it has two values.
*The [Minecraft Wiki](https://minecraft.gamepedia.com/) lists the NBT data for different entities and blocks.*

Our code should look like this:

```
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}
```

Now that we have identified what we are firing (`spectral_arrow`) and where it will be (`inGround`), we now need to define what will happen at that point.

We used `@e` as a *target selector* for *all entities*.
Now we want to use the *target selector* for *nearest player* which is `@p`.
The `tp` command will give use the desired effect of teleportation.
We need to add the following code for these additions of the player and teleportation to occur.

```
as @p rotated as @p run tp ~ ~ ~
```

With the final result looking like this:

```
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
```

**Make sure to save your work as `tick.mcfuction` in the correct location. In the `functions` folder.

Now we need to run a command that will delete the arrow once our teleportation is complete.

This is done by running the `kill` command in our code.
We still need to define what is being removed, and when.

We also need to use the *target selector* `@s` which refers to the entity executing the command (yourself).
So on a new line of code in `tick.mcfunction` we need to add the following string of code.

```
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
```

With our code looking like this:

```
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
```

**Save your work.**

You should now be able to open up your Minecraft world, `Teleport`.

- Give yourself a `Bow`
- Give yourself a `Spectral Arrow`.
*And test your new mod. Shoot water, lava, ground and mobs to test.*

## Part 4: Fixing our code 

Now, having tested your mod, you will have noticed that it does not *teleport* when you target a mob (zombies, villagers or cows).
We need to customise our existing code to make these changes.

We need to add to the beginning of our code in the `tick.mcfunction` file.

Firstly we are going to use the *scoreboard objective* and *dummy* command to track which entities (mobs) have been hit by our spectral arrow. The *dummy* command determines that section can only be changed by code/commands.
We will call the new *sore board* objective `tpArrowHit`.

Our code will look like this:
`scoreboard objectives add tpArrowHit dummy`

Our next piece of code will be placed after all of code on line 4. We need to change our players location from the location of firing the arrow to the new location of the arrow.
We will again use `execute at @e` and combine our object identifier with the new location. 

It will look like this:
`execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}]` 

This next line tells all entities to store their result, and update their tpArrowHit scoreboard objective if they succeed in running the command to clear the glowing effect bestowed by the spectral arrow.

Our code will look like this:
`execute as @e store success score @s tpArrowHit run effect clear @s minecraft:glowing`

Our last part  will set the entity to run the command as the nearest player, and will use the `rotated as` subcommand to set the rotation to the same as the nearest playeras well. This will ensure that their rotation does not change after they have teleported.

This part "run tp ~ ~ ~" simply says to run the teleport command at the same relative position as the target that was already set, being the arrow.

It will look like this:
`execute at @e[scores={tpArrowHit=1}] as @p rotated as @p run tp ~ ~ ~`

The final result in our fixed result should look like this:
`scoreboard objectives add tpArrowHit dummy
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
execute as @e at @s store success score @s tpArrowHit run effect clear @s minecraft:glowing
execute at @e[scores={tpArrowHit=1..100}] as @p rotated as @p run tp ~ ~ ~`

**Save your work.**

You should now be able to open up your Minecraft world, `Teleport`.

- Give yourself a `Bow`
- Give yourself a `Spectral Arrow`.

*And test your new mod. Shoot water, lava, ground and mobs to test.*

**Congratulations on completing the Teleportation Arrow!**

**Make sure to save your file in the correct location and test your mod**
