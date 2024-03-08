
[TOC]

---

### Static 

```sh
    show ip route (proto)

    ip route 目的地網路位址 遮罩 送出介面(自己) 下一站IP(別人)  (指定AD) 
    # Ethernet用的 需完全指定
    # example: ip route 172.16.1.0 255.255.255.0 G0/1 172.16.2.2

    ip route 目的地網路位址 遮罩 {下一站IP(別人) | 送出介面(自己)} (指定AD) 
    # 給專線用的
    # example: ip route 172.16.0.0 255.255.0.0 { s0/0/0 / 192.168.1.2 }
    
    # 0.0.0.0 : 匹配所有網路位址/網路遮罩

    # ip default-gateway {gateway} # !! 連接外部網路時必須設定 !! 

    ip route 0.0.0.0 0.0.0.0 {next-Interface | next-hop-IP} 
    # 預設靜態路由 用於對外網路 !! 不會被交換出去 需要另外設定 !! 
    ipv6 route ::/0 {next-Interface | next-hop-IP} # for ipv6

    clear ip route *
```

#### static route with sla (tracking object)
```bash
    ip sla {num}
        icmp-echo {ip}
        frequency {second} # check every ${second} seconds
        threshold {msec}
    ip sla schedule {num} start-time now life forever # start a sla

    show ip sla statistics

    track {num2} ip sla {num} state 
        delay {?}

    ip route 0.0.0.0 0.0.0.0 s0/0/1 track {num2}
```

### 預設AD

    rip(AD=120) update every 30 seconds, hold down 180, flushed after 240
    ospf(AD=110) update every 10 seconds, dead time is 40 seconds 
    eigrp(AD=90) update every 5 seconds, dead time is 15 seconds

### RIP

```sh
    # R1(config)# 
    router rip # R1(config-router)#
    version 2 # 啟用RIPv2 , 且僅 收/送 v2資訊

    no router rip # 關閉rip路由
    no version # 還原預設版本 , 即 送v1 收v1 v2

    show ip protocols # to check
    

    network {網路位址} # 使該符合網路位址的界面允許收/送RIP路由資訊 ex: 192.168.1.0
    # !!可以宣告靜態網路!!

    # 
    # 允許接收該網路位址送來的資訊 / 允許發送該網路位址的交換資訊
    # !! 網路遮罩會根據該網路的網路類別來計算(A B C類網路) 同一網段下的都算該網路位址 !!
    # !! 所以可以直接打自己介面的IP就好 !! 
    # ex: network 172.16.1.0 >> network 172.16.0.0  (B類網路)
    #           192.168.1.32 >>        192.168.1.0  (C類網路)   
    # 172.16.1.0/24 , 172.16.3.0/30 -> network 172.16.0.0
    #

    no auto-summary # !! 關閉自動摘要 , 當兩邊網路依照分類網路算下來後有衝突時應開啟
    # RIP EIGRP適用

    #
    # 例: 172.16.0.0 172.16.1.0 172.16.3.0 
    # 在自動摘要下會只有 172.16.0.0/16 (class B)
    #
    # !! RIPv1關閉自動摘要沒有用 , 因為RIPv1沒有網路遮罩的資訊 !!
    # 

    neighbor {ip} # on non-multicast network 

    int g0/1
        ip rip v2-broadcast

    distance {AD} # custom AD def. = 120

    debug ip rip events
    
    timers {?} # set timers

    passive-interface G0/0 # 在G0/0上設定被動介面 即只收不送

    passive-interface default # 設定預設被動
    no passive-interface G0/0 # 再獨立設定不需要被動的

    int g0/1 
        ip summary-address rip {network} {netmask}

    timers basic {update} {invalid} {hold-down} {flush}

# 加密:
    key chain test
        key 0
            key-string Skills39 

    int g0/1
        ip rip authentication mode md5
        ip rip authentication key-chain test

# offset:
        offset-list {acl} out {off}
# filter:
        distribute-list {acl} {in/out}
# 設定預設靜態路由方式:

    default-information originate # 傳播預設路由 (RIP / OSPF 適用)
    # 設定自己為預設對外路由 , 即允許自己的預設靜態路由轉發 讓其他路由器連外不需設定
    # redistribute static # EIGRP適用

    network 0.0.0.0 # 把預設路由當作靜態路由

    # not working method:

    ip default-network 0.0.0.0 # not working 

    redistribute static # 匯入靜態路由 not working 

```

