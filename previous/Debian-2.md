[TOC]

### unsorted

```bash
    man samba
    info samba
    echo "hello" > greetings.txt
    echo "hello" >> greetings.txt
    apt-get install curlftpfs
    curlftpfs ftp://username:password@192.168.1.111 /mnt/ftp
    umount /mnt/smb
    ln -s 
    crontab
    mount -a

    eject
    eject -t
    
    ps -aux
    netstat -tlnup
    # Windows alternatives:
    # netstat -a | Select-String "Listening"
    # netstat -a | find /i "Listening"
    
    lsof -i -P -n
    service --status-all

    mount -t cifs -o username="Username",password="Password" //10.0.0.1/test /home/user/

    hostnamectl set-hostname Scharf

    fallocate -l ?M 檔案 # 製作特定大小檔案

    dd if=/dev/zero bs=1G of=/test count=1

    touch ~/.hushlogin

    #檢查群組

	getent group groupname
	vim /etc/group
	
	groups username
	id username

    tar -c 創建包 -x 釋放包 -v 顯示命令過程 -z 代表壓縮包 -f

    apt search network-manager

    systemctl is-active apache2 --quiet

    echo "error message" > /dev/null 2>&1

    for i in $(ls -d */); do echo $i; done

    nohup /path/my_program &

    date '+%D'
    
    last
    lastb

    apt install tree 
    apt install neofetcfh screenfetch

    cat /test | awk '{print $1}' 
    sed -a 's/test/test2/g' testfile

    update-alternatives --config editor

    getent group groupname
	vim /etc/group
	groups username
	id username

    test -e /dmtsai && echo "exist" || echo "Not exist"

    printenv
```






### MOTD

#### specific user motd
```bash
    mkdir /user-motd/

#   make a execution that will excute when user logged in
#   e.g. /etc/profile
    vim /etc/profile

#   if /user-motd/${USER} exists, then print /user-motd/${USER}
       test -f /user-motd/${USER} && cat /user-motd/${USER}
    
#
    echo "usermotd test for root" > /user-motd/root
```

#### specific ssh login motd
```bash
    mkdir /ssh-motd/

    vim /etc/ssh/sshrc 
    #   #! /bin/bash
    #   test -f /ssh-motd/${USER} && cat /ssh-motd/${USER}
    chmod a+x /etc/ssh/sshrc 
    
    echo "sshmotd test for root" > /ssh-motd/root
```

#### check whether shell is controlled by SSH 
Source: <https://unix.stackexchange.com/questions/9605/how-can-i-detect-if-the-shell-is-controlled-from-ssh>
```bash
#   be sure about that you've 
#   added double quotation mark!!
    test -n "$SSH_TTY" && echo 'ssh' || echo 'not ssh'
    test -n "$SSH_CONNECTION" && echo 'ssh' || echo 'not ssh'
```

### rc File

#### general rc file
- bash: `/etc/profile`
- ssh: `/etc/ssh/sshrc`

#### each user rc file
- bash: `~/.bashrc` or `~/.profile`
- ssh: `~/.ssh/rc`

#### logout rc
`~/.bash_logout`

### some dirs
```bash
    /etc/motd
    /run/motd.dynamic
    /etc/update-motd.d
    /etc/issue

    /etc/skel/
    /etc/login.defs
```

### Permissions

```bash
    umask 022
    chown nouser:nogroup ./test 
    chown -R root:root /tmp/tmp1

    chmod a/u/g/o +/-/= (r/w/x | u/g/o) {file/dir} 
    chmod {bin_Code} {file/dir}
    chmod [who] [+ | - | =] [mode] {file/dir}
    
    usermod

    # SUID 4XXX
    chmod u+s {路徑}
    # SGID 2XXX
    chmod g+s {路徑}
    # SBIT 1XXX
    chmod o+s {路徑}

```
chmod:<https://blog.csdn.net/shaobingj126/article/details/7031221>
suid,sgid,sbit: <https://kknews.cc/code/xex48g9.html>

### rsyslog

<https://www.cnblogs.com/hftian/p/6542454.html>

```bash
    logger -p local0.info "ni hao"
    logger -s "stderr test"

    vim /etc/rsyslog.conf
#   you can specify local0-7 as you want.
    # local1.*                /mylog
```

- debug     有調式資訊的,日誌資訊最多
- info      一般資訊的日誌,最常用
- notice    最具有重要性的普通條件的資訊
- warning   警告級別
- err       錯誤級別,阻止某個功能或者模組不能正常工作的資訊
- crit      嚴重級別,阻止整個系統或者整個軟體不能正常工作的資訊
- alert     需要立刻修改的資訊
- emerg     核心崩潰等嚴重資訊
- none      什麼都不記錄
---
- .xxx: 表示大於等於xxx級別的資訊
- .=xxx:表示等於xxx級別的資訊
- .!xxx:表示在xxx之外的等級的資訊
--- 
```bash
#   UDP 
    *.* @192.168.0.1        
#   TCP
    *.* @@192.168.0.1:10514
#   send to user(s)
    *.*   root,kraken,test
    *.*   * # all users
#   trigger 
    local3.*    ^/tmp/a.sh      # ^号后跟可执行脚本或程序的绝对路径
    # 日志内容可以作为脚本的第一个参数.

#   filter
    :msg, contains, "sshd"   /testlog
    :msg, regex, "fatal .* error" /testlog1 # using regex

#   example
    :msg,!contains, "sshd"  stop # discard not sshd log
    :msg, contains, "root"  /sshlog # filter logs with "root" string

```

#### rsyslog server & client setup
```bash
    vim /etc/rsyslog.conf

# server
    
#   uncomment the proto you want to use

    # provides UDP syslog reception
    module(load="imudp")
    input(type="imudp" port="514")

    # provides TCP syslog reception
    module(load="imtcp")
    input(type="imtcp" port="514")

    local1.*    /remote-log

    # specify host ip 
    :FROMHOST-IP, isequal, "192.168.0.161" /var/log/host161.log
    :FROMHOST-IP, startswith, "192.168.1." /var/log/network1.log

#   client

    local1.*    @10.0.0.200     # send local1.* to 10.0.0.200 via udp 
    local1.*    @@10.0.0.200    # via tcp
```

### logrotate
```bash
    vim /etc/logrotate.conf
#   example
    # /var/log/syslog
    # {
    #     rotate 7
    #     daily
    #     missingok
    #     notifempty
    #     delaycompress
    #     compress
    #     postrotate
    #             /usr/lib/rsyslog/rsyslog-rotate
    #     endscript
    # }
```

### data processing
<http://linux.vbird.org/linux_basic/0330regularex.php>
#### sed
<https://www.runoob.com/linux/linux-comm-sed.html>
#### grep 
#### awk 
<https://www.runoob.com/linux/linux-comm-awk.html>
#### regex
<https://transbiz.com.tw/regex-regular-expression-ga-%E6%AD%A3%E8%A6%8F%E8%A1%A8%E7%A4%BA%E5%BC%8F/>
