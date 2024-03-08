## Windows Server

### GPO

GPO繼承  連結順序(執行順序)越前面的優先權越高

產生登入失敗事件記錄:
	
	電腦設定 > 原則 > Windows設定 > 安全性設定 > 本機原則 > 稽核原則 > 稽核賬戶登入事件

失敗次數後鎖定賬戶:

	電腦設定 > 安全性設定 > 賬戶原則 > 賬戶鎖定原則 > 賬戶鎖定閥值	

固定天數後強制更改密碼:
	
	電腦設定 > 安全性設定 > 賬戶原則 > 密碼原則 > 密碼最長使用期限

設定登入訊息:

	電腦設定 > 原則 > windows設定 > 安全性設定 > 本機原則 >
	安全性選項 > 互動式登入:給登入使用者的訊息標題

### 伺服器角色
1. 域名服務: ADDS
1. Domain Services: ADCS -> 認證平臺: Cert Auth Web Enroll
1. Web Server: IIS
1. DHCP: DHCP Server
1. DNS: DNS Server
1. Routing: Remote Access -> Routing

### Import Windows 10 Policy Templates
<www.youtube.com/watch?v=bUrfqfHJosg>
- 模板位置: C:\Windows\PolicyDefinitions

將PolicyDefinitions複製到: C:\Windows\SYSVOL\sysvol\(你的域名)\Policies
或是: \\(你的域名)\SYSVOL\(你的域名)\Policies

### Password Restricts
AD管理中心 > 本機 > System > Password Settings Container > 新增密碼設定


### Set up ADCS
```sh
ADCS > Certification Authority Web Enrollment 
```

### Server Roles

- ADCS -> Certifications(SAN https...)
- ADDS -> Domain, foundational AD Service

- File and Storage Services -> DFS, NFS, iSCSI

- Network Load Balancing
	- <https://www.youtube.com/watch?v=NFtp7_U83jg>
- Network Policy and Access Services -> RADIUS 
	- <https://www.youtube.com/watch?v=GDunrEzwc3g>
	- <https://www.youtube.com/watch?v=dB8aH3Kysg0>
- Remote Access -> VPN, NAT, Routing...
- Cert:
	- <https://www.youtube.com/watch?v=lWZIHoAwu2c>
- VPN:
	- <https://www.youtube.com/watch?v=uMtJgN0prME>
	- <https://dailysysadmin.com/KB/Article/1934/create-an-sstp-vpn-server-in-windows-server-2016/>
- radius:
	- <https://www.youtube.com/watch?v=oOmIOG1eu3w>
	- <https://www.youtube.com/watch?v=QSni2IP0QJM>