### RIPng(for ipv6)
```sh
    ipv6 unicast-routing # !! 讓路由器允許傳送IPv6路由資訊 !!

    ipv6 router rip {domain-name}
        exit

    # !! ipv6協定路由都要在介面上設定 !! 
    int g0/1 
        ipv6 rip 自定域名 enable
        ipv6 rip 域名 default-information originate # 傳播預設路由

    show ipv6 route
    show ipv6 protocols

    ipv6 route 0::/0 {next-Interface | next-hop-IP} # 預設靜態路由
```

### OSPFv2 (Open Shortest Path First)
<https://www.jannet.hk/zh-Hant/post/open-shortest-path-first-ospf/>
```sh
    show ip ospf neighbor/database/interfaces
    show ip route

    clear ip ospf process 

    router ospf {process-id | usu. 1}

    ip ospf priority {0-255} # 在介面設定下使用
    # 此值用來決定DR/BDR 越大越優先 可以利用開機次序決定

    router-id rid # 指定路由器ID 
    # ID設定優先順序 : 自定ID > Loopback最大IP > 已啟用介面最大IP
    # router-id > loopback interface > ospf-enabled interface ip (srt.max)

    network 網路位址 wildcard/mask area 0

    #
    # wildcard 為 255.255.255.255 - 網路遮罩 !! 打網路遮罩也行 !! 
    # area 0為骨幹區域 , 設定single area網路時用0即可
    #

    passive-interface G0/0

    passive-interface default
    no passive-interface G0/0

    default-information originate # 預設路由 自治系統邊界路由器(ASBR)需設定
    # 預設靜態路由設定為 ip route 0.0.0.0 0.0.0.0 {next-Interface | next-hop-IP}

    #
    # 使用OSPF獲得的預設靜態路由會顯示: O*E2 (E: External)
    # E1會累計Cost(成本) , E2不會 成本算法: 10^8(參考頻寬) / 頻寬速度(bps)
    # 100Mbps+的都算1 , 小數點以後無條件捨去
    # 

    auto-cost reference-bandwidth {ref(Mbps)} # 調整參考頻寬 預設為100 (10^8)

    redistribute {proto} {specifications.} (metric-type (1 | 2*) ) (metric value) # type1會加入ospf的cost type2不會
    # redistribute routes from another protocol

    int g0/1 # 在介面下設定

        bandwidth kilobits 
        # 修改eigrp和ospf協定計算時Serial的頻寬度量 (登記的頻寬) 不會修改實際頻寬
        # 修改實際速度為在DCE端上用clock rate

        ip ospf cost value # 手動調整ospf成本

        ip ospf hello-interval {seconds}
        ip ospf dead-interval {seconds}
        # 修改hello dead時間間隔 修改hello時間間隔後dead間隔會自動變為hello的四倍
        # !! OSPF要求路由器兩邊的hello和dead間隔要相同才會傳送路由資料 !!

        ip ospf network {? type}

    # auth 
    # 不管是在介面或是區域啟動, 密碼都必須設在介面!
    router ospf 1 
        area {area} authentication (message-digest)

    int s0/1/0
        # 無加密密碼
        ip ospf authentication 
        ip ospf authentication-key {pw}
        
        # MD5加密
        ip ospf authentication message-digest
        ip ospf message-digest-key {id} md5 {pw}

    router ospf {proc-id}

        maximum-paths {number}

    # 壓縮路由 
        area {area} range {network} {mask} # 適用於ABR 

        summary-address {network} {mask} # ASBR External Route 匯入summary area

        no discard-route {internal | external} # 把指向Null0的Route取消掉

    # on non-broadcast networks
        neighbor {ip} (priority ) # 需要加上這行讓ospf知道鄰居

    # 以下設定須在所有參與該area的路由器下設定

    # stub
        area {area} stub (no-summary) # 自動設定預設路由, no-summary 設定totally stubby area
    # nssa not so stubby area
        area {area} nssa (no-summary) # 適用於有連接external的area

    # virtual-link 適用於沒有連接backbone的網路
        area {transit area} virtual-link {other's' Router-ID} # 兩邊都要做!!
        # 也適用於連接兩個Area 0


    # redistribute between different process-id
	router ospf 1
		redistribute ospf 2 subnets
	router ospf 2
		redistribute ospf 1 subnets

    # 在介面設定啟動
    int g0/1 
        ip ospf {proc-id} area {area}

```

