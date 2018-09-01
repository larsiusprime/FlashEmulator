# GlimmerEmulator
A proposal for a true "Flash Emulator"

Once upon a time, nearly all "rich internet applications" and web games were written in Flash. Today, most web browsers block Flash content by default, if they even feature a Flash plugin at all. Some Flash content (including what were once quite popular games) is already lost and gone forever. We have a race against time to preserve what's left, but we have a terrible dilemma in that we don't have a reliable Flash player that we can count on standing the test of time.

There's the existing Flash player binaries, but with Adobe sunsetting Flash in general there is no guarantee that the Flash player will continue to work reliably into the future, even on desktop targets, and it is already persona-non-grata on all major web browsers.

As of this writing, there exists no complete, "Flash Emulator" that supports arbitrary SWF playback with sufficient accuracy and performance. There do exist several incomplete and/or abandoned projects, namely Mozilla's Shumway, as well as the Lightspark project. However, there *do* exist a wide variety of open source projects that have solved nearly every problem necessary to build the sort of true "Flash Emulator" we want.

# What do we want?

There are several different ways to go about this.

1. Do we want a *native* emulator (C++ or similar) or an *HTML5* (JS or similar) emulator?
2. Do we want a *true emulator* that plays back SWF's unaltered, or do we merely want a *preprocessor* that transforms SWF's into a new format?

I argue that we want a *native* emulator, that is also a *true emulator*, for various reasons.

**Insert reasons**

# How will we build it?

All the components we need already exist:

- OpenFL for rendering & SWF parsing
  - TODO: also list some alternatives
- Adobe AVMPlus for ActionScript Bytecode execution
  - TODO: also list some alternatives
  
TODO: ~~Draw the rest of the owl~~ Write the rest of the spec.