### the use of powershell 
```powershell

	Start-Job {"wsl.exe nohup ~/a.sh &"} 

	Get-Help
	New-ADUser 
	Set-ADUser
	New-ADGroup
	Set-ADGroup
	Add-ADGroupMember
	Move-ADObject
	For(;;){

	}

	Get-Service
	Restart-Service

	Start-Sleep

	ConvertTo-SecureString -AsPlainText -Force "Skills39"

	$i = 1

	$username = "usr{0:d4}" -f $i
	# username: usr0001
	$username = "usr$($i.ToString('0000'))"
	# username: usr0001
```
#### powershell add domain user script 
```powershell
	for($i=0;$i -le 100;$i++){
		$username = "rad{0:d3}" -f $i
		$password = "passwd{0:d3}" -f $i

# Copied from AD Administrative Center, 
# Ctrl + H replace previous username and password with variable.
# You can also change user's properities and copy those commands to automation it.

		New-ADUser -DisplayName:"$username" -GivenName:"$username" -Name:"$username" -Path:"OU=RADIUS,DC=test,DC=org" -SamAccountName:"$username" -Server:"SRV2.test.org" -Type:"user" -UserPrincipalName:"$username@test.org"

# the command below has to use 'ConvertTo-SecureString' method to get encypted password.
# $(ConvertTo-SecureString -AsPlainText -Force $password)

		Set-ADAccountPassword -Identity:"CN=$username,OU=RADIUS,DC=test,DC=org" -NewPassword: $(ConvertTo-SecureString -AsPlainText -Force $password) -Reset:$true -Server:"SRV2.test.org"

		Enable-ADAccount -Identity:"CN=$username,OU=RADIUS,DC=test,DC=org" -Server:"SRV2.test.org"
		Add-ADPrincipalGroupMembership -Identity:"CN=$username,OU=RADIUS,DC=test,DC=org" -MemberOf:"CN=radius,OU=RADIUS,DC=test,DC=org" -Server:"SRV2.test.org"
		Set-ADAccountControl -AccountNotDelegated:$false -AllowReversiblePasswordEncryption:$false -CannotChangePassword:$false -DoesNotRequirePreAuth:$false -Identity:"CN=$username,OU=RADIUS,DC=test,DC=org" -PasswordNeverExpires:$true -Server:"SRV2.test.org" -UseDESKeyOnly:$false
		Set-ADUser -ChangePasswordAtLogon:$false -Identity:"CN=$username,OU=RADIUS,DC=test,DC=org" -Server:"SRV2.test.org" -SmartcardLogonRequired:$false
	}
```
### command prompt 
```powershell
	net group grp1 /add
	for /L %%x in (1,1,9) do (
		net user user0%%x password0%%x /add
		net group grp1 user0%%x /add
	)
```

### Connect to Debian Samba Server
<https://social.technet.microsoft.com/Forums/en-US/e63f1d76-3913-4b33-85b5-e04581d59f8b/windows-server-2019-smb-share?forum=winserverfiles>
1. Enable SMB1.0 Client
2. Change Local Group Policy 
	- `Computer Configuration > Administrative Templates > Network > Lanman Workstation > Enable insecure logons`

### NTP Server

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Config`
	AnnounceFlags = 5
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer`
	Enabled = 1

```powershell
	Restart-Service W32Time
```

### cacls
<https://ss64.com/nt/cacls.html>
```powershell
	cacls
	cacls .\test.txt /E /G user:F # grant full conteol
	cacls .\test.txt /E /R user # revoke permission
	cacls .\test.txt /T /E /D user # deny
```

### create new SID
```powershell
	System32\sysprep\sysprep.exe # check "generalize"
```
### IPv6
<https://rakhesh.com/windows/enabling-ipv6-router-advertisements-on-windows/>
```powershell
	netsh interface ipv6 set interface "ethernet0" forwarding=enabled
	netsh interface ipv6 set interface "ethernet0" advertisedefaultroute=enabled
	netsh interface ipv6 set interface "ethernet0" advertise=enabled
	
	netsh interface ipv6 set route cafe::/64 "Ethernet0" publish=yes

	netsh interface ipv6 show interface "ethernet0"

	Set-NetIPInterface Ethernet0 -AddressFamily IPv6 -AdvertiseDefaultRoute Enabled
```
#### isatap(failed)
```powershell
	# IPEnableRouter=1
	netsh interface isatap set state enabled
	netsh interface ipv6 isatap set router 1.1.1.1
```
### migrate dc 
```powershell
	netdom query fsmo
	Move-ADDirectoryServerOperationMasterRole -Identity srv1 -OperationMasterRole DomainNamingMaster, InfrastructureMaster, PDCEmulator, RIDMaster, SchemaMaster
```
### rras auto start
```powershell
	# RRAS auto start
	Stop-Service NlaSvc,IAS,RemoteAccess -Force
	Start-Service NlaSvc,IAS,RemoteAccess
```
### run startup application
```powershell
	shell:startup
	shell:common startup

	%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe -file "C:\Users\Administrator\Desktop\Untitled2.ps1"
```
### misc

```powershell
	sconfig

	pkiview.msc
	certutil -url {cert}

	certutil -cainfo xchg # for oscp respond server

	start .

	sc sidtype IAS unrestricted
```



