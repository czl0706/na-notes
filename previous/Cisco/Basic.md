
[TOC]

---

### 學習資料

- 教學影片及簡報

[加肥貓的教學網站](https://www.jychen.idv.tw/Skills39/index.htm "Taiwan Skills Competition #39")

>帳號: skills39
>密碼: HUSTint

- EVE-NG

<https://www.jannet.hk/zh-Hant/post/eve-ng/>

---
### 基本操作

#### 通用

```sh
    ?
    e?
    enable ?
```

<kbd>Ctrl</kbd> + <kbd>E</kbd> cursor top <br/>
<kbd>Ctrl</kbd> + <kbd>A</kbd> cursor bottom <br/>
<kbd>Ctrl</kbd> + <kbd>X</kbd> delete text before cursor <br/>
<kbd>Ctrl</kbd> + <kbd>N</kbd> delete text <br/>
<kbd>Ctrl</kbd> + <kbd>P</kbd> previous command <br/>
<kbd>Ctrl</kbd> + <kbd>Z</kbd> <br/> 
<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>6</kbd> <br/>


#### 使用者執行模式

`Router#disable`
`Router> `

#### 特權執行模式

`Router>enable`
`Router# `

```sh
    show interface (interface)
    show ip interface brief
    show ipv6 interface brief
    show ip route

    show version
    show running-config
    copy running-config startup-config # important!!
    write # 功能同上
    show clock
    clock set 10:48:01 Feb 14 2020
    reset

    show flash:
    delete flash:
    
    erase startup-config
```

#### 設定模式

- 全域

    `Router#configure terminal`
    `Router(config)#exit/end`
    ```sh
        hostname test1234
        enable secret passwd
        # enable password passwd
        
        do {cmd} # 在conf模式下使用exec指令

        service password-encryption # 加密
        ip host _name_ _IP1_ _IP2_ _IP3_ ... # telnet _name_
        banner (motd) 結束符號 訊息 結束符號 # message of today

        # 符號自定
        enable algorithm-type {md5(type. 5) | scrypt(type. 9) | sha256(type. 8)} secret _passwd_

        banner motd {} # when enter vty 
        banner login {} # when login (before password)
        banner exec {} # when login (after password)
    ```

- 界面(低速用line)

    - Console
        ```sh
            # !! console 0 即為Packet Tracer內CLI連接介面 !!
            line console 0  -> Router(config-line)#
            login # !! enable password checking
            password Core2Quad
            logging synchronous # 記錄(輸入)同步 避免輸入被訊息中斷
            exec-timeout {min} {sec}
            exit
        ```

    - vty
        ```sh
            line vty 0 4 # (0 ~ 4)
            login
            password Core2Duo

            no login
            login local
            privilege 15

            history size {size}
        ```

- 路由器網路界面
    ```sh
        interface 界面形態 界面編號
        ip address IP位置 遮罩 (secondary)
        no shutdown
        exit

        clock rate clock-rate
        # serial中 , DTE指Data Terminal Equipment; DCE指Data Circuit-terminating Equipment
        # DTE = 使用者端 ; DCE = 服務業者端
        
        show controller int s0/0/0
        description __

        speed {10/100/1000}
    ```

- IPV6

    注: 同一個介面下可以有多個ipv6位置

    ```sh
        # 加入FF02::2的Multicast位址
        ipv6 unicast-routing # initialize !! 開啟才能使路由器轉發IPv6資訊 !!

        # 設定指令: 把原本ip的地方改成ipv6
        ipv6 address ipv6-addr/prefix-lens (eui-64)

        no ipv6 address ipv6-addr/prefix-lens # 取消
        ipv6 address {link-local-address(pref. fe80::)} link-local

        ipv6 enable # link-local + eui-64
        ipv6 address autoconfig # slaac + eui-64
 
        show ipv6 neighbors
        show ipv6 interface (if-name)
    ```

### Misc

#### configure auto secure
```sh
    auto secure
```

#### 密碼復原程序

- 路由器

    <kbd>Ctrl</kbd>+<kbd>pause break</kbd> 開機時阻止映像檔載入

    `rommon 1 >`

    ```sh
        confreg 0x2142 # 開機跳過startup-config
        reset

        copy startup-config running-config # 還原
        configure terminal # 特權模式下進行
        # 更改密碼... 
        config-register 0x2102 # recover 
        copy running-config startup-config # 覆蓋
    ```

- 交換器

    ```sh
        # 先進入恢復模式
        
        flash_init
        load_helper
        rename flash:config.text flash:config.text.old
        boot

        enable 
        copy flash:config.text.old system:running-config

        # 更改密碼...

        write
    ```

#### CDP(Cisco Discovery Protocol)
```sh
    cdp run # 全域啟動

    int g0/1
        no cdp enable # 正常方法為設定全域啟用 再把不要開的介面關掉 !! 不能全域關閉再開啟特定介面 !!

    show cdp neighbors (detail)

    cdp timer {time}
    cdp holdtime {time}
    show cdp 
```

#### LLDP(Link Layer Discovery Protocol)
```sh
    lldp run # 全域啟用

    int g0/1
        no lldp (transmit/receive) # 設定介面收送

    show lldp neighbors (detail)

    lldp timer {time}
    lldp holdtime {time}

    show lldp
```

#### NTP
<https://www.jannet.hk/zh-Hant/post/network-time-protocol-ntp/>
```sh
    clock timezone GMT +8

    ntp master (stratum:層數 (1-15)) # 設定為server
    ntp server {server-ip} # 設定為client
    ntp peer {ip}

    ntp source {if}

# md5 encryption

    # server
    ntp authenticate
    ntp authentication-key {id1} md5 {pw}
    ntp trusted-key {id1}

    ntp master 1

    # client
    ntp authenticate
    ntp authentication-key {id2} md5 {pw}
    ntp trusted-key {id2}

    ntp server {ip} key {id2}

# use broadcast
    int g0/1
        ntp broadcast
        ntp client broadcast

    show clock (detail)
    show ntp associations 
    show ntp status
```

#### 用TFTP下載bin
```sh
    copy tftp: flash:
        172.16.2.3
        c2600-advipservicesk9-mz.124-15.T1.bin
```

#### FTP
```sh
    ip ftp username {username}
    ip ftp password {password}
```

#### Block Failed Attempts in vty 
```sh
    login block-for {ban-period} attempts {times} within {attempt-period}

    login quiet-mode access-class {acl-name}

    login delay {sec}

    login on-failure log 
```

#### enable linux shell 
```sh
    shell processing full # enable globally 
    terminal shell # enable for this terminal
```

#### Limiting Command Availability 
<https://www.jannet.hk/zh-Hant/post/authentication-authorization-accounting-aaa/>
```sh
    privilege {mode} {level | reset} {command}

    show privilege
    enable {level}

# examples:

    privilege exec level 5 ping 
    
    enable algorithm-type scrypt secret level 5 Skills39
    enable secret level 5 passwd
    username admin privilege 10 algorithm-type scrypt secret Skills39

    enable secret level {lvl} {pw} # enable secret 

    line con 0 
        privilege {lvl}

    enable {lvl} # enable privilege
    disable {lvl} # disable privilege
    show privilege
```

#### enable http server 
<https://www.cisco.com/c/en/us/td/docs/ios/12_2sb/feature/guide/sb_http1.html>
```sh
    ip http server 
    ip http authentication {}
    ip http max-connection {}
    ip http port {port-number}
    ip http path {path}
    ip http access-class {acl-name}

    ip http secure-server # https
```

#### dns server 
<https://leo19950830.pixnet.net/blog/post/215762307-cisco-router-%E8%A8%AD%E5%AE%9Adns-server>
```sh
    ip dns server 
    ip domain-lookup 

    ip name-server {ip} # external dns server 
    ip host sivs.local 192.168.0.1 # local record
```
#### ftp server 
<https://ithelp.ithome.com.tw/articles/10159670>
```sh
    ftp-server enable
```
#### Netflow
```sh
    int g0/1 
        ip flow ingress
        ip flow egress
    ip flow-export destination {ip} {port}
    ip flow-export version {version. 1 | 5 | 9}
```
#### SNMP
```bash
    snmp-server community {password} (perm)
    snmp-server {?}

    snmp-server host {ip} version {ver.} {password}
    snmp-server enable traps
```
#### Misc
```sh

# set history size
    terminal history size {size}

    line con 0 
		history size {size}

# disable password recovery
    no service password-recovery

# disable Translating ...domain server (255.255.255.255)
    no ip domain-lookup

# set minimum password length
    security passwords min-length {len}

# let line session show logs
    logging monitor

    terminal monitor

# disable error-disable 
    no errdisable detect cause all

    show users
	show version 
	traceroute {IP}

# interface range macro
    define interface-range {name} {if-set}
    interface range macro {name}

# backup ports
    interface f0/1 
        switchport backup interface f0/2 
    show interface switchport backup 
```

- IPv4 Multicast MAC Calculate:
    - prefix(*25 bits) 01:00:5E + IP(the last 23 bits) 
        example: 227.138.0.1 -> 01-00-5E-0A-00-01 

- IPv6 Multicast MAC Calculate:
    - prefix(2 bytes) 3333 + IP(the last 4 bytes) 
        example: FF02::1 -> 3333.0000.0001

- IPv6 Solicited Node Multicast MAC/IP Calculate:
    - FF02:0:0:0:0:1:FFXX/104
        - network address(24bits) same as Unicast Address(Global,Link-local)
            - example: fe80::10 -> FF02::1:FF00:0010
    - MAC Address: same as Multicast MAC 
        - 3333.FFXX.XXXX