### OSPF(Multi Area Specifics)
```sh
    #
    # 在Area0內的路由器稱為Backbone Router
    # 同時連接兩個以上Area的路由器稱為ABR(Area Border Router) 
    # 連接非OSPF協定的路由器稱為ASBR
    # 路由器只需要知道自己Area內的路由資訊即可
    #

    # 設定方法: 在設定network時把Area區別開來即可
```

### OSPFv3(for ipv6)
```sh
    ipv6 unicast-routing # !! 啟用ipv6路由轉發 !!

    ipv6 router ospf {proc-id} # 進入全域設定
        router-id A.B.C.D # !! 因為ipv6沒辦法自己設定router id 所以要先自定一個 !! 

    # !! ipv6類的路由都要在介面上設定 !! 
    int g0/1 
        ipv6 ospf {proc-id} area 0

    show ipv6 ospf neighbor
    show ipv6 ospf interfaces brief
    show ipv6 protocols
    show ipv6 route
```

### EIGRP 
<https://www.jannet.hk/zh-Hant/post/enhanced-interior-gateway-routing-protocol-eigrp/#leak>
```sh
    router eigrp {autonomous-system} # !! 不同路由器間編號需相同才能交換路由資料 !!

        network network-address (wlidcard/mask)

    show ip eigrp topology
    show ip eigrp neighbors
    show ip route 

    no auto-summary # 停用預設路由 EIGRP預設會用

    redistribute static # 設定預設路由 -> D*EX
    # 要先設定預設靜態路由: ip route 0.0.0.0 0.0.0.0 {next-Interface | next-hop-IP}
    # !! 指令與rip ospf不同 !!

    variance {multiplier} # 修改負載平衡度量 {multiplier}倍以內的路徑都會拿來用
    metric weights tos k1 k2 k3 k4 k5
    
    neighbor {} {if} # unicast method to send neighbor message

    int g0/1 
        # timer
        ip hello-interval eigrp {as-num} {sec}
		ip hold-time eigrp {...}

        ip bandwidth-percent eigrp {as} {percent} # max occupied bandwidth

    # 壓縮路由
    int g0/1 
        ip summary-address eigrp {as-id} {network} {mask}

        ip summary-address eigrp {as-id} 0.0.0.0 0.0.0.0
    

    # 或使用ip default-gateway
    ip route {network} {mask} {hop}
    ip default-network {network}
    router eigrp {id}
        network {network}
        
	# 加密傳送
    key chain {name}
        key {id}
            key-string {pw}
    int s0/1/0
        ip authentication mode eigrp {as-id} md5
        ip authentication key-chain eigrp {as-id} {name}

    eigrp stub {?mode} # set on stub router
```
### EIGRPv6
```sh
    ipv6 router eigrp {as}
        eigrp router-id {rid}
        no shutdown # !!
    int g0/1 
        ipv6 eigrp {as}

    show ipv6 eigrp neighbors/interfaces/topology
```
### BGP
```sh
    router bgp {as-number}
        neighbor {IP} remote-as {as-number}
        neighbor {IP} shutdown 

        network {network} mask {mask}
        network 0.0.0.0
    show ip bgp summary 
    show ip bgp ({network}))
```

### ACL 

#### 標準ACL

```sh
    # ACL分為入站ACL和出站ACL 每個ACL的末尾都會有一項預設deny
    # host = 萬用遮罩為0.0.0.0(只有一台) , any = 萬用遮罩為255.255.255.255(全部)
            
	access-list {number: 1-99} {permit/deny/remark} condition (??log)
        
    # 條件寫法: !! 由小到大設計 !!
        #   IP-addr  Wildcard
        #   any
        #   (host) sing-IP 
        
    no access-list {number} # !! 只能刪掉全部 !!
        
    # with specified name:
        
    ip access-list {extended / standard} name # -> Router(config-std-nacl)#
            permit/deny/remark condition (??log)
            no permit/deny/remark condition # 可以刪掉其中一個規則
            no 10 # 用序列號取消某條指令
        
    int f0/0
        ip access-group {number/name} {in: 進來時比對/out: 出去時比對} # link !! in和out是由路由器的角度看的 !!
        no ip access-group
        
    show access-lists
    show ip int g0/0
    clear access-list counters
```

