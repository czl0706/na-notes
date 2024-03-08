
[TOC]

---

### Switching
```sh

    mdix auto
    no mdix auto

    # !! 網路分享器的 LAN端跟switch一樣 12rx 36tx , WAN端則跟電腦一樣 12tx 36rx !!

    vlan vlan_id
        name {name}

    interface vlan 1 # vlan 1 has enabled by default
        ip address {ip} {mask}
        no shutdown

    interface f0/0 
        switchport access vlan vlan_id # combine a vlan to a real interface

    show ip interface brief

    ip default-gateway {IP} # 設定預設閘道 !! 連接外部網路時必須設定 !! 

    show interface {if-name} switchport
    show interface trunk
```

### Vlan
```sh
    # vlan標記會在frame的 mac位置(來源) 和 type 之間插入4bytes的資料
    # 前兩個byte為標記為802.1q (0x8100)
    # 後兩個中的前4個bits為順序(3)和識別碼(1) 後面12個為vlan的id 所以支援4096個vlan id
    show vlan (brief)
    show interfaces trunk

    switchport mode trunk # 更改介面為永久主幹模式
    switchport mode access

    switchport trunk allowed vlan {?vlan-list ex. 10,20,30-40}
    switchport trunk native vlan {vlan-id}

    no switchport trunk allowed vlan 
    no switchport trunk native vlan 

    show interfaces {if-name} switchport
    show interfaces trunk

    vlan 100 
		shutdown

    # https://www.jannet.hk/zh-Hant/post/virtual-lan-vlan/
    vlan internal allocation {?} # L3 Switch 
    show vlan internal usage 

    delete flash:vlan.dat

# vlan types

	# normal vlan (0-1005)
	switchport access vlan {num}
	# voice vlan 
	switchport access vlan {num}
	switchport voice vlan {num}
	# extended vlan (1006-4094, saved to running-config)
	vtp mode transparent
	vlan 1006

# private vlan 

	vlan {num}
		private-vlan {?mode}
		private-vlan association {numset}
```

#### VTP(Vlan Trunking Protocol)
<https://www.jannet.hk/zh-Hant/post/vlan-trunking-protocol-vtp>
```sh
    # 交換器vlan預設模式為server 沒有vtp domain name 版本編號只要每次更改就會加一
    # vtp需要靠主幹通道才能互相交換資料
    show vtp status

    vtp domain {domain-name} # 設定domain name, 啟用vtp server
    # 只要在其中一台設定好domain name , 全部交換器就會套用(!! 更新以版本編號最大的作為標準 !!) 
    # !! 設定domain的那台機器版本編號會變回0 !!

    vtp mode {mode}
    # VTP mode 分為server(儲存vlan.dat) client(每次開機都跟server更新) transparent(轉送收到的更新資訊 但自己不更新) 三種 !! 設定為transparent以後版本編號會歸零 !!
    # 處於Transparent模式的Switch會將Vlan存在config檔裡 !!
    # 必須要是Trunk mode才可以傳送VTP更新資訊

    # 更改domain name可以讓VTP Revision編號重置

    vtp password {pw}
    vtp version {ver.}

    vtp primary vlan # works on vtp version 3

    vtp pruning # 修剪vlan trunking

    show vtp status
    show vtp password
```

#### Vlan Routing
```sh
    # 原生vlan傳送資料不需標記
    # 管理vlan需在SVI上設定IP mask 
    # 802.1Q

    vlan {vlan-id} # 啟用vlan
    name {name}
    
    no vlan {vlan-id} # 刪除後原本連接port應手動加回
    delete flash:vlan.dat # 刪除vlan設定

    show vlan (brief)

    interface range f0/1 - 15
        switchport mode access # 使該port標記為連接vlan的 避免DTP(Dynamic Trunking Protocol)
        switchport access vlan {vlan-id}

        no switchport access vlan

        switchport mode trunk # !! 設定該介面為主幹道路 !!
        switchport trunk native vlan {vlan-id} # 設定該vlan為原生vlan !! 每個trunk介面都要設定 !!

    vlan dot1q tag native # disable native vlan tag
```

- 設定傳統vlan

    > 即將每個vlan都拉一條線至路由器上(且設定完介面ip)即可 
    > 因為沒有連接其他網段 所以只需要連接上去即可
    
        v1---|
        v2--Router
        v3---|
    
- 設定單臂路由器

    > Router只需要連接一個介面 一個介面即可接通所有vlan ! 不能超過50個 

    ```sh
        # concept : 將一個實際介面(ex g0/0)切成數個子介面(ex g0/0.10) 
        
        # 在實際介面上設定 :
        int g0/0
            no shutdown # no ip address

        # 子介面 :
        int g0/0.{sub-interface_id (usu. vlan-id)}
        # !! 以下順序不能顛倒 !!
            encapsulation dot1q {vlan-id} (native) # !! 設定該介面封裝協定為802.1Q !!
            ip address {IP} {mask} # 與一般設定一樣
        
    ```
