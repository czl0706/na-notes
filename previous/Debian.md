## Debian 套件指令
---

[TOC]

### Some useful commands
```bash
    find {location} -name {name}
    man {confi_name application_name}

    Ctrl - Alt - Fn
```
<kbd>Ctrl</kbd> - <kbd>Alt</kbd> - <kbd>F3</kbd> switch from gui to cli 
<kbd>Ctrl</kbd> - <kbd>Alt</kbd> - <kbd>F2</kbd> switch back
<kbd>Alt</kbd> - <kbd><-</kbd> switch to gui from cli 


### apt
```bash
    apt-cdrom add
    apt-cdrom ident

    apt-cache search samba
    apt install samba
    # remove
    apt remove --purge samba
    apt autoremove samba
    # same as
    apt purge --autoremove samba
```

### Set up interfaces
```bash
# set ifname 
    vim /etc/default/grub
    #   GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
    uptate-grub
    
# setup manually
    vim /etc/network/interfaces
    
    #   auto eth0
    #   iface eth0 inet static
    #       address 10.0.0.1
    #       netmask 255.255.255.0

# using nmtui
    apt install network-manager
    nmtui

    nm-connection-editor # desktop environment

# using ip  
    # https://blog.51cto.com/zkxfoo/1758528

    ip addr 

    ip addr show eth0

    ip addr {add/del} 10.0.0.1/24 dev eth0

    ip addr flush eth0

    ip link set eth0 {up/down}

    ip route show 

    ip route add 0.0.0.0/0 via 10.0.0.254

    ip route add 0.0.0.0/0 src 10.0.0.1 dev eth0

    ip route del 0.0.0.0/0

    ip addr help 

    ip link help bond 
```
#### rename interfaces
```bash
    vim /etc/udev/rules.d/70-persistent-net.rules
        # SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="b8:ac:6f:65:31:e5", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="wan0"
```
#### bonding
<http://linux-ip.net/html/ether-bonding.html>
```bash
# use ip     
    modprobe bonding 
    ip link add bond0 type bond mode 802.3ad
    ifenslave bond0 eth2 eth3
    ip addr add 192.168.100.33/24 dev bond0
    ip link set dev bond0 up
# or use nmtui
    nmtui # ...
```
[Tutorial for nmtui](https://blog.gtwang.org/linux/nmtui-centos-linux-network-manager-text-user-interface/)

### RAID
<http://linux.vbird.org/linux_basic/0420quota.php#mdadm_create>
<http://shinchuan1.blogspot.com/2013/11/software-raid-mdadm.html>
```bash
    apt install mdadm
    mdadm --create /dev/md0 --auto=yes --level={0 | 1 | 5} \
        --chunk=64k --raid-devices=N --spare-devices=N /dev/sda /dev/...

    mkfs.ext4 /dev/md0

    mdadm --manage /dev/md0 --remove /dev/sda7

    mdadm --detail /dev/md0 | grep UUID
        # UUID: 2256da5f:4870775e:cf2fe320:4dfabbc6
    vim /etc/mdadm.conf
        # ARRAY /dev/md0 UUID=2256da5f:4870775e:cf2fe320:4dfabbc6

    blkid /dev/md0
        # /dev/md0: UUID="494cb3e1-5659-4efc-873d-d0758baec523" TYPE="xfs"
    vim /etc/fstab
        # UUID=494cb3e1-5659-4efc-873d-d0758baec523  /srv/raid ext4 defaults 0 0
```

### vim
```bash
    apt install vim # 安裝vim
    vim /etc/vim/vimrc  # 設定檔
    set nu  # 設定行號
    set belloff=all
    update-alternatives --config editor # 設定預設編輯器
    editor # 呼叫預設編輯器

    # in editor:
    :x # save and exit
    :e {location} # explore that directory or file
    :split , :vsplit
    :n , :prev
    :close
```
<kbd>Ctrl</kbd> + <kbd>6(^)</kbd> : back to last buffer
<kbd>Shift</kbd> + <kbd>V</kbd> : visual mode

- <kbd>Shift</kbd> + <kbd>\<</kbd> or <kbd>\></kbd> shift blocks left or right

<kbd>Ctrl</kbd> + <kbd>W</kbd> <kbd>W</kbd> : Switch between two tabs

### Apache
```bash
    apt install apache2 # 安裝
    vim /etc/apache2/apache2.conf # 設定
```
#### install manual 
```bash
    apt install apache2-doc
    # in /usr/share/doc/apache2-doc
```
#### convert .pfx to .pem
<https://ssorc.tw/7142/>
```bash
    # .pem檔密碼和證書合一 -nodes免加密
    openssl pkcs12 -in cert.pfx -out cert.pem -nodes 
```
#### change permission to directory
```bash
    # DocumentRoot /web
    # <Directory /web>
    #         require all granted
    # </Directory>
```
#### allow directory listing
```bash
    # Alias "/file" "/file/"
    # <Directory /file>
    #         require all granted
    #         Options +Indexes +FollowSymLinks
    # </Directory>
```
#### Reverse Proxy
```bash
    vim /etc/apache2/sites-enabled/000-default.conf
    #   ProxyPass "/outside" "http://12.34.56.78/"
    a2enmod proxy proxy_http
```
#### Load Balance
```bash
    a2enmod proxy proxy_balancer proxy_http
    a2enmod lbmethod_byrequests # !! important!!

# conf file
    SSLProxyEngine on
    ProxyPass "/" "balancer://hotcluster/"
    <Proxy "balancer://hotcluster">
        BalancerMember "https://lnxsrv1_web.wsc2019.org.tw"
        # status=+H for hot-standby (does not perform load balancing)
        BalancerMember "https://winsrv2_web.wsc2019.org.tw" status=+H 
    </Proxy>
```
#### user auth
```bash
    # <Directory /file>
    #             require valid-user
    #             Options Indexes
    #             AuthType Basic
    #             AuthName "File Share Login"
    #             AuthUserFile "/.htpasswd"
    # </Directory>

    htpasswd -b -c /.htpasswd nsc Skills39
```

### bind9
#### Basic
```bash
    apt install bind9  
    vim /etc/bind/named.conf
    ## options {};
    #   zone "skills39.org"{
    #   file "/etc/bind/db.comp";
    #   type master;
    #   };
    #   zone "0.0.10.in-addr.arpa"{
    #   file "/etc/bind/db.comp";
    #   type master;
    #   };

    cd /etc/bind
    cp db.empty db.skills39.org
    vim db.skills39.org
    #   www     IN      A       10.0.0.1
    #   1       IN      PTR     www.skills39.org.
    
    service bind9 restart

# To verify:
    apt install dnsutils

    nslookup www.skills39.org 10.0.0.3

    dig kraken.skills39.org @10.0.0.3
    dig -x 10.0.0.1 @10.0.0.3

    host kraken.skills39.org 10.0.0.3
    host 10.0.0.1 10.0.0.3
```
#### 設定主副
```bash
    # master
    type master;
    allow-transfer{
        10.0.0.1;
    };
    #notify yes;
    #also-notify{
    #    10.0.0.1;
    #};

    # slave
    type slave;
    masters{
        10.0.0.3;
    };
```
#### forwarders
```bash
    type forward;
    #forward only;
    forwarders{ 10.0.0.1; };

    vim named.conf.options
        # dnssec-validation no;
```
#### conditional resolver
```bash
    vim /etc/bind/named.conf
    #
        include "/etc/bind/named.conf.options";
        #include "/etc/bind/named.conf.local";
        #include "/etc/bind/named.conf.default-zones";

        acl local{
                10.0.0.0/24;
        };
        acl external{
                any;
                !10.0.0.0/24;
        };
        view "local"{
                match-clients {
                        local;
                };
                zone "skills39.org"{
                        type master;
                        file "db.skills39.local";
                };
        };
        view "external"{
                match-clients {
                        external;
                };
                zone "skills39.org"{
                        type master;
                        file "db.skills39.external";
                };
        };
    #
```
#### for ipv6
```bash
    vim /etc/bind/named.conf

    # zone "skills39.org"{
    #     file "/etc/bind/db.skills39.org";
    #     type master;
    # };

    #   cafe::/112 arpa 

    # zone "0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.e.f.a.c.ip6.arpa"{
    #         file "/etc/bind/db.skills39.org";
    #         type master;
    # };

    vim /etc/bind/db.skills39.org
    # www     IN      AAAA    cafe::1
    # 1.0.0.0 IN      PTR     www.skills39.org.
```

### isc-dhcp-server
```bash
    apt install isc-dhcp-server

    vim /etc/default/isc-dhcp-server
    #   INTERFACESv4="eth0 eth1"

    vim /etc/dhcp/dhcpd.conf
    #   option domain-name "example.org";
    #   option domain-name-servers 10.0.0.1;

    #   subnet 10.0.0.0 netmask 255.255.255.0 {
    #   range 10.0.0.2 10.0.0.4;
    #   range 10.0.0.8 10.0.0.12;
    #   option domain-name-servers ns1.internal.example.org;
    #   option domain-name "internal.example.org";
    #   option routers 10.0.0.1;
    #   option broadcast-address 10.0.0.255;
    #   default-lease-time 600;
    #   max-lease-time 7200;
    #   }

    #   host cp2 {
    #   hardware ethernet 08:00:07:26:c0:a5;
    #   fixed-address 10.0.0.3;
    #   }

    #option nis-domain "domain.org"; 設定 NIS網域
    #option ntp-servers 192.168.1.1; 設定時間伺服器
    #option netbios-name-servers 192.168.1.1; 設定WINS伺服器
```
#### isc-dhcp-server for ipv6(failed, no radvd)
```bash
    vim /etc/default/isc-dhcp-server
    #   INTERFACESv6="eth0"  

    vim /etc/dhcp/dhcpd6.conf
    #   subnet6 cafe::/64 {
    #       range6 cafe::3 cafe::10;
    #   }

    service isc-dhcp-server restart

#   In Windows:
    #ipconfig /release6  
    #ipconfig /renew6
```
### Samba
<https://noob.tw/samba/>
```bash
    apt install samba

    man smb.conf
    
    vim /etc/samba/smb.conf
    
        [www]
        comment = www
        path = /var/www
        browseable = yes
        read only = no
        create mask = 777
        directory mask = 777
        vaild users = user, @users
        guest ok = no

        [web]
#       valid users = Administrator
        hosts allow = Administrator , 192.168.10.10
        read only = no
        path = /web
        veto files = /*.exe/*.dll/*.txt/


    pdbedit -a user
    echo -e "passwd\npasswd\n" | smbpasswd -a -s user

    service smbd restart

# mount samba filesystem
    apt install cifsutils
    mount.cifs -o username="Username",password="Password" //IP/share /mnt/smb

# fstab mount samba filesystem
    apt install cifsutils
    vim /etc/fstab
        # //10.0.0.2/web  /web    cifs    username=anon,password=anon     0       0

# in Windows
    net use
    net use * /del # clear credentials
```

### SSH
```bash
    man sshd_config
    # in config
    PermitRootLogin yes # 允許root登入 
    
    Match Address {IP1},{IP2} # Specific IP !! Append at the end of the config !!
        Permitrootlogin yes
    Match User {},{}

    AllowUsers  *@10.0.0.* *@192.0.0.0/24 <u2> <u3>
    AllowGroups 
    DenyUsers *
    DenyGroups 

    PubkeyAuthentication yes
    AuthorizedKeysFile %h/.ssh/authorized_keys
    PasswordAuthentication no
    
    Protocol 2 
```
#### using passphrase
```bash
    ssh-keygen (-t dsa)  # in Windows
    scp .\.ssh\id_rsa.pub root@10.0.0.1:~/.ssh/authorized_keys

    # or append it
    scp .\.ssh\id_rsa.pub root@10.0.0.1:~/
    ssh root@10.0.0.1
    mv id_rsa.pub ./.ssh/authorized_keys # cat id_rsa.pub >> ./.ssh/authorized_keys

    #!! better to chown/chgrp it !!
```

#### require both key and password
```bash
    vim /etc/ssh/sshd_config
        # AuthenticationMethods password,publickey
        # #PasswordAuthentication no
        # #PubkeyAuthentication yes
```
#### SSH tunnel  
```bash
    ssh kraken@163.23.180.232 -p 2222 -L 3380:localhost:3389 # 163.23.180.232:3389 mapped thru localhost:3380
```
#### AutoSSH
```bash
    apt install autossh
```
#### sftp
<http://blog.jzscs.com/2015/01/debian-sftp.html>
```bash
# sftp is already enabled by default, the setting below shows how to chroot to specific directory
    Subsystem sftp ... internal-sftp
    Match User root
        ChrootDirectory /root
        Forcecommand internal-sftp
```
#### Client Config
```bash
    # ~/.ssh/config
    Host *.skills39.org
        User Kraken 
        IdentityFile ~/.ssh/id_rsa1
```

### User Management
```bash
    /etc/default/useradd 
        # SHELL=/bin/bash
    /etc/skel # 預設家目錄內的東西

    # 設定密碼
    echo -e "password\npassword\n" | passwd user
    echo "user:password" | chpasswd

    apt install whois
    useradd user -m -p `mkpasswd password`

# better version:
    #! /bin/bash
    for i in `seq -f "mgt%02g" 1 60`; do
            echo -e "Skills39\nSkills39" | adduser -q $i
            adduser -q $i mgts
    done
# prev version 
    for i in {1..9}
    do
        useradd user00$i -m -g employees -p $(mkpasswd Skills39)
        passwd -e user00$i # password must be changed next login
        # or chage --lastday 0 user00$i
        echo -e "Skills39\nSkills39\n" | smbpasswd -a -s user00$i
    done
```
#### PAM
```bash
# https://cloud.tencent.com/developer/news/265554
# http://cn.linux.vbird.org/linux_basic/0410accountmanager_5.php
    
    vim /etc/pam.d/common-password
        # password.... {minlength/minlen}=8

    vim /etc/login.defs
        # PASS_MIN_LEN    8

    vim /etc/pam.d/gdm-password
```
#### ulimit
```bash
    vim /etc/security/limits.conf
    ulimit -a 
```

#### Usermanagement with python

`vim a.py`
```py
    import os
    for i in range(1,101):
        os.system('useradd user{0:0>3d} -m -p $(mkpasswd pw{0:0>3d})'.format(i))
        #os.system(f'useradd user{i:0>3d} -m -p $(mkpasswd pw{i:0>3d})')
```

### vsftpd 
<https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-debian-10>
<http://linux.vbird.org/linux_server/0410vsftpd.php#server_vsftpd.conf>
```bash
    apt install vsftpd

    vim /etc/ftpusers # specify disallowed users
        #root

    man vsftpd.conf

    vim /etc/vsftpd.conf
        
        # basic permissions...
        local_enable=YES 
        write_enable=YES
        
        local_umask=022

        # disallow chroot
        chroot_local_user=YES
        allow_writeable_chroot=YES # !! or just chmod a-w your homedir

        #chroot_list_enable=YES
        # (default follows)
        #chroot_list_file=/etc/vsftpd.chroot_list

        # spec login directory
        user_sub_token=$USER
        local_root=/home/$USER/ftp

        # spec allowed/disallowed user
        userlist_enable=YES
        userlist_file=/etc/vsftpd.user_list
        userlist_deny=(NO/YES)

        # enable TLS verification
        rsa_cert_file=
        rsa_private_key_file=
        ssl_enable=YES
        # ssl_tlsv1=YES
        # ssl_sslv2=NO
        # ssl_sslv3=NO
        
    
    vim /etc/shells
        /usr/sbin/nologin
    usermod kraken -s /usr/sbin/nologin
```
#### for anonymous init
```bash
    anonymous_enable=YES
    anon_root=/var/ftp
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES

    chown ftp /var/ftp/test/ # or chmod 777
``` 
#### Specify per user_root
<http://howto.gumph.org/content/setup-virtual-users-and-directories-in-vsftpd/>
```bash
    user_config_dir=/user_confdir
    # vim /user_confdir/user
        local_root=/test
```

### proftpd
```bash
    apt install proftpd*
```

### TCP Wrappers
```bash
    /etc/hosts.{allow/deny}
    sshd : 192.168.0.0/24, 127.0.0.1, [::1]
    sshd : ALL
```

### crontab
```bash
    crontab -e 
        * * * * * {command} # 分 時 日 月 週
        @reboot        #Run once, at startup.
        @yearly        #Run once a year, "0 0 1 1 *".
        @monthly       #Run once a month, "0 0 1 * *".
        @weekly        #Run once a week, "0 0 * * 0".
        @daily         #Run once a day, "0 0 * * *".
        @hourly        #Run once an hour, "0 * * * *".
```

### nginx
```bash
    apt install nginx
```
#### user auth
```bash
    apt install apache2-utils
    htpasswd -b -c /.htpasswd kraken Skills39 

    vim /etc/nginx/sites-enabled/default
        location / {
            # satisfy all; # when use both IP and user Restriction
            auth_basic "test";
            auth_basic_user_file /.htpasswd;  
            location /public/ {
                auth_basic off;
            }
        }
```
#### IP restriction
```bash    
        location / {
            # satisfy all; # when use both IP and user Restriction            
            allow 192.168.0.0/24;
            deny all;
        }
```
#### VirtualHost(server)
```bash
    server {
        listen 8004;
    #   listen [::]:80;
    #   server_name example.com;

        root /var/www/8004;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
```
#### Load Balancing
```bash
    upstream myserver {
        server 192.168.0.1:8001 weight=1;
        server 192.168.0.1:8002 weight=1;
        server 192.168.0.1:8003 weight=1;
        server 192.168.0.1:8004 weight=1;
        server 192.168.0.10:80 backup;
    }
    location / {
        proxy_pass http://myserver;
        # proxy_redirect http://myserver / ;
    }
```
#### ssl 
```bash
    server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
        include snippets/snakeoil.conf;
    }
    
    vim /etc/nginx/snippets/snakeoil.conf
        ssl_certificate /cert/cert.pem;
        ssl_certificate_key /cert/cert.pem;
```
#### redirect http to https 
<https://linuxconfig.org/how-to-use-nginx-to-redirect-all-traffic-from-http-to-https>
```bash
    server {
        listen 80;
        return 301 https://example.com$request_uri; # 301: permament 
        # or return 301 https://$host$request_uri; for global settings
        # ...
    }

    server {
        listen 443;
        # ...
    }
```
#### enable php
<https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04#step-3-%E2%80%93-installing-php>
```bash
    apt install php7.3-fpm php7.3-mysql # or just php7.3*

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
    }
```
#### home directory
https://www.server-world.info/en/note?os=Debian_10&p=nginx&f=3
```bash
    location ~ ^/~(.+?)(/.*)$ {
            alias /home/$1/public_html$2;
            index  index.html index.htm;
            autoindex on;
        }

    # ~ ^/~(.+?)(/.*)$
    # the first ~ = regex
    # ^$ require full match
    # (.+?)(/.*) matches all
    # store at $1 before second / 
    # then store the rest at $2

    # example: 
    # alias: /~test/dir123/ 
    # /home/test/public_html/dir123
```

### dnsmasq(tiny dns and dhcp server)
<https://www.cnblogs.com/sunsky303/p/9238669.html>
```bash 
    apt install dnsmasq
    vim /etc/hosts # vim /etc/banner_add_hosts
    man dnsmasq

    # host-record=<name>[,<name>....],[<IPv4-address>],[<IPv6-address>][,<TTL>]
    # ptr-record=<name>[,<target>]
    # address=/ad.youku.com/127.0.0.1
    # server=/google.com/8.8.8.8
    
    # host-record=www.skills39.org.,10.0.1.1
    # ptr-record=1.1.0.10.in-addr.arpa,www.skills39.org
    # address=/srv1.skills39.org/10.0.1.100
```
#### dhcp
```bash
    vim /etc/dnsmasq.conf
        domain=skills39.org
        dhcp-range=192.168.0.50,192.168.0.150,255.255.255.0,12h
        dhcp-option=option:dns-server,10.0.0.1
        dhcp-option=option:router,10.0.0.1

        # /dhcp-option
        # no-hosts
        # addn-hosts=/etc/banner_add_hosts
```
#### dhcpv6
```bash
    vim /etc/dnsmasq.conf
# SLAAC
        dhcp-range=cafe::10, cafe::20, slaac
# Stateless 
        dhcp-range=cafe::, ra-stateless
        dhcp-option=option6:dns-server,[cafe::1]
        #enable-ra
# Stateful
        #dhcp-host=id:00:03:01:00:08:00:27:5c:b9:f0, [cafe::40]
        dhcp-range=cafe::10, cafe::20, ra-names
        enable-ra
```

### install desktop env
```bash
    apt install tasksel
    tasksel
```

### Extract/Zip files
```bash
# tar
    tar -c -f -x -v -z -j -t # create file extract verbose gzip bzip2 list
    tar -cvf arc.tar *
    tar --delete -f
    tar -r -k 

    cd /log
    tar -cvf /backup/test.tar --exclude="*.tar" *

    tar -tf test.tar
    tar -xf test.tar --directory=/restore

# with encryption
    tar -cz your_dir | gpg -c -o your_archive.tgz.gpg
    gpg -d your_archive.tgz.gpg | tar xz

# zip

    apt install zip 
    zip -r myfiles *.log -x "not-necessary-log.log" 
    unzip myfiles.zip -d mydir

    # https://itsfoss.com/password-protect-zip-file/
    zip encrypted.zip * -x "*.zip" -e # -e for encryption

# gzip  
    gzip data.txt
    gzip -d data.txt.gz
```

### time management
```bash
    date;hwclock;timedatectl;ntpdate
```

### NTP 
```bash
    apt install ntp
    vim /etc/ntp.conf
    ###
    apt install ntpdate
    ntpdate 10.0.0.1
```
#### NTP sync time
<https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-debian-10>
<https://blog.gtwang.org/linux/linux-ntp-installation-and-configuration-tutorial/>
```bash
    # NTPd has time synchronization already!! 

# use systemd
    systemctl enable systemd-timesyncd
    vim /etc/systemd/timesyncd.conf
        # NTP=12.34.56.78
    systemctl restart systemd-timesyncd
# user ntpdate
    apt install ntpdate
    crontab -e  
    # @reboot ntpdate 12.34.56.78
```

### install packages
```bash
    dpkg -i httpd.deb
    
    tar -xvf httpd.jar
    apt install build-essential pkg-config intltool

    ./configure 
    make
    make install 
```

### sudo 
```bash
    apt install sudo
    visudo 
    #%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
    usermod user -aG sudo # grant sudo permission
```

### disk quota
<https://www.digitalocean.com/community/tutorials/how-to-set-filesystem-quotas-on-debian-10>
```bash
    apt install quota

    vim /etc/fstab
        # errors=remount-ro,usrquota,grpquota
    
    # mount a directory
    # /dev/sda1    /home        ext4        defaults,usrquota   0      0
    

    mount -o remount / # remount

    # enable quota
    quotacheck -ugmc /home

    # or these command:
        # quotacheck -ugm /
        # quotaon /
    

    #update-alternatives --config editor
    edquota -u user
    # Disk quotas for user user (uid 1000):
    # Filesystem                   blocks       soft       hard     inodes     soft     hard
    # /dev/vda1                        24       100M       110M          7        0        0

    setquota -u user 100M 110M 0 0 / # another way to set up quotas

    quota -s user # check

    setquota -t 864000 864000 / # set grace time (seconds)
```

### ipython
```bash
    apt install ipython3

    ipython3
    #
    #     In [1]: print('save file test')
    #     save file test

    #     In [2]: %save test.py ~0/
    #     The following commands were written to file `test.py`:
    #     print('save file test')
    #     get_ipython().magic('save test.py ~0/')
```

#### script example
```python
    #!/usr/bin/python3
    # coding: utf-8
    import requests, os
    try:
        requests.get('http://localhost')
        print('successfully connected to website!')
    except:
        print('failed to connect to website!')
        print('restarting nginx service...')
        os.system('systemctl restart nginx')
```
```bash
    crontab -e 
    #   */1 * * * * /statuscheck.py  >> /crontest
```

```python
    # https://medium.com/@sean22492249/introduction-e72bea2fc9a2
    # https://www.runoob.com/python/python-func-open.html
    # https://www.runoob.com/python/os-file-methods.html

    with open('/test','a+') as a:
        a.write()

    #file:路徑
    #mode:'r','w','a'
    #r:讀取模式(預設)
    #w:寫入模式(清除原本內容)
    #a:附加模式(附加在檔尾)

    with open('/test','r+') as a:	
        a.readline()
        a.readlines()
        for line in a: print(line)

    import os, shutil

    os.? # show manual in ipython interpreter
    os.getcwd()
    os.chdir()
    os.rename()
    os.listdir()
    os.system()
    os.path.isdir()
    os.path.isfile()
    os.rename()
    os.walk()

    shutil.copytree('./music', './new_dir')
    shutil.rmtree('./music')

    for file in os.listdir(os.getcwd()):
        if not os.path.isdir(file):
            print(file)
```

### mail server

#### smtp
```bash
    # Add DNS Record
    # @       IN      MX 10   mail
    # mail    IN      A       10.0.0.1
    apt install postfix #sasl2-bin
    dpkg-reconfigure postfix # reconfigure

    vim /usr/share/postfix/main.cf.dist
    vim /etc/postfix/main.cf
        # mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.10.0/23
        # smtpd_tls_cert_file=/cert.pem
        # smtpd_tls_key_file=/cert.pem

    apt install mailutils
    echo "context" | mail -s "subject" root

    apt install postfix-doc
    # /usr/share/doc/postfix/html
```

#### pop3 imap
```bash 
    apt install dovecot-pop3d dovecot-imapd
    
    vim /etc/dovecot/conf.d/10-ssl.conf
        # ssl_cert = </cert.pem
        # ssl_key = </cert.pem
    vim /etc/dovecot/conf.d/10-master.conf
        # ssl=yes
    vim /etc/dovecot/conf.d/10-openssl.conf
```

### format & partition & mount
<https://blog.gtwang.org/linux/linux-add-format-mount-harddisk/>
```bash
    fdisk -l /dev/sdb # check

    fdisk /dev/sdb
        n
        ...
        +10G

        w
    
    mkfs.ext4 /dev/sdb1

    blkid # obtain UUID

    vim /etc/fstab
    #UUID=1832b1b1-8ce3-44fb-bb5b-fe3fa8b427f9 /data         ext4    errors=remount-ro,usrquota,grpquota 0       1

    mkdir /data
    mount /data

    df -h # check
```
### Parted
<https://blog.gtwang.org/linux/parted-command-to-create-resize-rescue-linux-disk-partitions/2/>
```bash
    apt install parted

    parted /dev/sdb

        mklabel gpt
        mkpart

        resizepart

    mkfs.ext4 /dev/sdb1

```

### remote desktop connection 
```bash
    apt install xfce4 xrdp
```

### install NFS Server
<https://linuxconfig.org/how-to-set-up-a-nfs-server-on-debian-10-buster>
<http://linux.vbird.org/linux_server/0330nfs.php#nfsserver_need>

Port: 111(sunrpc) 2049 tcp/udp

```bash
    apt install nfs-kernel-server
    vim /etc/exports
        # /nfs  10.0.0.0/24(rw,sync,no_subtree_check)
        # /nfs  *(rw,sync,no_subtree_check)
    man exports
```

### Mount NFS
```bash
# debian 
    apt install nfs-common
    mount.nfs {ip}:/{vol} {dir}

    vim /etc/fstab
        # {ip}:/{vol} {dir} nfs defaults,user 0 0
# windows
    mount 10.0.0.1:/test1 v: 
```

### wsl
<https://docs.microsoft.com/zh-tw/windows/wsl/install-on-server>
```powershell
    # first step: enable wsl feature!
# install on windows 10 
    # just double click the .appx package
# install on windows server 
    Rename-Item .\DebianGNULinux_1-1-3-0_x64__76v4gfsz19hv4.Appx .\Debian.zip # convert to zip
    Expand-Archive .\Debian.zip .\Debian # extract 


    #wslconfig.exe /l
    ./debian config --default-user root
    ./debian 

# start a daemon outside the bash
    wsl -u root service ssh start # if debian can be invoked by wsl.exe
    .\debian.exe run service ssh start


    # \\wsl$\Debian for wsl2
    # C:\Users\Kraken\AppData\Local\Packages\TheDebianProject.DebianGNULinux*\LocalState\rootfs
```
#### add cdrom sources
<https://www.howtogeek.com/331053/how-to-mount-removable-drives-and-network-locations-in-the-windows-subsystem-for-linux/>
<https://superuser.com/questions/1331936/how-can-i-get-past-a-repository-is-not-signed-message-when-attempting-to-upgr>
```bash
    mkdir /cdrom1
    mkdir /cdrom2
    mkdir /cdrom3

    mount -t drvfs E: /cdrom1
    mount -t drvfs F: /cdrom2
    mount -t drvfs G: /cdrom3

    nano /etc/apt/sources.list
    #     #deb http://deb.debian.org/debian stretch main
    #     #deb http://deb.debian.org/debian stretch-updates main
    #     #deb http://security.debian.org/debian-security/ stretch/updates main
    #     deb [trusted=yes] file:/cdrom1/ buster contrib main
    #     deb [trusted=yes] file:/cdrom2/ buster contrib main
    #     deb [trusted=yes] file:/cdrom3/ buster contrib main

    apt update

    # if you want to install services, you should open port(s) via Windows Firewall!
```

### join ad domain 
<https://www.server-world.info/en/note?os=Debian_10&p=realmd>
```bash
    apt -y install realmd sssd* adcli krb5-user packagekit samba-common

    realm join skills39.org

# disable FQDN
    vim /etc/sssd/sssd.conf
        # use_fully_qualified_names = False

# create homedir at init login
    vim /etc/pam.d/common-session
        # session optional      pam_mkhomedir.so skel=/etc/skel umask=077
```

### ansible 
<https://www.cnblogs.com/kevingrace/p/10601309.html>
<https://medium.com/@chihsuan/ansible-%E8%87%AA%E5%8B%95%E5%8C%96%E9%83%A8%E7%BD%B2%E5%B7%A5%E5%85%B7-b2e8b8534a8d>
```bash
    apt install ansible

    vim /etc/ansible/hosts
    # [servers]
    # server1 ansible_host=203.0.113.111
    # server2 ansible_host=203.0.113.112
    # server3 ansible_host=203.0.113.113
    # 172.16.60.220 ansible_ssh_user=root ansible_ssh_pass=123456 ansible_ssh_port=22

    ansible all -m ping -u root

    ansible all -a "apt -y install apache2" -u root

    vim restart-apache2.yml
    # - hosts: all
    #   tasks:
    #   - name: restart apache2
    #     shell: service apache2 restart

    ansible-playbook restart-apache2.yml
```

### systemd
<https://www.itread01.com/content/1549616778.html>
<https://blog.longwin.com.tw/2018/02/linux-systemd-auto-start-daemon-service-2018/>
```bash
    vim /etc/systemd/system/example.service
    
    #    [Unit]
    #    Description=example daemon

    #    [Service]
    #    Type=simple
    #    WorkingDirectory=/usr/local/
    #    ExecStart=/usr/local/bin/example
    #    Restart=always

    #    [Install]
    #    WantedBy=multi-user.target
```

#### limit max tty
```bash
    vim /etc/systemd/logind.conf
        # NAutoVTs=
        # ReserveVT=
```

### shell scripting 
```bash
    sh -n myscript.sh # test 
    sh -x myscript.sh # debugging 

# case
    case $1 in 
        hello)
            echo hello!
            ;;
        date | dt)
            echo `date "%Y-%m-%d"`
            ;;
        *)
            echo "usage: $0 {hello | date(dt) | help}"
            ;;
    esac
# select menu
    PS3="select your favorite game:"
    select i in csgo lol overwatch
    do
        echo "Your favorite game: $i"
        break
    done
# while 
    i=1
    while [ $i -le 5 ];do
        echo $i
        ((i++))
    done

    while read i;do
        echo $i
    done
# for 
    for i in `ls -d /*/`;do
        echo $i
        for j in `ls -d $i*`;do
            echo -e "\t$j"
        done
    done
# if
    if [ -f /motd/$USER ]; then
        cat /motd/$USER
    fi

# 
    echo "You've sent $# arguments!"
    for i in $@; do 
        echo $i
    done

# if condition
# https://www.runoob.com/w3cnote/shell-summary-brackets.html
```
### ACL
```bash
    apt install acl

    setfacl -h
    setfacl -b file # remove all properities
    setfacl -k file # remove default properities
    
    setfacl -R -m u:user:rx directory # Recursively set permissions

    setfacl -d -m u:user:rx directory # set default ACL
    setfacl -m d:u:myuser1:rx /srv/projecta # set defualt ACL

    getfacl file
```
### Apparmor
<https://wiki.debian.org/AppArmor/HowToUse>
```bash
    aa-complain
    aa-status
```


### rsync
```bash
    apt install rsync 
    rsync -r /test/ root@10.0.0.2:/syncdir
```
### iSCSI
<https://blog.haostudio.net/hwp/%E5%9C%A8debian-%E4%B8%AD%E5%AE%89%E8%A3%9Discsi-initiator/>
<https://www.howtoing.com/setup-iscsi-target-and-initiator-on-debian-9>
<https://www.server-world.info/en/note?os=Debian_10&p=iscsi&f=2>
<https://www.server-world.info/en/note?os=Debian_10&p=iscsi&f=3>
```bash
# as a initiator
    apt install open-iscsi
    vim /etc/iscsi/initiatorname.iscsi
    vim /etc/iscsi/iscsid.conf
    iscsiadm -h
```
### Anti-Virus(clamAV)
<https://www.server-world.info/en/note?os=Debian_10&p=clamav>
```bash
    apt install clamav
```
### arp proxy
<https://wiki.debian.org/BridgeNetworkConnectionsProxyArp>
<https://ixnfo.com/en/how-to-enable-or-disable-proxy-arp-on-linux.html>
```bash
    cat /proc/sys/net/ipv4/conf/all/proxy_arp
    vim /etc/sysctl.conf
        # net.ipv4.conf.all.proxy_arp=1
```
### radius server
```bash
    apt install freeradius # has no installation candidate
    vim /etc/freeradius.conf
```
### squid
<http://www.study-area.org/linux/servers/linux_nat.htm>
```bash
    apt install squid
    vim /etc/squid.conf
        # /acl
        # /http_access

        # # 在第 1163 行下面增加一行﹕
        # acl siyongc src 192.168.100.0/255.255.255.0 # 請修改為您自己的實際設定
        # # 然後在第 1197 行下面再增加一行﹕
        # http_access allow siyongc	# 請修改為上一行定義的 acl 名稱
```
