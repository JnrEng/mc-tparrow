# Teleportation Arrow

In this course, we will explore how we can edit the outcome of an arrow being fired from a bow. In this case to teleport the player to the location of the arrow when it lands.

## You will need

* A text editor - such as Visual Studio Code
* A copy of Minecraft: Java Edition

## Prerequisites

You should already be familiar with the idea of resource packs, in-game commands and ticks. You should also know how to navigate to your Minecraft installation's working directory.

## Part 1: Creating a Data Pack

Just like resource packs (and behaviour packs in Bedrock Edition), data packs provide a way to customise Minecraft. Data packs can be used to override or add new advancements, functions, loot tables and more without changing any of Minecraft's code.

Data packs are stored in a world's `datapacks` folder as either a sub-folder or a ZIP file. **Create a new Minecraft world in creative mode with cheats enabled. Go to your Minecraft installation's working directory, then find your world's folder inside the `saves` folder. Open the world's `datapacks` folder, and create a new folder with a name of your choosing.** This is your data pack folder. During development, it's easier to work with datapacks as a sub-folder, then package them as a ZIP to distribute once you're done.

When we're working on files in data packs, there is no need to exit your world. Just pause the game and switch to another application.

Data packs are identified by Minecraft with a `pack.mcmeta` file stored in the data pack. **Create a new file in your data pack folder called `pack.mcmeta`, where `.mcemta` is the *file extension*.** This is a JSON file that contains a description and a version number. The file follows this format:

```json
{
    "pack": {
        "pack_format": 5,
        "description": "Add arrows that teleport the player"
    }
}
```

There are two properties in the `pack` object of this file:

* `pack_format`, which describes the version of the data pack format that the pack uses. In this course, we're using Minecraft 1.15.2, which uses data pack version 5.
* `description`, which is displayed when hovering over the data pack's name when using the `/datapack list` command in-game.

**Copy this code into `pack.mcmeta`, using a description of your choice.**

Moving forward, we need to understand *namespaces*. A namespace is a domain for a particular set of contents, which prevent things with the same name from interfering with each other. For example, if a mod adds a new type of furnace with the block ID *furnace*, Minecraft would find a conflict between the default furnace and our new one with the same name — and the game breaks. When we use different namespaces for the mod and the vanilla furnace, the blocks become *minecraft:furnace* and *mod:furnace*, which no longer conflicts.

**In your data pack, create a new folder called `data`. In this folder, create a new folder with the name `tp_arrow`.** Namespace names can only include numbers, lowercase letters, underscores and the hyphen/minus symbol. The convention for namespaces and names is `snake_case`. This means that all words are in lower case, and spaces are created with underscores.

## Part 2: Coding

In Minecraft, *functions* are a way to group several commands together and run them all at once. Instead of typing each command into the chat window or chaining them together using command blocks, we can write each command as a line in a text file and add them to our game using data packs.

We can define new functions by creating `.mcfunction` files. **In your namespace folder (`tp_arrow`), create a new sub-folder called `functions`.** This is where we can add our functions. These follow the same naming rules for namespaces.

To create a teleporting arrow, we need to tell every arrow that's landed in the ground to execute a `teleport` command whenever an arrow lands in the ground. Instead of making this work for all arrows, we're just going to give spectral arrows teleportation powers. Minecraft enables this functionality through the `execute` command. We'll build our command bit by bit.

Our data pack will work by checking if there are any arrows in the ground once per *game tick*. To start, we'll make a new function in our namespace that we want to run every tick. **In the `tp_arrow` folder, create a `functions` sub-folder, and add a new file called `tick.mcfunction`.**

**In `tick.mcfunction`, type `execute`.** Note that functions do not use slashes at the start of commands. We want to run our command *as* a landed arrow, so we can add the `as` keyword next. **Add `as` to your command.**

We want to target arrows that are in the ground. To do this, we use `@e`, which is a *target selector* meaning *a*ll *e*ntities. **Add `@e` to your command.** 'Entities' are any object in Minecraft that isn't a block. For example, items, players, and animals are all entities.

