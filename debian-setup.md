### Basic setup
```sh

# apt install network-manager 
# systemctl disable NetworkManager
apt install net-tools
apt install vim 
apt install 

vim /etc/inputrc
# set bell-style none

vim /etc/vim/vimrc
# set nu
# set belloff=all

# Change Default Network Name to ethx
vim /etc/default/grub
# GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
uptate-grub

set-timezone Asia/Taipei

vim /etc/network/interfaces
# # The primary network interface
# auto eth0
# iface eth0 inet dhcp

# auto eth1
# iface eth1 inet static
#     address 192.168.2.254
#     netmask 255.255.255.0

vim /etc/sysctl.conf
# net.ipv4.ip_forward=1
sysctl -p
```

### Install Wireguard
<https://github.com/wgredlong/WireGuard/blob/master/2.%E7%94%A8%20wg-quick%20%E8%B0%83%E7%94%A8%20wg0.conf%20%E7%AE%A1%E7%90%86%20WireGuard.md>
```sh
apt install wireguard -y

# vim /etc/wireguard/wg0.conf
scp d:/Downloads/wg0.conf root@192.168.2.254:/etc/wireguard

wg-quick up wg0

# set start on boot  
systemctl enable wg-quick@wg0.service
```

### isc-dhcp-server
```bash
apt install isc-dhcp-server

vim /etc/default/isc-dhcp-server
# INTERFACESv4="eth0"

vim /etc/dhcp/dhcpd.conf
# subnet 192.168.2.0 netmask 255.255.255.0 {
#   range 192.168.2.111 192.168.2.222;
#   option routers 192.168.2.254;
# }

# host na-agent {
#   hardware ethernet 00:0c:29:3d:bf:41;
#   fixed-address 192.168.2.123;
# }

cat /var/lib/dhcp/dhcpd.leases # 檢查lease
```

### nftables
```bash
# https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes
systemctl enable nftables.service
nft flush 
nft list ruleset

nft list tables [<family>]
# The hooks for ip, ip6 and inet families are: prerouting, input, forward, output, postrouting.

# https://wiki.gentoo.org/wiki/Nftables/Examples
nft add rule inet filter forward iif eth0 oif eth1 drop
nft add rule inet filter forward iif wg0 oif eth1 drop

nft add rule inet filter forward iif eth0 oif eth1 ct state related,established accept
nft add rule inet filter forward iif wg0 oif eth1 ct state related,established accept

nft add rule inet filter input ct state related,established accept

# NAT
# https://sven.stormbind.net/blog/posts/misc_nftp_masquarading/
nft add table nat
nft 'add chain nat postrouting { type nat hook postrouting priority 100 ; }'
nft add rule nat postrouting ip saddr 192.168.2.0/24 oif eth0 masquerade

# Delete rules
# https://wiki.nftables.org/wiki-nftables/index.php/Simple_rule_management
nft -a list table nat # nft -a list table ip nat
# table ip nat { # handle 2
#         chain postrouting { # handle 1
#                 type nat hook postrouting priority srcnat; policy accept;
#                 masquerade # handle 4
#         }
# }
nft delete rule nat postrouting handle 4

# save config file
nft list ruleset > /etc/nftables.conf
# read config file
nft -f /etc/nftables.conf
# clear 
nft flush ruleset

# https://blog.csdn.net/wangcg123/article/details/108739367
```

### Routing
```bash
ip route add 10.113.0.0/16 dev wg0
ip route add 192.168.0.0/16 dev wg0

netstat -rt
```