#### ACL for vty 
```sh
    line vty 0 4
        access-class {number/name} in
```

#### 延伸ACL
```sh
        # !! 延伸ACL不會自動調順序 !!

    access-list {number} {permit/deny} {proto.} {src. cond.} {dest. cond.} {type spec.}

    ip access-list extended {name}
        no ip access-list extended {name}

    permit ip any any # 允許全部 因為所有協定都在IP之下

    permit tcp any any established

    # example for allow http/s: (103 in, 104 out)
        access-list 103 permit tcp 192.168.10.0 0.0.0.255 any eq 80
        access-list 103 permit tcp 192.168.10.0 0.0.0.255 any eq 443
        access-list 104 permit tcp any 192.168.10.0 0.0.0.255 established # !! important !!

    # example for mail server:
        access-list 104 permit tcp any 192.168.10.0 0.0.0.255 eq 25
        access-list 104 permit tcp any 192.168.10.0 0.0.0.255 eq 110
        access-list 104 permit tcp any eq 25 192.168.10.0 0.0.0.255 # !! 接受回傳資訊 !!

    # 設定ftp需要設定port 20 和 21

    # wildcard計算方式:
        # 例: 64~79
        # 79-64+1 = 16 (16個位址)
        # 64 % 16 = 0 (不會衝突)
        # wildcard = 0.0.0.15 (16-1: 0~15)

    # protocol = ip
        # condition = host ip / ip wildcard / any 

    # protocol = tcp/udp
        # condition = {host ip / ip wildcard / any} {neq/eq/lt/gt/range...} {port/app name}
        # tcp後面可以加上"established"標籤

    # protocol = icmp 
        # condition = {host ip / ip wildcard / any} {?type}
        # echo: 傳送ping , echo reply: 回應
```

#### IPv6 ACL

- 與ipv4 ACL的差異:
        1. ipv6只有延伸命名ACL

    2. 介面套用命令與v4不同

    3. ipv6的規則不用wildcard 是用prefix

    4. 預設隱藏敘述多兩條permit:
        ```sh
            # 作用同ipv4的ARP 用途為尋找目的地主機mac位址
            # ARP: 知道別人L3資訊 想要查別人L2的資訊
            permit icmp any any nd-na # ICMP鄰居請求
            permit icmp any any nd-ns # ICMP鄰居通告
        ```
        

```sh
    ipv6 access-list {name} 
        permit/deny ... # 與ipv4相同
        # 位址寫法為 IP/Prefix

    int g0/1 
        ipv6 traffic-filter {ACL name} {in/out}
    
    line vty 0 4
        ipv6 access-class {ACL name} in # ACL on vty 

    show ipv6 interface g0/0
        show access-lists
```

### DHCPv4
DHCP使用UDP協定 在Application Layer, 且port為67(Server)/68(Client)

DHCP分為四個動作:
```sh
    Server   ::     Client 
            <<<<    Discover(廣播Broadcast) 
    Offer   >>>>    (usu. 單點傳送Unicast)
            <<<<    Request (廣播)
    ACK     >>>>    (usu. 單點傳送)
```
廣播資訊為 from 0.0.0.0 to 255.255.255.255 Dest. MAC ffff.ffff.ffff
租約更新只需要後面兩個動作 且都為單點傳送

若Client端請求DHCP失敗便會得到169.254.0.0/16網段的IP位址(APIPA)

```sh
    ip dhcp excluded-address {low-address} (high-address) 
    # 排除的IP(range) conf模式下設定
    # !! 只能設定排除 不能限定供應範圍 !! 

    ip dhcp pool {name}
        network {ip} {mask}
        default-router {router}
        dns-server {server}
        domain-name {name}
        lease {period: days hours minutes | Inf , default: 1day}

# static ip 
    ip dhcp pool Static
        host {ip} {mask}
        client-identifier {show ip dhcp binding}

    no service dhcp
    service dhcp

    show running-config | section dhcp
    show ip dhcp binding
    show ip dhcp server statistics
    debug ip dhcp server events
    show ip dhcp conflict


# DHCP Relay (中繼DHCP資訊)
    int g0/1 # 需要轉發廣播的介面
        ip helper-address {ip} # 轉發去的IP位置
    service dhcp # important !! 
    show ip int g0/1

# 把自己設定成DHCP Client
    int g0/1
        ip address dhcp
        no shutdown
```

