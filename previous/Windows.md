## Windows

### GPO
```powershell
    gpupdate /force # 更新
    RSoP.msc # 檢查
    gpresult /H /Report.html /f
    gpresult /R
```

### Time Sync
<https://blog.miniasp.com/post/2009/06/08/How-to-adjust-Time-using-Windows-Time-Service>
```powershell
    w32tm /resync
    w32tm /query /status
    w32tm /query /peers
```

### SSTP disable revocation check 

`HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\SstpSvc\Parameters`
create `DWORD` `NoCertRevocationCheck` in registry and set to `1`

### IPv6
```powershell
    # enable eui-64
    Get-NetIPv6Protocol
    Set-NetIPv6Protocol -RandomizeIdentifiers Disabled # disable randomize and use eui-64
    getmac # check 

    ipconfig /release6
    ipconfig /renew6

    # disable temporary address
    Set-NetIPv6Protocol -UseTemporaryAddresses disabled

# https://ipv6-literal.com/
    # \\cafe--1.ipv6-literal.net when connecting to cafe::1 on Windows Explorer
    # http://[cafe::1] when connecting to cafe::1 on Internet Explorer
```

### 7zip
<http://dotnetperls.com/7-zip-examples>
```powershell
    7z a -t7z files.7z *.txt # -tzip -x garbage
    7z e archive.zip
```

#### misc
```powershell
    net use \\server\share "" /user:""
    slmgr.vbs -rearm
```

#### Web Browser Shortcuts
```powershell
    microsoft-edge:https://www.google.com
    iexplore "https://www.pornhub.com"
    chrome "https://www.pornhub.com"
```
#### windows sandbox 
.wsb file
```json
    <Configuration>
        <MappedFolders>
            <MappedFolder>
            <HostFolder>C:\temp</HostFolder>
            <ReadOnly>true</ReadOnly>
            </MappedFolder>
        </MappedFolders>
    </Configuration>
```

