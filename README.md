# Teleportation Arrow

In this course, we will explore how we can edit the outcome of an arrow being fired from a bow. In this case to teleport the player to the location of the arrow when it lands.

You'll learn how to create new functions and tags, how to use scoreboards to store data and how to execute commands as other entities.

## You will need

* A text editor
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

## Part 2: Creating Functions

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

We've added an `execute` command that will run at every spectral arrow that's landed in the ground. Now, we need to add the command we want to run at the spot where that arrow lands.

We used `@e` as a target selector for all entities. Now we want to use the target selector for *nearest player*, which is `@p`. We'll tell the nearest player to the arrow to teleport themseles to that location. **Add `as @p` to your command** to execute it *as* the player. By default, the player will be telported facing the same way as the arrow. We can keep the player's rotation using the `rotated as` argument. **Add `rotated as @p` to your command.** Your code should look like this:

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p
```

Now we can add the command to execute. **Add `run tp ~ ~ ~` to your command.** This will tell the player to *run* the `tp` (teleport) command, and teleport them to the *execution location*. We know from earlier that this location is the arrow, since we specified it with the `at` keyword. Your code should looks like this:

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
```

If we used this function right now, Minecraft would telport the player every tick until the arrow despawned.  We need to make sure we remove the arrow after we've teleported the player.

To do this, we can get the arrow to execute the `kill` command on the same tick. We've already written the code to select our arrow, so **copy-and-paste the command from line 1 onto a new line.** We can just change the command at the end of the line. **Remove `as @p rotated as @p run tp ~ ~ ~` from the end of the second command, and replace it with `as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s`.** The `as` keyword makes sure we're telling the arrow to destroy itself (instead of the player), and the `@s` selector targets the *executor* of the command — in this case, it's the arrow that we selected with the `as` keyword. Your code should look like this:

```mcfunction
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
```

## Part 3: Overriding Tags

We've finished our function, but we need to tell Minecraft that we want it to run every game tick. To do this, we can add our function to the built-in `tick` tag. *Tags* are used by Minecraft to group like things together. They can group items, blocks and more. Functions or commands that we list in this tag will run once every game tick.

**In your data pack folder, create a new folder called `minecraft`.** Items we put in this folder wil override any built-in resources. We're going to override the `tick` tag. Any functions that are part of this tag will run once per game tick.

To do this, we need to add a new definition for the `tick` tag in the `minecraft` namespace. **In the `minecraft` folder, create a new sub-folder called `tags`, with a sub-folder in that called `functions`. Create a new file in this folder called `tick.json`.** The file follows this format:

 ```json
 {
    "values": [
    "tp_arrow:tick"
    ]
}
```

**Copy this code into `pack.mcmeta`.**

Minecraft reads our data packs once when we load a world. Any changes we make after that won't be seen until we close and re-open the world or use the `/reload` command. **Save the files, then go back to your game and type `/reload` into the chat window.** You should see the message `Reloading!` appear. **Now, try shooting a spectral arrow.** When the arrow lands, it should teleport the nearest player to the location where it landed, and then disappear.

## Part 4: Fixing Arrow Behaviour

Our arrows work well if we shoot them at the ground or a wall, but they currently don't work if we shoot a mob directly. This is because the arrow never lands if it hits a mob — it simply gives them the `minecraft:glowing` status effect, so our function doesn't target them. Because of the unique behaviour of the arrow, however, we can use this to execute our function on mobs with the glowing status effect.

In this section, we'll be using scoreboards to keep track of mobs. Scoreboards are an advanced feature of Minecraft that allow map makers to disaply information in different areas of the game, to target specific entities and store different kinds of data.

Our first step will be to make a new *objective*. Entities can gain *points* in an objective. To create a new objective, **add `scoreboard objectives add tpArrowHit dummy` to the top of `tick.mcfunction`**. This command will add a new objective called 'snowed' to our game, and make it a 'dummy' type. This means that the objective will not score anything (like monsters destroyed or blocks placed) unless we use commands to change the score. This will run every tick; if the objective already exists, Minecraft will ignore the command.

Next, we'll tell every entity to attempt to remove the glowing status effect from themselves. This will be successful if they have the glowing effect, which they can receive from being hit by a spectral arrow. If they succeed, we'll tell the entity to add themselves to the 'tpArrowHit' objective. This will make it much easier to target them later.

We'll start with a new `execute` command. **In `tick.mcfunction`, add a new `execute` command to the end**. We want every entity to try to remove the status effect, so **add `as @e` to the command**. Each entity will try this at their own location, so **add `at @s` to the command**.

The `execute` command allows us to store the result of commands we run in different places. We can do this with the `store` keyword. **Add `store` to the command**. We want to store wether or not our command is successful, so the next argument will be `success`. **Add `success` to the command**.

Next, we specify where we want to store the result. In our case, we want to store it in a scoreboard objective, so **add `score` to the command**. The `score` needs a `name` and an `objective` argument. For `name`, we can use the executing entity (`@s`), and for the objective, we use `tpArrowHit`, which we created earlier. **Add `@s tpArrowHit` to the command**.

Finally, we can add the command we want each entity to attempt. To remove the glowing effect, we use the same command we used to give it, but use `clear` instead of `give`. **Add `run effect clear @s minecraft:glowing` to the command**.

Your command should look like this:

```mcfunction
execute as @e at @s store success score @s tpArrowHit run effect clear @s minecraft:glowing
```

Now we have a list of entities, as a scoreboard objective, that contains all the entities which had the glowing effect. Remember that we told every entity to remove the glowing effect, so they won't glow anymore. Our last step will tell every entity with a score in `tpArrowHit` to teleport the nearest player to them.

**In `tick.mcfunction`, add a new `execute` command**. We can target entities based on their scores using the `score` target selector argument. **Add `at @e[scores={tpArrowHit=1}]` to the command**.

This will target every entity who has scores matching the list we provide (remember that entities can have scores for more than one objective). We specify the 'tpArrowHit' objective, then say `1`. Just like with the arrow, we want the entity to teleport the nearest player to them, so add **`as @p rotated as @p run tp ~ ~ ~` to the command**. Your command should look like this:

```mcfunction
execute at @e[scores={tpArrowHit=1}] as @p rotated as @p run tp ~ ~ ~
```

That's all we need to add. Your finished function file should look like this:

```mcfunction
scoreboard objectives add tpArrowHit dummy
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @p rotated as @p run tp ~ ~ ~
execute at @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] as @e[type=minecraft:spectral_arrow,nbt={inGround:1b}] run kill @s
execute as @e at @s store success score @s tpArrowHit run effect clear @s minecraft:glowing
execute at @e[scores={tpArrowHit=1..100}] as @p rotated as @p run tp ~ ~ ~
```

**Save your file and switch to Minecraft. Run `/reload` to load the new function then try shooting a spectral arrow at a mob.** When the arrow hits the mob, the nearest player should be teleported to the location the arrow landed at as normal.

## Done

That's it! You've now got an arrow that will teleport you to wherever it lands.

If you want to take it further, you could try:

* Teleporting another (random) player
* Teleporting nearby mobs with the player