### IPv6

- SLAAC (StateLess Address Auto Configuration)

    1. 客戶端設定為自動取得IPv6位址
    1.  (送出)路由器請求Router Solicitation(>> FF02::2 只給路由器) 跟 
        (回覆)路由器通告Router Advertisement(>> FF02::1 類廣播位址 multicast) 
        來提供IPv6的"網路"位址(不需使用DHCP)
    1. 確定旗標狀態為使用SLAAC

        - 使用ICMPv6 RA訊息中的 M(管理)旗標 和 O(其他)旗標 來決定要用哪一種方式:
            SLAAC: 0 0
            無狀態DHCPv6 (SLAAC定址 + DHCPv6): 0 1
            有狀態DHCPv6: 1 0

    若確認為使用SLAAC:

    1. 客戶端自動產生後64位元的網路位址(eui-64 或 隨機) 
    1. 發出ICMPv6 Neighbor Discovery資訊
    1. 確認沒人使用後就會設定自己的位址(Duplicate Address Detection)
    1. 並將閘道位址改成路由器的Link-Local Address(FE80::/10)

```sh
    # 純SLAAC為Cisco路由器上的預設選項: M=0 O=0
    ipv6 unicast-routing # 讓路由器發送RA資訊要先執行
    ipv6 enable # 開啟ipv6 產生ipv6 Link-Local位址(不需先設定IPv6位址)

    # 在介面下設定:
    ipv6 nd managed-config-flag # 設定M=1
    no ipv6 nd managed-config-flag
    ipv6 nd other-config-flag # 設定O=1
    no ipv6 nd other-config-flag
```

### DHCPv6

DHCPv6使用UDP協定 在Application Layer, 且port為547(Server)/546(Client)

1. 透過RA(SLAAC)取得形態
若為有/無狀態DHCPv6:
1. 將DHCPv6 SOLICIT訊息發送到 FF02::1:2 (給所有DHCPv6 server)
1. 伺服器將用DHCPv6 ADVERTISE訊息回覆 (此和以下皆為單點傳送)
1. 客戶端回應
    - 若為無狀態DHCPv6: 
        客戶端回覆 DHCPv6 INFORMATION-REQUEST (只要求額外參數)
    - 若為有狀態DHCPv6: 
        客戶端回覆 DHCPv6 REQUEST (要求資訊包含IP)
1. 伺服器回覆DHCP REPLY


```sh
# 無狀態DHCPv6伺服器設定:

    ipv6 unicast-routing # 啟用路由 傳送ICMPv6 RA資訊

    ipv6 dhcp pool {name}
        dns-server {server}
        domain-name {name}

    int g0/1
        ipv6 dhcp server {name}
        ipv6 nd other-config-flag
    

    show ipv6 interface
    show ipv6 dhcp pool
    debug ipv6 dhcp detail
    undebug all 

    # 有狀態DHCPv6伺服器設定:

    ipv6 unicast-routing

    ipv6 dhcp pool {name}
        address prefix {prefix/length} (lifetime {?time}) # 分配IP 
        dns-server {server}
        domain-name {name}
        # !! 預設閘道會直接用Router的鏈路本地位置 所以不需特別設定 !!

    int g0/1
        ipv6 dhcp server {name}
        ipv6 nd managed-config-flag # M=1 O=0


    show ipv6 interface
    show ipv6 dhcp pool
    show ipv6 dhcp binding
    show ipv6 dhcp conflict
    debug ipv6 dhcp detail
    undebug all 

    # DHCPv6客戶端設定:
    
    ipv6 unicast-routing # ??
    int g0/0
        no shutdown
        ipv6 enable # 取得鏈路本地位址
        ipv6 address {autoconfig/dhcp} # 無狀態或有狀態

    show ipv6 interface
```

#### DHCPv6 Relay (not working in packet tracer)
```sh
    int g0/1
        ipv6 dhcp relay destination {DHCPv6Srv-Address}
    show ipv6 dhcp interface
```

