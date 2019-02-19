---
layout: post
title: Covenant quickstart
---

Covenant is a .NET C2 that, like SILENTTRINITY, capitalizes on Windows Defender's blindspot for .NET applications.
it won't be until the release of .NET 4.8 that AMSI will have insight into .NET. 
Elite is the client to Covenant's Server. Below is a walkthrough on how to set everything up with docker and run a module.

Install docker (for kali docker will be in the repo so `apt install docker` will do):
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo 'deb https://download.docker.com/linux/debian stretch stable' > /etc/apt/sources.list.d/docker.list
apt-get update
apt-get install docker-ce
```

Build and start Covenant container (make sure you're in the same directory as the Dockerfile):
```
docker build -t covenant .                                                   
```

Start the container and expose ports 80, 7443, and 443. (make sure you don't have anything else listening on those ports or the docker daemon will error out):
```
docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant covenant --username brax --computername 0.0.0.0
```
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startup.png)


Build and start elite container and connect with Covenant container server.
Again, make sure the directory you run the command contains the Dockerfile.
```
docker build -t elite .
```

Create the Elite container client and connect with the Covenant server and remove when closed.
```
docker run -it --rm --name elite -v /root/cov/brax:/app/Data elite --username brax --computername 192.168.0.128
```
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startelite.png)


Start listener:
```
listeners
http
set ConnectAddress 192.168.0.128
start
```
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startLISTENER0.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startLISTENER.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-GRUNTS.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-interact.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-modules.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-indicators.png)