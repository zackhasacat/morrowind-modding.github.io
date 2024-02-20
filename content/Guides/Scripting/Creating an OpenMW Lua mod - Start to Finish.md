
#OpenMW-Lua 


To begin, let’s go over the tools I’d recommend that you use for this.

I use visual studio code, but you can use any text editor. I like code for its project view and lua language server, mostly to show syntax errors.

I’d also recommend installing git and using it to host your mod on github or gitlab, but that’s beyond the scope of this tutorial.

Note, you can only run lua mods in OpenMW 0.48 and development builds for 0.49. This tutorial will assume you are running OpenMW 0.49.

Let’s start off with a development environment. I like to have a folder with all of my projects inside of it. I’ll make one inside of that, and I’ll name the folder FreeGoldForMages.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image001.png)
Now, I’ll open this folder in visual studio code. Before we go any further, make sure you have the lua extension installed. You can search for it over on the extensions tab.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image002.png)

Let’s take a look at the lua documentation. Note, that this site defaults to the development documentation. If you’re making a mod for OpenMW 0.48, you may find it useful to only show functions that are added to that version. You can switch right here.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image004.png)

The documentation for Lua is in two main sections. The first section “Overview of Lua Scripting” is a tutorial, that goes over a lot of what I’m showing today and more, and the lower section the reference for which functions that work with the game are available to you. I would read the first section in its entirety. Now, going to [Lua scripts naming policy](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/overview.html#lua-scripts-naming-policy), let’s follow it, and create a few lua files.

We’ll make the scripts file, and inside of it, the FreeGoldForMages folder, and inside of it, let’s put two files, FGFM_p.lua, and FGFM_g.lua. That’s not a standard, just what I like to use for my filenames, you can follow whichever you’d like.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image005.png)

Now, let’s create our omwscripts file. This will define what scripts are attached to the game.

We’ll put in the two we created, checking [Format of .omwscripts](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/overview.html#format-of-omwscripts), we’ll want to put in the following:

![
](Assets/Guides/ZHAC_LuaGuide/clip_image006.png)

Note that one starts with PLAYER, and the other with GLOBAL. If you check the [Format of .omwscripts](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/overview.html#format-of-omwscripts) page, it’ll provide more information, but OpenMW scripts must have a context, and that can be player, global, local, or menu. Menu is only available in recent 0.49 builds.

You can have scripts that aren’t listed in this file, but you’ll need to use [require](https://www.lua.org/pil/8.1.html) on them. Additionally, any file required will have the same context as the file that’s requiring it.

For example, if we look at the API reference, and go over the [engine handlers](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/engine_handlers.html), it’ll say which can be used by which script type.

Now, I want to create a mod that places a button that gives the player money, but only if they are part of the mages guild.

Most of this will take place in our global script.

For example’s sake, I’ll replace any chair that has the record ID furn_de_r_Chair_03 with our button ex_gg_gateswitch_01, but I only want to do it in the Balmora guild of mages.

We can use local variables to store these values. We can define them at the top of the file.

Let’s create a function to help with this. I’ll name it replaceChairWithButton(chair)

Now, I think it would be useful to also place a button without any replacement, so we’ll split this into two functions, the second of which is named placeButton(cell,pos,rot).

Inside of our first function, we can call the second function. But note the order, I can only use a function from the same file if it’s above the function I’m currently in. You can get around this with tables, or interfaces.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image007.png)

Since I want to replace any chair with it, we’ll want to use an engine handler.

At the bottom of the script, we’ll put return {}, and inside that table we can specify our interface, engine handlers, and events.

For now, we just want the engineHandler onObjectActive. You just need to add the engineHandlers table to your empty table from before. Inside of here, we can define a function to be used for the specified engine handler.

![A computer screen shot of a computer
](Assets/Guides/ZHAC_LuaGuide/clip_image008.png)

As the documentation specifies, the first parameter is the object that became active.

Game objects have several useful bindings. These are listed in the [core API page](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html).

Please note, where it says type, that does not mean it’s the name you use in scripting, for example you’ll never type GameObject quite like this, it’s just the name of the object type.

We will check if the record ID matches the object we want. Note, there is another id property here, it is the objects actual ID, unique to that object, as opposed to the record ID, which is the same for all the chairs of this type.

We also want to check if the cell matches.

I only want this to happen in the Balmora, Guild of Mages cell, so let’s add a check here. If we go back to our [gameObject document](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(GameObject)), we’ll see the cell property. If we click that, then click #cell, it will take us to the [definition for that object.](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(Cell)) Here we find the name property, which is all we care about.

So, we check if that record ID matches, and that the cell name matches, and if it does, we’ll want to create our button, by calling the empty function from before. Make sure that you pass “obj” in the function.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image009.png)

To place the button, we’ll need to add a require for world at the top of the file. Every package reference file has this at the top, you can copy paste from there.

[World](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_world.html) is a package only usable by global scripts. Global scripts have the most power in OpenMW Lua, but some other functions are only available to player, local, or menu scripts.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image010.png)