- L3 Switch Routing
    ```sh
        vlan 10 
        int vlan 10 
            ip address {} {}

        vlan 20 
        int vlan 20
            ip address {} {}
        
        vlan 30 
        int vlan 30 
            ip address {} {}

        ip routing
    ```
- 設定多層交換器(Layer3 Switch 或 MultiLayer3 Switch)

    ```sh
        vlan {id}

        interface vlan {id}
            ip address {IP} {mask}
            no shutdown 

        int g0/1 
            switchport trunk encapsulation dot1q # !! 更改協定 !! 
            switchport mode trunk
            switchport trunk native vlan {native-vlan}
        
        exit 
        ip routing # !! 開啟layer3 routing !! 

        show interfaces (int {if-name}) trunk
    ```

    ```sh
        no switchport # 讓該介面變成和Router一樣的port 可以直接設定IP
        ip routing # !!
    ```

### SSH 
```sh
    show ip ssh # 檢查是否支援

    ip domain-name {domain_name} # 產生金鑰必需
    hostname {hostname} # 不能為預設

    ip ssh version 2

    crypto key generate rsa # 產生金鑰 並啟用ssh

    username {username} password/secret {password}
    
    line vty 0 15 

        transport input ssh # 允許ssh登入並關閉telnet 在vty下設定
        login local # 設定vty從本地資料庫獲取帳號密碼
```

### Spanning Tree Protocol(cisco PVST)
<https://www.jannet.hk/zh-Hant/post/spanning-tree-protocol-stp/>
<https://www.jannet.hk/zh-Hant/post/rapid-spanning-tree-protocol-rstp/>
```sh
    # !! Bridge ID和ospf的比較不同 是小的優先!! (router是用IP switch是用MAC位址) !!
    # 選root bridge Switch: Priority > Mac Address (Bridge ID = Priority << 48 + mac_addr)
    # 選port: root cost > (對方的)bridge ID > (對方的)Port ID


    # blocking -> listening -> learning -> forwarding
    # port根據STA分為 根連接 指定連接 非指定連接 三種port
    
    # PVST          ->  IEEE 802.1D
    # Rapid-PVST    ->  IEEE 802.1w

# Per Vlan Spanning Tree (pvst)

    spanning-tree mode rapid-pvst # change mode to rstp
    
    spanning-tree vlan 1 root primary # 讓此交換器在 vlan 1 下成為樹根

    spanning-tree vlan 1 root secondary # 備用

    spanning-tree vlan 1 priority {4096.mul default.32768}

    int g0/1

    # port type

        # full duplex port -> p2p non-edge port 
		# half duplex port -> shr non-edge port 

        # set up edge port 
        spanning-tree portfast # 讓該在交換器開機時不需要等待(跳過30秒檢查 可能會有迴路風險)

        # switchport host # predefined macro 

    # guard stp topology 

        spanning-tree guard root # 保護原有的root 如果priority比較低就block 避免後來新增的switch改變整個topology
        # spanning-tree rootguard

        spanning-tree bpduguard enable # 只要有switch插入, 收到bpdu後就直接關上該port
        spanning-tree bpduguard disable

        spanning-tree bpdufilter enable # 停止發bpdu 當收到bpdu時就恢復正常STP

        spanning-tree guard loop # 避免loop
        # guard loop(disable port when it doesnt receive bpdu)

# Advanced STP behavior 

    spanning-tree uplinkfast # 設定一個正處於blocking的port為standby狀態 適用於directly fail

    spanning-tree backbonefast # 適用於indirectly fail
    # 當使用BackboneFast功能時，必須在所有交換器上均啟用BackboneFast功能，否則會有 RLQ(Root Link Query) 處理問題。

    spanning-tree vlan {vlan-id} hello-time {seconds}

    # spanning-tree vlan {vlan-id} forward-time {seconds}

    # spanning-tree vlan {vlan-id} max-age {seconds}

    spanning-tree loopguard default # guard loop(disable port when it doesnt receive bpdu)

    # portfast
    spanning-tree portfast default

    # spanning-tree edge portfast default

    spanning-tree portfast bpduguard default

    spanning-tree portfast bpdufilter default
    # spanning-tree portfast edge bpdufilter default

    spanning-tree vlan {vlan} root primary diameter {hop} # set optimized timer

    # disable stp on vlan 1 
    no spanning-tree vlan 1



    show spanning-tree (summary)

    show spanning-tree vlan {vlan-set}

    int g0/1
        spanning-tree link-type ?
        spanning-tree link-type shared # enforce type to be shared
        
        # 以下設定皆為更改自己的priority
        spanning-tree cost {value}
        spanning-tree vlan {vlan} cost {cost}

        spanning-tree port-priority {pri.}
        spanning-tree vlan {vlan} port-priority {priority} # 更改port ID


# udld(undirectional link detection) uses on fiber connection
    # globally enable 
	udld enable # turn undirectioned port to undetermined state
	udld aggressive # to err-disable state
	# in specific port 
	int g0/1 
		udld port 
		udld port aggressive 
	show udld 
```

