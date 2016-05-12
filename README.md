#### uddepsecuvpsubu14
#####2
allow sudo
```
cd /etc
vim group
```
edit:add username to group
```
sudo:x:27:test
```
#####03 configuring SSH
```
cd /ssh
vim sshd_config   (not ssh_config)
```
edit
```
Port 22->2501   (disable common port)
PermitRootLogin no
AuthorizeKeyFile .ssh/id_rsa.pub
PasswordAuthentication no
```
Add
```
AllowUsers test
```
then
```
service ssh restart
```
#####config firewall
```
wget https://download.configserver.com/csf.tgz
```
```
tar zxvf csf.tgz
cd csf 
sh install.sh
```
then test
```
perl /usr/local/csf/bin/csftest.pl
```
conf:
```
cd /etc/csf
vim csf.conf
```
set
```
TCP_IN="80,443,2501"
UDP_IN="53"
UDP_OUT="53,113,123"
```
check
```
service csf status/restart
```
cd /etc/csf
vi csf.pignore
```
edit
```
```
then
```
service lfd restart
#####4
######install helpful tools
```
apt-get install landscape-common
```
```
apt-get install alpine
apt-get install postfix
```

######create swap space
```
sudo dd if=/dev/zero of=/swapfile count=1024 bs=1M
```
```
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
cd /etc
vi fstab
```
edit
```
/swapfile none swap sw 0 0
```
then
```
sudo swapon -s
sudo sysctl vm.swapiness=10 //temp change
```
to make a perm change:
```
vi /etc/sysctl.conf
```
edit(add)
```
vm.swapiness=10
```
#####nginx php
```
apt-cache search nginx
apt-get install nginx
```
```
apt-get install rcconf
```
to configure:
```
press spacebar (toggle on/off)
tab ->quit
```
install php5.6
```
add-apt-repository ppa:ondrej/php5
apt-get update
apt-get install php5 php5-fpm
```

######setting up PHP for nginx
```
cd /etc/php5/fpm
vim php.ini
```
edit
```
cgi.fix_pathinfo=0 (uncomment)
```
restart
```
service php5-fpm restart
```
edit nginx
```
cd /etc/nginx/sites-available
cp default default.dist
vim default
```
edit
```
server{
  server_name y.k.me;
}

LOCATION ~ \.PHP${
  try_files $uri =404;
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  fastcgi_pass unix:/var/run/php5-fpm.sock;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  include fastcgi_params;
}
```

######virtual hosts in nginx
```
cd /etc/nginx/sites-available
vim new.conf
```
edit
```
server{
  listen 80;
  server_name site.com www.site.com;
  root /home/path;
  index index.php index.html index.htm;
  access_log /;
  error_log /;
  location /{
    try_files $uri $uri/ /index.php?$args;
  }
  gzip on;
  gzip_disable "msie6";
  
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 3;
  gzip_buffers 16 8k;
  gzip_http_version 1.1;
  gzip_types text/plain text/css application/json;
  
  location ~* \.(jpg|png)$ {
    expires 365d;
  }
  
  error_page 403 =404;
  
  location = /robots.txt {access_log off; log_not_found off;}
  ```
  
  create a soft link in sites-enabled
  ```
  cd .. && cd sites-enabled
  ln -s /etc/nginx/sites-available/new.conf .
  ```

######
copy ssl.key to local and decrypt
```
openssl rsa -in ssl.key -out ssl.key
```
######install startSSL certificate
go to toolbox->retrieve certificate create crt file,copy  
wget pem
combine these two fileswget
```
cat crt pem >pem
```

redirect
```
return 301 http://link
```
ssl config:
```
server{
  listen 443 default;
  listen[::]:443 default ipv6only=on;
  ssl on;
  ssl_certificate /home/httpd/...pem;
  ssl_certificate /home/...ssl.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  add_header Strict-Transpost-Seciruty "max-age=30000000";
```

######Testing our SSL
use ssllabs.com
#####8
######Regular maintanance
```
cd /etc/apt/apt.conf.d
vim 20auto-upgrades
```
edit
```
APT::Perodic::Unattended_updtade "0";
```
edit csf.conf
```
AUTO_UPDATES ="0"
```
```
