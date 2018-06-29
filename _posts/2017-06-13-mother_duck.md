---
layout: post
title: "Mother Duck!"
date: 2017-08-14
categories: robotics sphero turtlebot duck ai
thumbnail: /images/motherduck/thumbnail.png
---

Goal
-------

The goal here was to get a swarm of Spheros to follow a Turtlebot as ducklings follow their mother. Why? For fun. Also, it was a proof of concept for BYU's HCMI lab. They had purchased a couple Turtlebots, and a whole slew of Spheros, and this was a good way to get a framework up, so things would be ready for more complicated, and interesting projects. This project required a few steps; sphero localization done via color blobs, creating a [ROS](http://www.ros.org/) package that could connect to and control many spheros simultaneously, and integrating the sphero system with the Turtlebot.

Feel free to skip to the [results](#result).


Process
--------
Localization was done via [color blob detection]({{ ste.baseurl }}{% link _posts/2017-06-13-color_blob.md %})

The library I used to control the Sphero in the past wasn't suitable for this task. It couldn't reliably run more then three spheros. This is because it didn't provide guarantees of message delivery, or responses from the sphero. It also ran all Spheros in the same python interpreter instance.  Which, due to python's lack of parallel processing, meant it couldn't keep up with the message load required to keep the connections alive.

To fix this I did a basic rewrite of the underlying Sphero SDK, [SpheroPy](https://github.com/darin-costello/spheropy). This package is independent of ROS, provides some level of guaranteed delivery and receipt of messages, and simplified the Sphero's interface. Then I wrapped this new library into a [ROS package](https://github.com/darin-costello/sphero). This was fairly simple, as I reused much of the code from the original one. Finally, I created  new [sphero_swarm](https://github.com/darin-costello/sphero_swarm) package to control a swarm of spheros in ROS. This package uses python's subprocess library to start a new sphero node for each swarm member running it in its own interpreter, allowing the system to be entirely parallel. The package also provided a central location to query robot statess and issue commands.

Testing
========

To test if the swarm work, I programed up a couple of simple behaviors. The first has the spheros spin around a central point.

<iframe width="560" height="315" src="https://www.youtube.com/embed/usyY9VhZJ_4" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Then I programmed a 2-dimensional version of the algorithm found in [Collective Memory and Spatial Sorting in Animal Groups](https://ac.els-cdn.com/S0022519302930651/1-s2.0-S0022519302930651-main.pdf?_tid=55c2f8cc-9ff3-4a25-b09c-68593eb01f62&acdnat=1528333176_923f4c61d58caf18bda87c2ede7ded79). In a nut shell, the spheros have limited vision, they try to group, but not get to close, and they try to move in the same direction.

 <iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?list=PL4FfHurFqHLX-pbkjNmFFtdl0--g7Z711" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


TurtleBot Integration
=======

Finally the whole thing needed to attached to the turtle bot. I took the PVC that held the camera to the ceiling, and flipped it upside down. To attach it to the turtlebot I initially designed a rig to 3d print, then my wife built it out of wood in about a minute. With ROS, software integration was a breeze. The biggest issue was keeping track of the rotation of the Turtlebot so the spheros stayed oriented. This information is made available by the Turtlebot, and linear algebra took care of the rest.

Result {#result}
-------

The final system attaches onto a turtle bot, and allows for swarm of 7 spheros (the max number of bluetooth connects per dongle). It's isolated so the turtlebot is free to run whatever program it wants. In the demo it is being manually driven. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/EZFwO68UK-s" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


A playlist of more videos can be found [here](https://www.youtube.com/watch?v=-hhLRCKvZAk&list=PL4FfHurFqHLXLKwpvOWm_gIwAgnY_hkay)