### NAT (Network Address Translation)
<https://www.jannet.hk/zh-Hant/post/network-address-translation-nat/>

- NAT使用RFC1918中規定的私有IPv4位址來實作
私有IPv4位址A B C類中各有一段:
    - 10.0.0.0    - 10.255.255.255    / 8
    - 172.16.0.0  - 172.31.255.255    / 12
    - 192.168.0.0 - 192.168.255.255   / 16

- NAT包括四類地址:
    - Inside Local Address
    - Inside Global Address
    - Outside Local Address
    - Outside Global Address

inside: 內部經NAT轉換的位址
outside: 目的裝置的位址
內網 src: 內部本地位址, dst: 外部本地位址 >> NAT >> 外網 src: 內部全域位址, dst: 外部全域位址

- NAT轉換有三種類型:
    - 動態NAT
    - 靜態NAT
    - 連接port位址轉換(PAT Port Address Translation) (NAT overload)

#### Static NAT
```sh
    ip nat inside source static {local-ip} {global-ip}

    int f0/1
        ip nat inside
    int g0/1
        ip nat outside
    

    show ip nat translations

    show ip nat statistics
    clear ip nat statistics
```

#### Dynamic NAT
```sh
    ip nat pool {name} {start-ip} {end-ip} netmask {mask} # 設定公用位址區段
    ip access-list standard {acl-name} # 設定標準ACL來允許哪些IP可以轉換
        permit {allowed-ip}
    ip nat inside source list {acl-name} pool {pool-name} # 設定ACL與pool的繫結

    int f0/1
        ip nat inside
    int g0/1
        ip nat outside

    show ip nat translations verbose
    clear ip nat translation *
```

#### Port Address Translation

- 一個IP大概只能記錄4000個連線
- 如果原本轉換出去的port已被佔用 , PAT會從相應連接port組(0-511 512-1023 1024-65535)的開頭開始分配第一個可以用的port編號 如果該區port被用完就會找儲存區內的其他IP
- 如果是ICMP , PAT會依據裡面的查詢ID來識別

```sh
# 其他設定方法跟Dynamic NAT一樣
    ip nat inside source list {acl-name} pool {pool-name} overload # 用pool
    ip nat inside source list {acl-name} interface {if-name} overload # 送到固定介面的IP

# Port Forwarding (Tunneling) 設定
    ip nat inside source static {tcp/udp} {local.IP local.port global.IP global.port} (extendable)

# 設定load-balancing
    ip nat inside destination list {acl-name} pool {nat-pool}
    ip nat pool {name} {from} {to} { prefix-length [len] / netmask [mask] } type rotary
    #access-list ..

# or just a bunch of ports to be Forwarding (seems not to be working)
    ip nat inside destination list {acl-name} pool {nat-pool}
    ip nat pool {name} {ip} {ip} { prefix-length [len] / netmask [mask] } type rotary
    # set up a acl with port ranges
    ip access-list extended {acl-name} 
        #permit tcp/udp any any range {from} {to}


    show ip nat translations
    debug it nat (detailed)
```

- IPv6
    - IPv6唯一本地位址(Unique Local Addresses, 類似RFC1918私有位址)範圍為 FC00::/7 ( FC00:: - FCFF:: )
    - IPv6轉換方式分為雙重堆疊、隧道和轉換

### FHRP (First Hop Redundancy Protocol)

#### HSRP (Hot Standby Redundancy Protocol)
<https://www.jannet.hk/zh-Hant/post/first-hop-redundancy-protocol-fhrp/#hsrp>
```sh
    # HSRP的vMAC位址有 0000.0C07.ACXX(v1) 以及 0000.0C9F.FXXX(v2) X為群組編號

    int g0/1
        standby {id} ip {vip}
        standby {id} preempt # let router with the highest priority to become the active router immediately
        standby {id} mac-address {mac-addr}
        standby {id} priority {pri. num srt. bigger} 
        standby {id} timers {hello} {hold} 
        standby {id} track {if} (decrease-value default. 10) # pri. gained -10 decreasement it that interface is down
        standby {id} authentication {?}

        # standby timers msec 250 msec 750
    
    show standby (brief)

# HSRP version 2 
    int g0/1 
        standby version 2
        standby {id} ipv6 {link-local-addr | autoconfig}
        #...
###
```

