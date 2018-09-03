# FlashEmulator
A proposal for a true "Flash Emulator"

Once upon a time, nearly all "rich internet applications" and web games were written in Flash. Today, most web browsers block Flash content by default, if they even feature a Flash plugin at all. Some Flash content is already lost forever -- including stuff that was quite popular in its day. We must rush to preserve what's left, but we don't even have a reliable open source Flash player that we can count on standing the test of time.

There's the existing Flash player binaries, but with Adobe sunsetting Flash in general there is no guarantee that the Flash player will continue to work reliably into the future, even on desktop targets, and it is already persona-non-grata on all major web browsers.

As of this writing, there exists no complete, "Flash Emulator" that supports arbitrary SWF playback with sufficient accuracy and performance. There do exist several incomplete and/or abandoned projects, namely Mozilla's Shumway, as well as the Lightspark project. However, there *do* exist a wide variety of open source projects that have solved nearly every problem necessary to build the sort of true "Flash Emulator" we want.

# Prior attempts

Here is a list of previous attempts to create a *replacement Flash player*

1. [Mozilla shumway](https://github.com/mozilla/shumway)
2. [Lightspark](https://lightspark.github.io/)
3. [Gnu Gnash](https://en.wikipedia.org/wiki/Gnash_%28software%29)
4. [Autodesk Scaleform](http://www.autodesk.com/products/scaleform/overview)

Mozilla Shumway is a the most interesting and relevant project but [is basically considered dead](http://www.ghacks.net/2016/02/23/flash-replacement-shumway-is-as-good-as-dead/) in terms of active development. It took a "full emulator" approach -- an entirely HTML5 based tech solution, with runtime AS3 interpretation (a full javascript AS3 VM), and the ability to render arbitrary swf files. The chief limitations of Shumway are that: a javascript based interpreter is sloooow, and lack of resources killed the project before it could really achieve its goals. Shumway uses the Apache License 2.0, which as I understand it is a pretty permissive license.

Lightspark is similar to Shumway, but it's written in C/C++. Unfortunately it is still very alpha, only implements about 60% of the Flash API's, and relies on plugin architecture to work in browsers, which means it's days on the web are numbered given how anti-plugin browsers have become. The good news is that it's still getting (slowly) updated! It seems to have really struggled to attract contributions, but it's a notable and relevant project, particularly for being written in C/C++. There might be great things to learn here. Lightspark is GPL v 3, which means picking its bones for code will require our entire project to be GPL as well. I'm not categorically anti-GPL, but I like to stick with the most permissive license possible.

Gnu Gnash evolved out of the GameSWF project and is basically considered dead and never really got too far beyond pretty okay SWF rendering and responding to AS2 logic (AS3 never worked properly as I understand it). Gnu Gash was also a C++ project, not sure if it ever ran in the browser as a plugin or not. Progress has always been extremely slow and basically nobody expects the project to ever fully deliver on its vision. Gnu Gnash is licensed under the GPL.

Autodesk Scaleform is basically a partial implementation of the flash player that can run SWF content within a C++ app. It's mostly used for AAA games that want an easier way to do UI. I am not 100% up to date on ScaleForm, but from what I hear it's A) nice for designers to work with (they can just use the Flash authoring tool), B) pretty slow & clunky, C) doesn't support much in the way of Actionscript logic beyond AS2. One noteable thing -- this is what *The Banner Saga* used to launch their game on consoles. As for licensing, it might have changed since I last evaluated it, but I'm sure it ain't open source. It's entirely proprietary.

There are probably useful things to learn from some or all of these. Shumway is the safest one to pick code from given its license, but Lightspark and Gnash will instantly flip our project to GPL. Pick code from Scaleform and we'll get sued, so we won't do that.

# What do we want?

There are several different ways to go about this.

1. Do we want a *native* emulator or an *HTML5* emulator?
2. Do we want a *true emulator* that plays back SWF's unaltered, or do we merely want a *preprocessor* that transforms SWF's into a new format?

I argue that we want a *native* emulator, that is also a *true emulator*, for various reasons.

## Native

By Native we mean C/C++. The reasons are as follows:

- Performance
  - Part of the project spec is an actionscript VM, so it's better not to write a VM inside a VM. IIRC, Shumway had notable performance issues trying to write an Actionscript VM in Javascript.
- Portability/Compatibility/Future-proofing
  - Being a web-first app puts us entirely at the mercy of web browsers. For instance, Google's been messing with the HTML5 audio API's on chrome, and Apple's Safari has become the new Internet Explorer, intentionally lagging way behind in updates and standards.
- Actionscript execution
  - Our best choice for actionscript execution is an already existing VM, written primarily in C++. If we go with HTML5 we're going to have to write our own VM and endure the long slog towards bug-for-bug-compatibility
- Webassembly and Emscripten exist
  - Native first + Wasm/Emscripten->HTML5 is a better pair of final products than HTML5 first + Electron->"Native".

## True emulator

A "true emulator" is favored over a preprocessor that merely transforms SWF content into a different format. For the purposes of this article, we won't be splitting hairs between "emulation emulation" (Higan, Dolphin, etc) and "drop-in runtime replacement with full API support" (WINE, FNA). Whatever you call it, we want to create an open source flash player you can drop a (SWF + dependent file structure) into and get the same behavior you'd expect from Adobe's proprietary flash player.

Anyways, the reasons we want an "emulator" instead of a preprocessor are as follows:

- Authenticity/preservation
  - A transformation step fundamentally alters the content and it's easy for something to get "lost in translation". Also, it creates an opportunity for preservationists to mistakenly or intentionally preserve the transformed output rather than the original source material.
- Less error prone
  - Transpiling adds all sorts of opportunities for subtle and difficult to fix bugs, not only because of mismatches between what Actionscript can do and whatever the target format can do, but also because 
- Don't complicate things
  - A transformation step kicks the responsibility of faithful rendering and execution to the target platform, and introduces external dependencies and complexities to the project. It's more complicated to maintain and harder for end users to use.

# How will we build it?

Here's what we need, at minimum:

1. A SWF file format parser
2. A renderer for SWF content
3. An actionscript virtual machine

And here's what we need to make it *good*:

4. Glue to tie it all nicely together
5. A system to manage disparate versions of actionscript and SWF format, simulate a server & filesystem, workaround sitelocks, etc.
6. A robust set of accuracy tests.

Fortunately for us, all of the maor components we need already exist in complete or nearly complete form.

## OpenFL for SWF format / SWF rendering

First and foremost, SWF is a well-understood file format and there are many existing third party solutions, both open source and commercial, for reading SWF data.

I am almost certainly biased here, but I recommend we use Haxe/Lime/OpenFL for SWF ingestion, parsing, and rendering.

Context for the uninitiated:
[OpenFL](https://www.openfl.org) is chiefly used as a compile-time library for authoring new content that can run on a variety of platforms, including flash. It uses the [Haxe](https://www.haxe.org) programming language, which gives it the ability to export to basically any language and platform.

So, you can make a C++ app for your mac/win/linux desktop (ie, a Steam game) with OpenFL. You can also make an iPhone or Android app. You can also make an HTML5 game. And you can export a regular ol' flash game too as a SWF file (which you can, in turn, wrap in Adobe AIR). All from one single code base. Obviously, there's a lot of various little caveats and in practice it's not 100% turnkey, but for the most part it just works, and works well.

You can also use SWF content in OpenFL apps, for all targets, not just Flash target. IE, you can use your SWF animations and assets in your C++ desktop target, or your C++ mobile app, or your HTML5 web app (as either WebGL or Canvas).

It should be noted that OpenFL, as currently used, is not attempting to directly perform the role of an "Open Source Flash Player." It does, however, contain many key technologies that could help build such a thing.

The basis for the "player" itself would thus be a Haxe/Lime/OpenFL application. A simple OpenFL app already has the ability to process and display SWF content, but crucially it lacks the ability to execute the SWF's ActionScript Byte Code (ABC hereafter).

## AVMPlus for ABC execution

Adobe has open sourced the Action Script Virtual machine that they used in the Flash player and Adobe AIR, it can be found here:

[AVMPlus](https://github.com/adobe/avmplus) -- 2016, this version looks to be the most recent
[AVMPlus](https://github.com/adobe-flash/avmplus) -- 2013, this looks a bit older
[Tamarin](https://developer.mozilla.org/en-US/docs/Archive/Mozilla/Tamarin/Tamarin_Build_Documentation) -- the AVM used to be called Tamarin, and apparently was donated to Mozilla at some point? 

AVMPlus is written (mostly) in C++ and is our best bet for ensuring bug-for-bug compatibility with the Flash Player in terms of actionscript execution. It's a big, daunting project, but there's no reason it can't in principle be coupled to another C++ app (such as our OpenFL app, compiled for C++ target).

## Putting it together

1. Build a native (C++) OpenFL app as the overall "container"
2. Provide a simple frontend for loading SWF content at runtime
3. Parse the SWF content, extract all library symbol references, frame scripts, ABC, etc.
4. Invoke the AVM, bootstrap the ABC program insertion point, and begin execution

Obviously, plenty of research will be required to work out the details -- when and how the internal AVM will need to call out to the container app to move objects around, all the fiddly details surrounding preloading and load timings, audio encodings, fonts, etc.

But all the pieces exist.

## Alternatives

- [] Blah
- [] Blah
- [] Blah
  
TODO: ~~Draw the rest of the owl~~ Write the rest of the spec.