### DTP(Dynamic Trunking Protocol)
<https://www.jannet.hk/zh-Hant/post/dynamic-trunking-protocol-dtp/>
```sh
    switchport mode dynamic {desirable/auto}

    show dtp
	
    int g0/1
	    switchport nonegotiate # disable DTP message
```

### Security
```sh
    interface range f0/1 - 3 # 選定範圍內界面 只能選定模組範圍內編號
```

#### DHCP snooping (DHCP 偵聽 避免偽造dhcp server)
```sh
    ip dhcp snooping
    ip dhcp snooping vlan {number}

    no ip dhcp snooping information option # 如是用cisco的router實作dhcp需開啟

    int g0/1
        ip dhcp snooping trust # 允許該介面上的DHCP server

        ip dhcp snooping limit rate {rate} # 限制該介面對dhcp請求速度

    int range f0/1-24 
        ip verify source # 檢查Host使用的IP是否為DHCP得到的 如果不是就不轉送
        ip verify source port-security # 連MAC位址也檢查 要先開port-security
    ip source binding {?}

    show ip dhcp snooping biding 
    show ip verify source
```

#### Port Security (避免MAC Address Table Flooding Attack)
```sh
    switchport mode access # 更改為access模式

    switchport port-security # 設定動態安全MAC位址 記錄第一個學習到的MAC位址 預設只能有一個 超過預設動作為shutdown

    switchport port-security maximum {number} # 當記錄mac位置 > 此數就會違規

    switchport port-security violation {?mode} # 設定違反後的動作
    # protect: 原MAC可用, 不記錄 restrict: 原MAC可用, 並將違規記錄 *shutdown: 關閉該介面(error-disable)

    switchport port-security mac-address sticky # 學到的MAC位址會存進running-config中 
    # switchport port-security mac-address sticky {mac-address}

    switchport port-security mac-address {mac-address} # 設定靜態安全MAC位址 

    switchport port-secutiry aging type {?type}

    switchport port-secutiry aging time {?time}

    show port-security

    show port-security interface {int}

    show port-security address

    show mac-address-table

    clear mac-address-table

    clear port-security all
```

### EtherChannel (Port Channel, Link Aggregation)

<https://www.jannet.hk/zh-Hant/post/etherchannel-pagp-lacp/>

- 一次最多可以將8個Port組合成一個EtherChannel 剩下的會當替補(替補最多也是8個)


```sh
    port-channel load-balance {?mode} # 設定負載平衡模式

    show etherchannel load-balance
    show int port-channel 1 
    show etherchannel summary
    show etherchannel port-channel
    show spanning-tree
```

#### Static On Mode
```sh
    int range g0/1-2 # 在兩台機器上設定
        shutdown
        channel-group 1 mode on 
        no shutdown
    
    port-channel load-balance {?}
```

#### Port Aggregation Protocol (PAgP)
```sh
    int range g0/1-2 # 在兩台機器上設定
        shutdown
        channel-group 1 mode {desirable/auto} # 設定為 主動/被動 模式 一邊主動一邊被動即可
        no shutdown
```

#### Link Aggregation Control Protocol (LACP,802.3ad)
```sh
    int range g0/1-2 # 在兩台機器上設定
        no channel-group 
        shutdown
        channel-group 1 mode {active/passive} # 設定為 主動/被動 模式 一邊主動一邊被動即可
        lacp port-priority {priority. sorted by min} # 設定優先權
        no shutdown        
```

#### L3 EtherChannel
```sh
    int port-channel 1 
        no switchport
        ip address 10.0.0.1 255.255.255.0
    int range g0/1-2
        no switchport 
        channel-group 1 mode {?mode}

    ip routing
```

### Storm Control
```sh
    int g0/1
        storm-control broadcast level {? (meter. default %) trigger (threshold)}
        storm-control broadcast level {trigger level %}
        storm-control action {?}

    show storm-control
```

### enable SPAN(Switched Port Analyzer)
```sh
# local span
    monitor session {number} {source | destination} {interface int.| vlan vlan.}
# remote span 
    vlan 99
        remote-span 
    monitor session {number} destination remote vlan 99

    show monitor session {id}
```

### errdisable recovery
```sh
    errdisable recovery cause security-violation

    errdisable recovery interval {sec}

    show errdisable recovery
```