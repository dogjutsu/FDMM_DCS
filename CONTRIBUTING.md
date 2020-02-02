# Contributing

Please feel free to reach out to NR before contributing.

## Building

### Downloading

Make sure to clone this repository into a "Saved Games/DCS/Missions/FDMM" folder so that you can dynamically debug scripts out-of-the-box, or at the very least make a symlink of some kind from this location to the folder you're using for development.

### Getting LDT's 'Build All' to work

While ability to 'Build All' isn't required for development, here's how you can make yours work.

Since DCS users love keeping things to the book, it follows common practice to compile all our various Lua sript files down into one master script file for release. While we accomplish this, we do so with some added complexity.

The building of all of FDMM's various Lua scripts, combined, down into one master file is currently being handled by [node-lua-distiller](https://github.com/yi/node-lua-distiller), which is a CoffeeScript script that accomplishes this task rather nicely.

Unfortunately, due to how this process's scanner works, it is limited to only the require()'s it finds in a single file. Thus we place the entire list of require()-able FDMM files into the master include "FDMM_MissionsStart.lua" (read as: any new files/modules added to FDMM will also need a require() call for them placed into "FDMM_MissionStart.lua").

As well, we also run the optionally recommended luasrcdiet afterwards - which is a performance enhancing stripper (or at least, that's the idea) and clojure-based require() function replacer.

While CoffeeScript is easiest installed via npm, node-lua-distiller and luasrcdiet are easiest to install via LuaRocks, a popular Lua package manager. However, one caveat: be sure to keep LuaRocks out of your Windows environment variables since you can mess up your DCS install if you try to globally rewire Lua to not be the one DCS natively ships with (don't quote me on that, though - but if your DCS install stops loading up and bails out at launch, never getting to the main menu, like it did me, that's probably why).

#### Note: require() re-routing

Since we don't get require() natively in DCS scripts we take full advantage of repurposing it. This has a lot of advantages, the primary one being that we can introduce our own debug/release mechanics.

In a dynamically loaded, and thus debugging-enabled (essentially what we devs use 90% of the time), environment the require() method acts much like it does in proper Lua: behind a guarded dofile() call, with check to ensure dofile() isn't called on any particular module/file 'required' more than once (copy of this code exists in "support/Dynamic Mission Start Script.lua", and is placed into our debug mode load script).

While this is nice, DCS scripts have a CWD natively as the "Saved Games/DCS/" folder, which our replacement then adds "/Missions/FDMM/workspace/FDMM/src/" to so that our require() calls treat FDMM's main src folder as the CWD. This keeps our require() methods short and tidy, but also allows us to use those require() calls in our build process.

At the same time, in a statically loaded, and thus release, environment the require() method is rewired via lua-src-diet, which does a great job of wrapping each require()'ed file in a clojure that then gets called when needed in the final mission file.

We have it set up such that the mission files in the main development branch are in debug/dynamic load mode, while it's planned to have mission files in release branches in release/static load mode.

#### Note: LDT environment build path libraries use absolute pathing

I would be so happy if LDT made it so that we could specify build path libraries using environment variables, but it would appear as if such is not supported. Please feel free to add new build path library entries to the project that correspond with your own DCS Scripts and MissionEditor folder locations, and just leave the ones that don't resolve correctly alone as a curtosey to other devs. (I would love to see someone fix this)

#### Note: 

## Debugging

In order to debug dynamically, you will need to follow the MOOSE instructions for how to get DCS debug mode to work, and also be working on the FDMM development branch files (or have debug mission user flag set).

The main gotcha is that you don't want to start the "dcsserver" debug server (which you must painfully select from the debug menu option dropdown, each time) until you have launched into the actual mission, in which, if you have it set up correctly, DCS will pause at 50% load and spend 5 seconds or so looking for a debug server connection before continuing - in which time you must start the debug server up from LDT in order for debugging to work.

(If you don't use the debug menu dropdown option for "dcsserver", despite having it as that menu option's favorite (which should make it not do this, but alas), then LDT will likely auto-create a build process for the file you currently have opened, which of course will fail, and you'll have to remove said build entry from the project files prior to check-in. All of which is just a large hassle.) (I would also love to see someone fix this.)

As you start the mission, however, be aware that before the game actually launches the main DCS instance (where this debug server connection occurs) you will be presented with a pre-launch screen that must be clicked away for the main DCS instance itself to start loading. It is only after this initial pre-launch screen is clicked away that it's best to alt-tab back to LDT and begin the "dcsserver" debug server, then alt-tab back into game to finish loading and click through the briefing screen in order for the mission to start, and thus your debug breakpoints to start getting hit as the script loads.