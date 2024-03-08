## Apache2
---

[TOC]

### Basic

```bash
    apt-get install apache2

    # default site directory
    /var/www/html/ 

    # configurations
    /etc/apache2/apache2.conf
    /etc/apache2/ports.conf

    # conf
    /etc/apache2/conf-available
    /etc/apache2/conf-enabled

    # mods
    /etc/apache2/mods-available
    /etc/apache2/mods-enabled

    # sites
    /etc/apache2/sites-available
    /etc/apache2/sites-enabled

    a2(en/dis)(mod/conf/site)

    a2query -(m/s/c)

    apache2ctl configtest
```

### Configs

```bash
    # 如果已啟動模組alias_module，則....
    <IfModule alias_module>
        ...
    </IfModule>

    # 檔案【目錄】的權限控制、設定
    <Directory "目錄位置">
        # Order Allow,Deny
        # Order Deny,Allow 順序有差
        allow from all
        deny from 162.23.180.0/24 10.0 192.168.9 172
        ...
    </Directory>

    # 網址【路徑】的權限控制、設定
    <Location "路徑位置">
        ...
    </Location>

    # VirtualHost
    # *表示對所有IP，提供80埠號(http)的服務 !!是伺服器網卡的IP
    <VirtualHost *:80>
    DocumentRoot /var/www/html
    ServerName www.kazan.ru
    #ServerAlias ru.kazan.com (NOT Verified)
        <Location /~*/>
        allow from all
        deny from 162.23.180.0/24 10.0 192.168.9 172
        #或是 Require ip 192.168.0.0/24
        ...
        </Location>
    ...
    </VirtualHost>

    # 可以同時設定多個網站
    <VirtualHost *:80>
    DocumentRoot /root/web
    ServerName cert.kazan.ru
    ...
    </VirtualHost>
```

### Some Tricks

#### Redirect http to https
<https://www.tecmint.com/redirect-http-to-https-on-apache/>
```bash
    # in :80 vhost
    Redirect (permanent/301) / https://www.lab.sivs.org/ # () stands for permament redirect
```

### Modules

#### ssl

```bash
    a2enmod ssl 
    a2ensite default-ssl.conf

    # in sites config
    # SSL認證引擎
    SSLEngine on
    # SSL認證檔案
    SSLCertificateFile    /etc/ssl/certs/ssl-cert-snakeoil.pem 
    # SSL認證金鑰
    SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key # 如果用pem可以不用這行

    # require a certification
    openssl req -x509 -nodes -days 365 -keyout apache.key -out apache.crt
    # convert .pfx to .pem
    openssl pkcs12 -in cert.pfx -out cert.pem -nodes # length must >= 2048
```

#### ratelimit 

```bash 
    a2enmod ratelimit

    # in sites config 
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 256
```

#### userdir 

```bash
    a2enmod userdir
    mkdir /home/user/public_html
    chmod +r /home/user/public_html
    echo hello > /home/user/public_html/index.html
    vim /etc/apache2/mods-enabled/userdir.conf
```

#### alias

```bash
    a2enmod alias
    vim /etc/apache2/mods-available/alias.conf # or vhost conf
    #   Alias /test /testdir
    #    <directory /testdir>
    #            require all granted
    #    </directory>
    ln -s /testdir /var/www/html/test # the way better and easier
```

#### redirect
	<If "(%{HTTP_HOST} == 'www.csr.com') || (%{HTTP_HOST} == 'csr.com') || (%{HTTP_HOST} == 'csrnew.com')">
	Redirect permanent / http://www.csrnew.com/ # 301重定向
	</If>
	<ElseIf "!(%{HTTP_HOST} == 'www.csrnew.com') && !(%{HTTP_HOST} == 'localhost')">
	Require all denied
	</ElseIf>

	#   <IfModule mod_rewrite.c> 
	#       RewriteEngine on # 開啟rewrite引擎
	#       RewriteCond %{HTTP_HOST} ^www.csr.com$ [OR]   
	#       RewriteCond %{HTTP_HOST} ^csr.com$ [OR]     
	#       RewriteCond %{HTTP_HOST} ^csrnew.com$     
	#       RewriteRule ^/(.*)$ http://www.csrnew.com/$1 [R=301,L] 
	#   </IfModule>

#### enable .htaccess
for more usage:<https://www.linode.com/docs/web-servers/apache/how-to-set-up-htaccess-on-apache/#restrict-directory-listings>
	vim /etc/apache2/apache2.conf
	# AllowOverride all

#### enable auth 
<http://10.0.0.1:8080/en/howto/auth.html>
```bash
	#if use .htaccess 
	vim /etc/apache2/apache2.conf 
	# AllowOverride all

	htpasswd -c -b /htpasswd kraken Skills39 # create password file

	vim /etc/apache2/sites-enabled/000-default.conf
	#<Directory "/var/www/html">
	#       AuthType basic
	#       AuthName "test"
	#       AuthUserFile "/htpasswd"
	#       Require valid-user
	#</Directory>
```

#### access control 
<https://www.cnblogs.com/top5/archive/2009/09/22/1571709.html>
```bash
# Obsoleted
	Order Deny,Allow # 會決定最後使用哪一條規則
	Deny from all 
	Allow from 10.0.0 # 10.0.0.0/24
	
# new method for spec file/dir
	Require all denied
	Require ip 192.168.0.0/16

```

#### set a 404 page
```bash
	ErrorDocument 404 /404.html
	ErrorDocument 403 /403.html
```

#### disable page Index
```bash
	vim /etc/apache2/apache2.conf 
	# Options Indexes FollowSymLinks 刪掉Indexes
```

#### enable server-info/status
```bash
    a2enmod info
    vim /etc/apache2/mods-enabled/info.conf # Require ip 10.0.0.0/24
    vim /etc/apache2/mods-enabled/status.conf # Require ip 10.0.0.0/24
```

#### enable php
```bash
    apt install libapache2-mod-php
    a2enmod php7.3
```


Reference:
> [Apache: Restrict access to specific source IP inside virtual host](https://stackoverflow.com/questions/19711716/apache-restrict-access-to-specific-source-ip-inside-virtual-host/19845618)
> [Apache Module mod_ratelimit](https://httpd.apache.org/docs/2.4/mod/mod_ratelimit.html)
> [Per-user web directories](https://httpd.apache.org/docs/trunk/howto/public_html.html)
> [difference between \_default\_:* and *:* in VirtualHost Context](https://serverfault.com/questions/567320/difference-between-default-and-in-virtualhost-context)
>
>
>