---
layout: post
title: SILENTTRINITY quickstart
---

This post is to get you started with SILENTTRINITY, by byt3bl33d3r

ST is the latest and greatest post-exploitation tool for windows devices that takes advantage of AMSI not being able to see into .NET applications by using one to create an unmamaged runspace where an IronPython script is called to do all our nefarious deeds. 

We'll clone the repo and install the requirements into a python3.7 virtual environment and get started.

clone the repo:

`git clone https://github.com/byt3bl33d3r/SILENTTRINITY.git`

create a virtual environment -- it's generally safer to install python applications in their own environments to keep competing dependency issues at bay, it's definitely not madatory though.

`python3.7 -m venv STvenv`

`source bin/activate`

to hop into the virtual python environment

`(STvenv) Server ❯ python3.7 -m pip install -r requirements.txt`

Now we can start up the app:

![](https://braaaax.github.io/braaaax.github.io/images/ST-startup.png)

Start-up a listener:
![](https://braaaax.github.io/braaaax.github.io/images/ST-startlistener.png)

I have a few stagers to choose from:
![](https://braaaax.github.io/braaaax.github.io/images/ST-liststagers.png)

Create an msbuild.exe stager:
![](https://braaaax.github.io/braaaax.github.io/images/ST-generatestagers.png)

Spin up an SMB server, we'll execute from our windows box via UNC path
![](https://braaaax.github.io/braaaax.github.io/images/ST-startSMBserver.png)

execute:
![](https://braaaax.github.io/braaaax.github.io/images/ST-windowscmd.png)


if there's any issue I can check and see that the share has indeed been reached.
![](https://braaaax.github.io/braaaax.github.io/images/ST-accessSMBserver.png)

Now I have an ST session established.
![](https://braaaax.github.io/braaaax.github.io/images/ST-startsession.png)

A great part of ST is the access to its modules. If you are familiar with python and csharp you can whip up your own modules pretty quickly.

![](https://braaaax.github.io/braaaax.github.io/images/ST-listmodules.png)

![](https://braaaax.github.io/braaaax.github.io/images/ST-runmodule.png)

There you have it, the basics of SILENTTRINITY. As of thsi writing Defender is unable to find and/or kill these payloads. Maybe things will change when .NET 4.8 is released when AMSI will have some insight into external .NET applications.

