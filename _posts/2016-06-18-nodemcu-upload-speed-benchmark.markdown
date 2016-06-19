---
published: true
title: NodeMCU Upload Speed benchmark
layout: post
tags: [nodemcu, esp8266, lua, benchmark]
categories: [IoT]
---
Here's a short comparison I did of 3 command-line tools for uploading Lua and other files to your esp8266 via USB. In this test I'm only looking at tools to transfer files to a Lua-based nodemcu board. (not flashing the actual firmware like you do with the Arduino IDE)


| Tool             | seconds | kb/s          |
| ---------------- | ----:| -------------:|
| [Luatool.py](https://github.com/4refr0nt/luatool/tree/master/luatool) | 35.9 |    0.08 |
| [nodemcu-tool](https://github.com/andidittrich/NodeMCU-Tool)          |  9.2 |     0.3 |
| **[nodemcu-uploader](https://github.com/kmpm/nodemcu-uploader)**          |  **1.3** |     **2.2** |


(Tested by uploading a 2.8kb file)

Apparently, there's a huge difference in the speed that these tools achieve when uploading files. The clear winner is [nodemcu-uploader](https://github.com/kmpm/nodemcu-uploader), which is 7x faster than [nodemcu-tool](https://github.com/andidittrich/NodeMCU-Tool) and a whopping 27x faster than [luatool.py](https://github.com/4refr0nt/luatool/tree/master/luatool)

Combining the upload speed of nodemcu-uploader with [my build script](https://remcoder.github.io/iot/2016/05/17/incremental-lua-uploads-for-nodemcu.html) that only uploads changed files, your dev cycle is a guaranteed to be fast ;-)