- VRRP (not supported in PT)
```sh
    # vMAC= 0000.5E00.01XX

    vrrp {id} ip {VIP/IP}
    vrrp {id} priority {pri.}
    vrrp {id} authentication {?}

    show vrrp (brief)
```
- GLBP (not supported in PT)
```sh
    # vMAC= 0007.B400.XXYY where XX= group id, YY= router id

    int g0/1
        glbp {id} ip {vip}
        glbp {id} preempt # let router with the highest priority to become the active router immediately
        glbp {id} priority {pri. num srt. bigger} 
        glbp {id} timers {hello} {hold} 
        glbp {id} authentication {?}

    show glbp
```
### Radius 
```sh
    aaa new-model

    radius-server host {ip} key {key}

    aaa group-server radius {?}

    # 遠端登入的時候用RADIUS驗證，如找不到RADIUS使用LOCAL
    aaa authentication login {default/name spec.} group radius local 

    #切換enable的時候用RADIUS驗證，如找不到RADIUS使用LOCAL
    aaa authentication enable default group radius local

    line vty 0 4 
        login authentication default
        transport input {method}
```
### Tacacs
```sh
    aaa new-model

    tacacs-server host {ip} key {key}
```

### logging 
```sh
# logging to syslog server 
    logging on 
    logging host {srv. ip} # logging {srv. ip}

    logging console 
    logging monitor
    logging buffered
    logging trap {level} # show logging levels within the level and above

    service timestamps # show timestamp on logs
    service sequence-numbers
    service timestamps log datetime localtime # add localtime to log 
    service timestamps log datetime msec

    show logging
```
#### key chain
```sh
    key chain {name}
        key {number}
            key-string {password}
            accept-lifetime {time}
            send-lifetime {time}
```
### Tunnel VPN Technologies
#### GRE Tunnel(VPN with no encryption)
```sh
    int tunnel {number}
        ip address {} {} 
        tunnel mode gre ip 
        tunnel source {out-if/out-ip}
        tunnel destination {des-ip}

    ip route {} {} {tunnel-if/tun.dstIP} 
```
#### ipv6ip Tunnel 
```sh
    int tunnel {number}
        tunnel mode ipv6ip
        ipv6 addr {ip/prefix}
        tunnel source {if6-name}
        tunnel destination {ipv4-ip}
```
#### IPv6 Automatic 6to4 Tunnel
<https://www.netadmin.com.tw/netadmin/zh-tw/technology/916B0672300D4D70B642EE1ED34794A4?page=1>
<https://www.jannet.hk/zh-Hant/post/IP-Address-Version-6-IPv6/#6to4>
<https://community.cisco.com/t5/networking-documents/ipv6-6to4-tunneling-configuration-example/ta-p/3139227>
```sh
    # need specific ip 2002:{ipv4:address}::/48
    int tunnel 0 
        ipv6 address 2002:{ipv4:address}::/48
        tunnel source {ipv4-address}
        tunnel mode ipv6ip 6to4

    ipv6 route {ipv6-tunnel destination} tunnel0
    ipv6 route {} {ipv6-tunnel destination}
```
#### isatap
```sh
# isatap server 
    int tunnel 0 
        ipv6 address {ip/pref}
        tunnel source {src-srv-IP}
        tunnel mode ipv6ip isatap
 
# client 
    int tunnel 0 
        ipv6 address autoconfig
        tunnel source {IP}
        tunnel destination {src-srv-IP}
        tunnel mode ipv6ip
        
```
#### multipoint GRE (mGRE)
```sh
    # set up NHRP
    # ...
    show ip nhrp

    int tun 0 
        ip addr {} {}
        tunnel source lo0
        tunnel mode gre multipoint
```
#### IPSec VPN
<https://www.jannet.hk/zh-Hant/post/internet-protocol-security-ipsec/>
```sh
# !! the steps follow have to be configured on both peer !!

# Step 1. Configure ACL 
    ip access-list extended VPN-Traffic
        permit ip {} {} {} {} 

# Step 2. Configure IKE Phrase 1 !! both peer should be paired !!
    crypto ISAKMP policy 1 
        encryption aes 
        hash {md5}
        authentication pre-share
        group 2 
        lifetime 30000 

# Step 3. Configure IKE Phrase 2 
    crypto ipsec transform-set {TS} {esp-3des ah-sha-hmac}

# Step 4. Configure Pre-Shared Key 
    crypto isakmp key 6 {PreSharedKey} address {another peer's' IP}

# Step 5. Configure Crypto Map
    crypto map {CMAP} 1 ipsec-keymap # bind phrase 1
        set peer {another peer's' IP} # bind Pre-Shared key 
        set transform-set TS # bind phrase 2 
        match address VPN-Traffic # bind ACL
    int g0/0
        crypto map CMAP 
```