To create the button, we’ll use the [world.createObject()](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_world.html##(world).createObject) function. As it says, the parameter needs to be the record ID, which will be “ex_gg_gateswitch_01”. There is another parameter for count, but it only matters when you are creating a stack.

We’ll create the button and pass it to a variable. Now, the object is created, but it isn’t anywhere that the player can reach. We’ll need to move it somewhere it can be used, specifically to the chair.

![ program
](Assets/Guides/ZHAC_LuaGuide/clip_image011.png)

We can use [teleport](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(GameObject).teleport), since it is a property of GameObject. We’ll pass the variables provided by the chair for this. Cell, position, and rotation.

In this case, cell is a [Cell](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(Cell)) object, and position is a [vector3](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_util.html##(util).vector3).

If you wanted to specify your own position and cell, you could do so like this.

For cell, you can just provide a cell name. Or, use one of the world functions to find the cell.

  
And for position, you can create one from the util package, just util.vector3(1,2,3)

![ program
](Assets/Guides/ZHAC_LuaGuide/clip_image012.png)

But right now, we’ll just use the provided cell, location, and rotation.

Now, we’ll call remove on the chair object, which deletes it. It’s important that we do this after teleporting the object. It’s also important that you do chair:remove(), with the colon, not a dot, as explained by [this page.](https://www.lua.org/pil/16.html) We’ll also update the call to placeButton to provide the [position](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(GameObject).position), [rotation](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(GameObject).rotation), and [cell](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_core.html##(GameObject).cell) of the chair.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image013.png)

Let’s open the game to see our work so far. Make sure you have your files saved. But we don’t have the mod added yet, so let’s add the directory we created to the files tab.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image014.png)

You’ll also want to enable the omwscripts file in that directory.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image015.png)

If you’re testing your mod, it’s very useful to enable Skip Menu, and specify a default cell.

  
![
](Assets/Guides/ZHAC_LuaGuide/clip_image016.png)

Go to the Balmora Guild of Mages, and go downstairs, to the tables.

Here they are. But if we activate them, it does nothing. We’ll want to go back and add that functionality.

![A screenshot of a video game
](Assets/Guides/ZHAC_LuaGuide/clip_image017.png)

Now, to be able to detect activation of the buttons, we’ll use [Script interfaces.](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/overview.html#script-interfaces) These can expose scripts to other scripts. These can only be used within the same type or the same object, however. Add the require to your file.

We don’t need to create an interface right now, but we want to access another interface, the [Activation Interface.](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/interface_activation.html) It can only be used by global scripts.

We can access it with the interfaces package, as if it was a sub package, such as: interfaces.Activation

Specifically, we want to use [**Activation.addHandlerForType(type, handler)**](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/interface_activation.html#%23(Activation).addHandlerForType).

We’ll also need to add the [types](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_types.html) package to use this. We can call the function at the bottom of the file and add our activateButton function right above it.

![ program
](Assets/Guides/ZHAC_LuaGuide/clip_image018.png)

Now, we’ll do a lot of work from before. We need to check that the record ID and cell match. Then, we will create the gold. This time, we specify the count.

![ program
](Assets/Guides/ZHAC_LuaGuide/clip_image019.png)

But hold on, we don’t want to add the gold unless they’re a part of the Mage’s Guild. If we go over the [types](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_types.html) package, under NPC, we’ll find [getFactionRank](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_types.html##(NPC).getFactionRank), which can be used to check if the player is in the faction.

Now, let’s take a moment to talk about nil checks. If the player is not in this faction, it will return either nil, or 0, depending on their version. Any time you are using a nested table, and it has the possibility of containing nil values, you should check each value is not nil before continuing.

After those changes, this is our code:

![A screen shot of a computer code
](Assets/Guides/ZHAC_LuaGuide/clip_image020.png)

It’s still incomplete. As before, we created the gold, but haven’t moved it anywhere.

This time however, we will move it into the player’s inventory. This part is very easy.

We can simply use GameObject:moveInto. The parameter here can be an inventory or a gameObject, as the page says.

There’s one more thing to do. We want to have the player have a popup on their screen when activating this button. You can do this with the [ui package](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_ui.html). But, the ui package is only available to menu and player scripts, and we’re working on a global script. We’ll need to use events. We’ll use the GameObject:sendEvent on the player. We’ll pass a name for the event, and the event payload, in this case, the message we want the player to see. We’ll also send a message telling the player they’re not eligible, if that’s the case.

  
The event name can be whatever string you want, but you’ll want it to be unique, as to not trigger unrelated scripts accidently.

For now, it does nothing. But here is what the script looks like

![
](Assets/Guides/ZHAC_LuaGuide/clip_image021.png)

Now, we can finally get started on the player script we created back in the beginning.

First, put in the require for the ui package we couldn’t use before.

Then, return the eventhandlers table, containing a function with the same name as the event before.

In that function, use the [ui.showMessage()](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_ui.html##(ui).showMessage) function, passing the message to it.

![
](Assets/Guides/ZHAC_LuaGuide/clip_image022.png)

It’s a very short script! This is one way to format it. You might also do this:

![ program
](Assets/Guides/ZHAC_LuaGuide/clip_image023.png)

You can simply define the table at the start of the file, and add to it, then return the table at the bottom. You should use this, or some other term for this. I would avoid self, to eliminate possible confusion with the [self](https://openmw.readthedocs.io/en/latest/reference/lua-scripting/openmw_self.html) package.

Your mod should now be fully functional. Let’s try it out. Open the game, run to the button, and press it.

Uh oh, it doesn’t do anything. Hit F10 in game or review your game output in the command line. It’ll look like this:

![
](Assets/Guides/ZHAC_LuaGuide/clip_image024.png)

  
The log is very useful, it will tell you what’s wrong in case of errors. In this case, it’s expecting a string, but I forgot to provide it. It tells me the line of the function that failed, and the line that triggered the error, in this case, 19. I forgot to provide the faction to check the rank for.

![ program
](Assets/Guides/ZHAC_LuaGuide/clip_image025.png)

We can see line 19, and we just need to provide the “mages guild” faction name. In addition, I’ll return false, to prevent the normal script or game event from happening, such as preventing it from being picked up.

If you didn’t close your game before, you don’t need to. You can open the console, and type in “reloadlua”, and it will reload the file, provided you remembered to save it.

Give it a shot, and the button should react according to your membership!

Now, see if you can make your own script. If you need help with that, come visit the [OpenMW Discord](https://discord.gg/YegsVV3aRb), and ask in the #lua channel! And, let me know what tutorials you’d like next.