We can limit our selection by using *target selector arguments*. We add these inside square brackets after the selector. the `type` selector can target entites like `zombie`, `ender_dragon` or `item`. We want to target spectral arrows, so **add `[type=minecraft:spectral_arrow]` right after the `@e`.**

We can also look at the *NBT data* of entities. This stores extra information about the entity. For example, chests use NBT data to store what items they hold and the direction they're facing. For arrows, the `inGround` NBT tag stores wether or not the arrow is in the ground.

We can target only these arrows by adding `nbt={inGround:1b}` to our target selector arguments, separated from the type argument by a comma. 0 means that the arrow is in the air, while 1 means that the arrow is in the ground. the `b` indicates that this is a *binary number*, meaning it has two values.  The [Minecraft Wiki](https://minecraft.gamepedia.com/) lists the NBT data for different entities and blocks. **Add `,nbt={inGround:1b}` to your command, inside the square brackets.** Your code should look like this:

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}]
```

Now that we have identified what we are firing (`spectral_arrow`) and where it will be (`inGround`), we now need to define what will happen at that point.

We used `@e` as a *target selector* for *all entities*. Now we want to use the *target selector* for *nearest player* which is `@p`. The `tp` command will give use the desired effect of teleportation. We need to add the following code for these additions of the player and teleportation to occur.

```mcfunction
as @p rotated as @p run tp ~ ~ ~
```

With the final result looking like this:

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
```

**Make sure to save your work as `tick.mcfuction` in the correct location. In the `functions` folder.

Now we need to run a command that will delete the arrow once our teleportation is complete. This is done by running the `kill` command in our code. We still need to define what is being removed, and when.

We also need to use the *target selector* `@s` which refers to the entity executing the command (yourself). So on a new line of code in `tick.mcfunction` we need to add the following string of code.

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
```

With our code looking like this:

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
```

**Save your work.**

You should now be able to open up your Minecraft world, `Teleport`.

* Give yourself a `Bow`
* Give yourself a `Spectral Arrow`.
*And test your new mod. Shoot water, lava, ground and mobs to test.*

## Part 3: Functions and Tags

We have created the framework for our Teleportation Arrow, now we need to fill out the body of the code to tell the program what we need it to do.

Firstly, we will fill out the `minecraft`.

* In `minecraft` we have to create a new folder named `tags`. *Tags* are used by Minecraft to group like things together. They can group items, blocks and more.
* In `tags`, we then create a new folder called `functions`. In Minecraft, *functions* are a way to group several commands together and run them all at once. Instead of typing each command into the chat window or chaining them together using command blocks, we can write each command as a line in a text file and add them to our game using data packs.
* Lastly we need to create a new File using our text editor, we will call it `tick.json`. Inside `tick.json` input the following code.

 ```json
 {
    "values": [
    "tp_arrow:tick"
    ]
}
```

Be sure to **save your progress to the correct location**.

We need to work on the contents of `tp_arrow`, so navigate back to `data` and then `tp_arrow`.

* Create a new folder inside `tp_arrow` called `functions`.
* In our new functions folder we need to create a new file using our text editor called `tick.mcfunction`. This file will contain the majority of our code.

## Part 4: Working with Mobs

Now, having tested your mod, you will have noticed that it does not *teleport* when you target a mob (zombies, villagers or cows).
We need to customise our existing code to make these changes.

We need to add to the beginning of our code in the `tick.mcfunction` file.

Firstly we are going to use the *scoreboard objective* and *dummy* command to track which entities (mobs) have been hit by our spectral arrow. The *dummy* command determines that section can only be changed by code/commands. We will call the new *sore board* objective `tpArrowHit`.

Our code will look like this:
`scoreboard objectives add tpArrowHit dummy`

Our next piece of code will be placed after all of code on line 4. We need to change our players location from the location of firing the arrow to the new location of the arrow. We will again use `execute at @e` and combine our object identifier with the new location.

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

* Give yourself a `Bow`
* Give yourself a `Spectral Arrow`.

*And test your new mod. Shoot water, lava, ground and mobs to test.*

## Done

That's it! You've now got an arrow that will teleport you to wherever it lands.

If you want to take it further, you could try:

* Teleporting another (random) player
* Teleporting nearby mobs with the player
