##  Managed Instance Group

####    Quick Demo

-   Create a Windows Machine and install everything

-   Execute [powershell script](https://github.com/ansible/ansible/blob/stable-2.8/examples/scripts/ConfigureRemotingForAnsible.ps1) on Windows machine that is to be managed by ansible. This scripts enables WinRM on managed Windows nodes.

-   Deploy everything required for our applications (ASP.Net application...etc)

-   Run "GCEsysprep"

-   Delete the instance and keep Disk

-   Create a Image from previously kept Disk

-   Create a Instance Template with below metadata

key|value
---------------|:--:|
windows-startup-script-url|<URL>/shutdown.cmd|

-   Startup Script

    -   setup.cmd
```bash
c:\windows\system32\netdom.exe JOIN %computername% /domain:demo.local /userd:demo.local\kalschi /passwordd:<PASSWORD> /reboot:60  >> c:\temp\setup.log

powershell C:\temp\setup.ps1 >> c:\temp\setup.log
```

    -   Setup.ps1

```powershell
Import-Module WebAdministration
Set-ItemProperty IIS:\AppPools\ad-joined-app-pool -name processModel -value @{userName="demo.local\service-account-demo";password="<PASSWORD>";identitytype=3}

iisreset
```

-   Shutdown script
    -   cleanup.cmd
```shell
netdom remove %computername% /domain:domainname /userD:demo.local\kalschi /passwordD:<PASSWORD>
```

-   Create an Instance Group with length of name less than 10 bytes

Name of the instance group become hostname prefix of our provisioned instances. Windows allows only 15 cahracters as computer name alias, and MIG automatically append a random string of 4 characters with "-" to MIG name. So we have 10 characters as our MIG name.