### HDLC(High-Level Data Link Control) encapsulation (when connecting two cisco devices)
```sh
    int s0/1/0
        encapsulation hdlc

    show interface s0/1/0
    show controllers
```

### ppp(Point to Point Protocol) encapsulation 

<https://www.jannet.hk/zh-Hant/post/point-to-point-protocol-ppp/>

```sh
    int s0/1/0
        no shu
        ip addr {} {} 
        encapsulation ppp
        ppp quality {%} # if this link doesn't meet the % of the quality, the link will close down
        compress {?}

        ppp authentication pap 
        ppp authentication chap

        ppp authentication pap chap # if pap failed, then try chap authentication
        ppp authentication chap pap # if chap failed, then try pap authentication

    show int s0/1/0
    show controller s0/1/0
```

#### PPPoE 
```sh
# Server(ISP)
    ip local pool Po1 10.0.0.100 10.0.0.200 
    username client password Skills39
    int virtual-template 1 
        ip address 10.0.0.1 255.255.255.0
        peer default ip address pool Po1 
        ppp authentication chap 

    bba-group pppoe Group1
        virtual-template 1
    
    int g0/1 
        pppoe enable group Group1 
        no shutdown 

# Client
    interface dialer {number}

        ip address negotiated
        ip mtu 1492 # ppp preamble occupied 8 bytes
        #mtu 1492 
        
        encapsulation ppp
        ppp chap hostname {name} 
        ppp chap password {pw}
        
        dialer pool {number}
        # no shutdown
    
    int g0/1
        pppoe enable 
        pppoe-client dial-pool-number {number}

        no ip address
        no shutdown

        ip tcp adjust-mss {max-seg-size opt.1452}

    show ip int brief
```

#### multilink ppp (mlppp)
```sh
    interface multilink {number}
        encapsulation ppp
        ppp multilink
        ppp multilink group {number} # the number should be the same 

        ip addr {} {}
        # auth...

    int s0/1/0
        no shutdown 
        no ip addr 

        encapsulation ppp
        ppp multilink
        ppp multilink group {number}


    show int multilink {number}
    show ppp multilink
```

#### PAP(Password Authentication Protocol)
```sh
    #Router1#
    username {usr1} password {pw1}
    int s0/1/0
        encapsulation ppp
        ppp authentication pap
        ppp pap sent-username {usr2} password {pw2}
        
    #Router2#
    username {usr2} password {pw2}
    int s0/1/0
        encapsulation ppp
        ppp authentication pap
        ppp pap sent-username {usr1} password {pw1}

    debug ppp authentication 
```

#### CHAP(Challenge-Handshake Authentication Protocol)
```sh
    # !! 兩台路由器都要做一樣的設定 兩邊的密碼要相同 !! 
    username {another router's' hostname} password {same as another router's' password}
    int s0/1/0 
        encapsulation ppp 
        ppp authentication chap
```

### set default gateway on a non-routing router
```sh
    no ip routing # disable routing 
    ip default-gateway {IP}
```

### Loopback Adapter
```sh
    interface loopback {num}
        ip address 192.168.0.1 255.255.255.255
    no int lo{num}
```
### arp
```
	show arp
	
	int g0/1
		arp timeout {time}

    no ip proxy-arp
```












### Frame Relay (Obsoleted, no longer in CCNA RS)
<https://www.jannet.hk/zh-Hant/post/frame-relay-switching/>
```sh
    int s0/0/0
        encapsulation frame-relay
        frame-relay interface-dlci {number}

        frame-relay map {ip} {number}
    show frame-relay pvc # Permanent Virtual Circuit
    show frame-relay map 
```