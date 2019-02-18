---
layout: post
title: SILENTTRINITY quickstart
---

This post is to get you started with SILENTTRINITY, by byt3bl33d3r

ST is the latest and greatest post-exploitation tool for windows devices that takes advantage of AMSI not being able to "see" into .NET applications by using one to create an unmamaged runspace where an IronPython script is called to do all our nefarious deeds. 

We'll clone the repo and install the requirements into a python3.7 virtual environment and get started.

clone the repo:

`git clone https://github.com/byt3bl33d3r/SILENTTRINITY.git`

Create a virtual environment with `python3.7 -m venv STvenv` -- it's generally safer to install python applications in their own environments to keep competing dependency issues at bay, it's definitely not madatory though.

Use `source bin/activate` to hop into the virtual python environment just created.

And install the dependencies with the command:
`(STvenv) Server ❯ python3.7 -m pip install -r requirements.txt`

Now we can start up the app and have some fun.
`python3.7 st.py`

We're greeted with the ST welcome screen. The basic prinicpal behind the tool, and with most C2s, is to create a listener, then generate a stager or payload to launch on the target machine which will call back to your listener. Then use a module in the session that's been created, which will run asynchronously. 

Welcome Screen:
![](https://braaaax.github.io/braaaax.github.io/images/ST-startup.png)

Start-up a listener:
![](https://braaaax.github.io/braaaax.github.io/images/ST-startlistener.png)

Choose a stager:
![](https://braaaax.github.io/braaaax.github.io/images/ST-liststagers.png)

Create an msbuild.exe stager:
![](https://braaaax.github.io/braaaax.github.io/images/ST-generatestagers.png)


I like to run my malware from shares I host so I don't have to put anything on the target machines disk. Impacket has a great tool to exactly this. 

Spin up an SMB server:
![](https://braaaax.github.io/braaaax.github.io/images/ST-startSMBserver.png)

Execute from our windows box via UNC path:
![](https://braaaax.github.io/braaaax.github.io/images/ST-windowscmd.png)

Verify connection from target machine:
![](https://braaaax.github.io/braaaax.github.io/images/ST-accessSMBserver.png)

Now I have an ST session established.
![](https://braaaax.github.io/braaaax.github.io/images/ST-startsession.png)

A great part of ST is the access to its ecosystem of modules. If you are familiar with python and csharp you can whip up your own modules pretty quickly.
![](https://braaaax.github.io/braaaax.github.io/images/ST-listmodules.png)

![](https://braaaax.github.io/braaaax.github.io/images/ST-runmodule.png)

There you have it, the basics of SILENTTRINITY. As of this writing Defender is unable to find and/or kill these payloads. Maybe things will change when .NET 4.8 is released when AMSI will have some insight into external .NET applications.

Also of note: ST accepts resource files. So if you wanted to immediately start up a listener you can do so by listing the necessary commands in a txt file, one command per line. 

```bash
echo $'listeners\nuse http\nstart\n' > ST.rc
python3.7 st.py -r ~/ST.rc
```

