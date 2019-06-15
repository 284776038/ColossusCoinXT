I2PD Compiling And Known Issues
=====================================

[![Build Status](https://travis-ci.org/COLX-Project/COLX.svg?branch=i2pd)](https://travis-ci.org/COLX-Project/COLX) [![GitHub version](https://badge.fury.io/gh/COLX-Project%2FCOLX.svg)](https://badge.fury.io/gh/COLX-Project%2FCOLX)

# Latest Status Summary
- **Gitian building/releases are now working** (fully I hope).
- 'server' nodes are still running w/o any manual intervention (I didn't check the logs if any issues but it seems stable). 
- no known crashes (doesn't include shutdown, see below) - though I am expecting occasional crashes till we filter it all out.
- Performance is the biggest issue that I'm seeing (mostly during the syncing) + some reconnecting issues (after hours and under specific circumstances), see below for more. 
- testnet is running on some old and maybe even forked data so not very good for testing and it's just to showcase connecting the nodes, exchanging messages. We need full old testnet moved over I guess and set up properly. 
- there is a new setup section - `Small Local Testnet Setup`.

# Gitian Compiling
- tag: **v0.1.1-i2pd-alpha** 
- tested with 'amd64 trusty' target (on a Debian host). 
- both Linux and Win descriptors and hosts are tested and should be fully compiling (and running as I could see).
- arm linux is failing with i2pd custom script somewhere (I probably won't be able look into that any time soon).

# Known Issues 

## Compiling Issues

**mostly resolved now** - and with Gitian compiling working.  

- **i2pd branch is a bit behind the master, it's based on the pre-NY master**, all the changes afterwards (including the major GUI/qt changes) are not yet merged in.  
- make is (still) not picking up changes in the src/i2pd folder. For now just remove *.a, *.o from it (i2pd subfolder) and recompile.  
- in certain cases after a while build will fail with libminzip related errors. Something due to make clean or make install. Fix is to do a clean clone and build.  

### Windows (mingw) Compiling  (non-Gitian)

- If manually building, I've **removed the `--with-libs` (e.g. from `CONFIG_SITE` build_all.sh, gitian is not affected)**, because the `libbitcoinconsensus` doesn't compile for Windows (as it's not really needed at this point).
- Note: if it fails to even start, with an `error: provided command 'x86_64-w64-mingw32-g++' not found`, you need the following: `sudo apt-get install g++-mingw-w64-i686 mingw-w64-i686-dev g++-mingw-w64-x86-64 mingw-w64-x86-64-dev`

## Setup Issues (different environments, clean setup, libs etc.)
- I've had an issue on an empty Ubuntu 16.04 box (remote vm, nothing on it, no desktop).  
` 	./colxd: /usr/lib/x86_64-linux-gnu/libstdc++.so.6: version 'GLIBCXX_3.4.22' not found (required by ./colxd)`  
I'm not sure if this is new (seems it's due to recent changes/i2pd) or why is it showing up now.  
Fix was (either that or static linking and increase the size):  
```
	sudo add-apt-repository ppa:ubuntu-toolchain-r/test
	sudo apt-get update
	sudo apt-get install gcc-4.9
	sudo apt-get upgrade libstdc++6
``` 



## Stability, Performance, Functionality
COLX should run relatively stable now, I've fixed couple major problems and things are running ok for a while.  
- Main issue left IMO is the performance (during the sync mostly), i.e. qt/GUI app is running somewhat slower, not responding well to mouse/input (you need to wait 10 secs or so to do anything). As I remember it wasn't like that before. It'll probably need adjusting thread priorities (or equivalent timing/locks etc.) as i2pd threads seem to have disrupted the previous status quo (or even halt i2p where required). Note: I've tried a few fast things but didn't help much so far, this will need some more optimizing.
- there's a recent issue when things stop syncing after many hours - restarting the app fixes it. Problems seems to be around renewing connection in some cases after the server closes the connection w/ the client, client doesn't pick up and continues waiting on it.
- it crashes on close, some i2pd Log issue (and we're not stopping the i2p threads on exit, something to fix).

# Needs Testing
- I've done little to no testing on the actual tx-s processing after the blocks are synced, as syncing took most of my attention so far. I'm guessing things should work as all messages are circulating in between just fine but there could be specific issues.
- Also MN-s need testing for any issues (not tested at all). 
- Test for (re)connectiing problems (as that's where the major changes were made, networking layer).   

# What needs to be done (todos, plan, testing...) 
- we need a proper testnet, i.e. having the stable blockchain, some of the main MN-s we've had before I guess. You can even bring a few nodes yourself and test things just between them but proper testnet is needed so we could really test things out.
