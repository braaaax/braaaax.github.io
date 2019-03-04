---
layout: post
title: HTB Access -- medium
---

Quick overview:
Access is a medium windows box build by egre55. It presents with open tcp ports on 21 (ftp), 23 (telnet), 80 (http). There are credentials in a password protected mdb file. The credentials from the Access db file gives us access to a telnet server. Enumerating the server reveals a stored credential to the System acct, which can be leveraged to get a shell as System.

Using nmap to find the open ports and associated services. 
![](https://braaaax.github.io/braaaax.github.io/images/ACCESS-nmapscan.png)

Anonymous access to ftp:
```
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
```

FTP, Telnet, and HTTP are open and FTP allows anonymous access. There is a Microsoft Access Database file and a password protected zip file, both of which are downloaded to the attack box for analysis.

![](https://braaaax.github.io/braaaax.github.io/images/ACCESS-ftpbackup.png)

Running strings against the .mdb file. Depending on the version there is a good chance that the password will be in plaintext or simply XOR'd. Strings does, in fact, reveal the password and a few potential users.

`strings backup.mdb`

```
backup_admin
admin
engineer
access4u@security
```

Trying some of the more interesting found strings in the .mdb file gives us access to the protected zip file. Which contains a .pst file. The .pst file is an Outlook mailbox file which will contain email messages in plaintext. There tool, readpst is in the kali repo and will yeild and .mbox file to further examine.

`readpst Access\ Control.pst`

`cat Access\ Control.mbox`

![](https://braaaax.github.io/braaaax.github.io/images/ACCESS-readmbox.png)

So now we have a user `Security` and their password `4Cc3ssC0ntr0ller`

![](https://braaaax.github.io/braaaax.github.io/images/ACCESS-logintelnet.png)

Telnet shells are horrible you cant do backspace, they will die or freeze at random seemingly and definitely if left idle for any amount of time. 
mshta, msbuild, rundll32, wmic are blocked by group policy, same with a few go and c binaries on a share executed from UNC path, but powershell works... er mostly. A few of the nishang shells httpRAT, tcponeline for example are full of errors, though they will execute and connect back.

Running JAWS or cmdkey /list will reveal a saved cred.

![](https://braaaax.github.io/braaaax.github.io/images/ACCESS-cmdkey.png)

After a bit more enumeration, there is an lnk file in the \Users\Public\Desktop dir which has some plaintext exposed showing the beginning of a runas commamd. Assuming the link exists for a low priv user to run the zteko application we can use runas to reveal the root flag.

![](https://braaaax.github.io/braaaax.github.io/images/ACCESS-zkteco.png)

```
runas /savecred /user:Administrator "cmd.exe /c whoami > C:\test.txt"
runas /savecred /user:Administrator "cmd.exe /c type \users\Administrator\Desktop\root.txt > C:\test.txt"
```