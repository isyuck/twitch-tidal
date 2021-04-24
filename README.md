# tidal-wave

tidal-wave is an experiment in live coding collaborative music and game design.

It provides a interface to TidalCycles from twitch.tv chat, by forwarding patterns to a ghci (command-line Haskell) instance.

Patterns are passed 1:1 between twitch and Tidal, so whatever works in Tidal will (in theory) work from a twitch message.

Twitch Message | Tidal Equivalent
--- | ---
`!t s "bd cp sn hh"` | `d1 $ s "bd cp sn hh"`
`!t s "psr*4" # speed "<1 2 3 4>"` | `d1 $ s "psr*4" # speed "<1 2 3 4>"`

In practice, you can think of `!t x` as shorthand for Tidal's `d1 $ x`.

Using Tidal 1.7.2, most if not all functions and effects will work, but transitions will not.  If you're using a version of Tidal newer than 1.7.2, newer functions will not work. You can still use some transitions via twitch commands, and this is explained more in-depth in the Game Design section.

Patterns are kept on a time based stack and are associated with users. There is a (configurable, maybe one day dynamic?) maximum number of patterns that can be active at any one time. When a user sends a new pattern to the chat, their currently active (old) pattern will be replaced by their new pattern. If they don't have any active patterns, the *oldest* pattern on the stack will be replaced, regardless of who it belonged to. As new patterns replace old patterns, new groups of patterns will form over time.

Tidal-Wave doesn't intend to be a twitch based [estuary](https://github.com/dktr0/estuary), [troop](https://github.com/Qirky/Troop), or [flok](https://github.com/munshkr/flok). All of these are better alternatives for collaborative live coding. Streaming on twitch incurs a 10-30 second delay between sending a pattern and hearing it's result. With enough active users, by the time you hear your pattern it may already have been replaced by a new one from someone else. Tidal-Wave aims to explore emergent music in a group context, created by users who will have to anticipate where the music will be by the time they get to hear their pattern.  Tidal-Wave also intends to create a game-like experience using tidal as the mechanic.

---

***Note: this is new software, and you'll probably find a way to break it! If/when you do, please let me know by opening an issue. 💖***

### Requirements

Tidal-Wave should work on all major OS's. You will need, in no particular order:

- A twitch channel to stream to
- A *seperate* account for Tidal-Wave to use as a bot
- People who want to make music
- [node.js](https://nodejs.org/en/)
- [TidalCycles](https://tidalcycles.org/Welcome)
- [SuperCollider](https://supercollider.github.io/)

A full guide for installing Tidal and SuperCollider for various platforms can be found [here](https://tidalcycles.org/Installation).

After these are installed and working, you will need to modify your Tidal boot file, `BootTidal.hs`, to
tell Tidal to listen for OSC messages. you can find this in the `tidal` directory wherever cabal is installed , e.g.
for Tidal v1.7 on Linux: `~/.cabal/share/x86_64-linux-ghc-8.4.4/tidal-1.7/BootTidal.hs`

change the line:

`tidal <- startTidal (superdirtTarget {oLatency = 0.1, oAddress = "127.0.0.1", oPort = 57120}) (defaultConfig {cVerbose = True, cFrameTimespan = 1/20})`

to include:

`cCtrlAddr = "0.0.0.0"` and `cCtrlPort = 6060`

so it looks like this:

`tidal <- startTidal (superdirtTarget {oLatency = 0.1, oAddress = "127.0.0.1", oPort = 57120}) (defaultConfig {cVerbose = True, cFrameTimespan = 1/20, cCtrlAddr = "0.0.0.0", cCtrlPort = 6060})`

`cCtrlPort` is the port that tidal listens for OSC messages on, so if you're using this port for something else,
or if you want it to be another number you can modify it. (if you do this, keep in mind that you'll have to also change the port from
6060 in `config.js`.

---

### Install

Once you've met the requirements above, you can download Tidal-Wave by opening a terminal and running:

`git clone https://github.com/isyuck/Tidal-Wave.git`

Again from a terminal, enter the Tidal-Wave directory by running:

`cd Tidal-Wave`

After this you have to install the packages Tidal-Wave uses. Do this by running:

`npm install`

---

### Config

Next, you need to configure Tidal-Wave to your channel. Replace `username` and `password` in `config.js` to
the username and password/OAUTH of your bot account. Then change `channels` to the channel name of the *different* account
hosting the stream. You can also change `maxActivePatterns` if you want to have more/less patterns active at once.

---


### Running Tidal-Wave

*Tidal related tutorials can be found on the Tidal wiki.*

Finally, you can start streaming! To get everything started:

- Run Tidal-Wave from a terminal: `node Tidal-Wave.js`.
- Start SuperCollider and SuperDirt.
- Test it by sending a `!t s "bd cp"` in your twitch chat (you don't have to be live to do this)

If everything is working correctly, you should hear a clap and a kick drum.

### Game Design

By default, each pattern is added with the transition `jumpIn'`. This adds a pattern always at the beginning of the next cycle.

You can make patterns expire by typing in the chat !expire `x`  where x is the number of cycles the pattern lasts. This uses the `mortal` transition from Tidal.  You can find a `expiration` parameter in `config.js`
Typing in !expire 0 will revert this functionality to the original `jumpIn'` transition.

To join a group, type !group `groupname`. Your group name may not contain spaces, not can you be in multiple groups at the same time.  But of course, you can edit any of these parameters in script to your liking :)
