# วิธีติดตั้ง PHP หลาย Version ให้ใช้งานกับ NGINX บน Ubuntu 20.04
ความรู้ความเข้าใจพื้นฐาน
- สามารถใช้งาน Linux command line ได้
- สามารถตั้งค่า NGINX ขั้นพื้นฐานได้

## 1. ติดตั้ง NGINX
ทำการอัพเทเวอร์ชั่นของ Ubuntu และติดตั้ง NGINX ได้เลย
```bash
apt update -y
apt upgrade -y
apt -y install nginx
```
จากนั้นทำการตรวจสอบรายการ Application ใน Firewall ด้วย
```bash
ufw app list
```
<b>ผลลัพธ์</b>
```
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
ระบบจะแสดงรายการ application ต่างๆออกมา เราสามารถ firewall allow ด้วยวิธี
```bash
ufw allow 'Nginx Full'
ufw allow 'Nginx HTTP'
ufw allow 'Nginx HTTPS'
ufw allow 'OpenSSH'
```
อย่าลืม Allow OpenSSH ด้วยนะจ๊ะเดี่ยวจะ Remote ไม่ได้ถ้าเราเปิดใช้งาน Firewall ตรวจสอบว่า application ที่เราทำการ allow ไว้ด้วยคำสั่ง
```bash
ufw status
```
<b>ผลลัพธ์</b>
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx HTTP                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```
ถ้าระบบแสดงผลเป็น
```
Status: inactive
```
แสดงว่า firewall ของคุณยังไม่เปิดใช้งานให้คุณใช้คำสั่งต่อไปนี้เพื่อเปิดใช้งาน firewall
```bash
ufw enable
```
ถ้าเราไม่ทราบ หรือไม่มี Domain name ให้ใช้คำสั่ง
```bash
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
ระบบจะแสดงผลดังนี้
```
99.38.118.190
10.3.0.16
fe00::905a:faff:faa2:20c7
```
จากนั้นให้ทดสอบการทำงานของ NGINX เปิด URL: http://99.38.118.190

![NGINX Wellcome](https://storage.kaikannook.com/image/showimage/common/blog/be384df195c3258cb34e1010b2051faeb0.png)

ถ้าขึ้นหน้าต่างแบบนี้ถือว่าการติดตั้ง NGINX เสร็จสมบูรณ์

## 2. ติดตั้ง PHP หลาย version
เช่นเคยให้ทำการอัพเดท ubuntu กันก่อน
```bash
apt update -y
```
จากนั้นตั้งค่า Repository ของ PHP กันครับด้วยคำสั่ง
```bash
apt install software-properties-common
add-apt-repository ppa:ondrej/php
apt update -y
```
ทำการแก้ไขไฟล์
```
nano /etc/apt/sources.list.d/ondrej-ubuntu-php-mantic.sources
```
แก้ไขจาก Suites: mantic เป็น Suites: jammy

จากนั้นติดตั้ง PHP กันได้เลย เริ่มจาก 5.6 ถึง 7.4 กันเลย
```bash
apt update -y
apt install -y php5.6 php5.6-fpm
apt install -y php7.0 php7.0-fpm
apt install -y php7.1 php7.1-fpm
apt install -y php7.2 php7.2-fpm
apt install -y php7.4 php7.4-fpm
apt install -y php8.1 php8.1-fpm
apt install -y php8.2 php8.2-fpm
```
เมื่อติดตั้งเสร็จไฟล์ php socket จะรวมไว้ใน Folder `/var/run/php/`
```bash
ls /var/run/php/
```
ผลลัพธ์
```
total 8
-rw-r--r-- 1 root     root     4 Feb 17 16:50 php5.6-fpm.pid
srw-rw---- 1 www-data www-data 0 Feb 17 16:50 php5.6-fpm.sock
-rw-r--r-- 1 root     root     5 Feb 17 16:51 php7.4-fpm.pid
srw-rw---- 1 www-data www-data 0 Feb 17 16:51 php7.4-fpm.sock
```
### ติดตั้ง PHP Extension
กรณีต้องใช้ `extension` อื่นต้องใช้คำสั่งต่อไปนี้
```bash
apt install php5.6-mysql # mysql extension
apt install php7.x-mysql # mysql extension ของ php7.xxx version ต่างๆ

#Install basic extension.

# php5.6
apt -y install php5.6-mysql php5.6-curl php5.6-gd php5.6-intl \
  php-pear php-imagick php5.6-imap php5.6-mcrypt php5.6-memcache  \
  php5.6-pspell php5.6-recode php5.6-sqlite3 php5.6-tidy \
  php5.6-xmlrpc php5.6-xsl php5.6-mbstring php5.6-gettext php5.6-curl

# php7.1
apt -y install php7.1-mysql php7.1-bcmath \
  php7.1-gd libgd-dev php7.1-imagick php7.1-imap \
  php7.1-intl php7.1-json php7.1-mbstring php7.1-mcrypt \
  php7.1-opcache php7.1-common php7.1-phalcon3 php7.1-soap \
  php7.1-tidy php7.1-xml php7.1-xmlrpc php7.1-xsl php7.1-zip php7.1-curl
  
# php7.2
apt -y install php7.2-mysql php7.2-bcmath \
  php7.2-gd libgd-dev php7.2-imagick php7.2-imap \
  php7.2-intl php7.2-json php7.2-mbstring php7.2-mcrypt \
  php7.2-opcache php7.2-common php7.2-phalcon3 php7.2-soap \
  php7.2-tidy php7.2-xml php7.2-xmlrpc php7.2-xsl php7.2-zip php7.2-curl
  
# php7.4
apt -y install php7.4-mysql php7.4-bcmath \
  php7.4-gd libgd-dev php7.4-imagick php7.4-imap \
  php7.4-intl php7.4-json php7.4-mbstring php7.4-mcrypt \
  php7.4-opcache php7.4-common php7.4-soap \
  php7.4-tidy php7.4-xml php7.4-xmlrpc php7.4-xsl php7.4-zip php7.4-curl

# php8.1
apt -y install php8.1-mysql php8.1-bcmath \
  php8.1-gd libgd-dev php8.1-imagick php8.1-imap \
  php8.1-intl php8.1-mbstring php8.1-mcrypt \
  php8.1-opcache php8.1-common php8.1-soap \
  php8.1-tidy php8.1-xml php8.1-xmlrpc php8.1-xsl php8.1-zip php8.1-curl

# php8.2
apt -y install php8.2-mysql php8.2-bcmath \
  php8.1-gd libgd-dev php8.2-imagick php8.2-imap \
  php8.2-intl php8.2-mbstring php8.2-mcrypt \
  php8.2-opcache php8.2-common php8.2-soap \
  php8.2-tidy php8.2-xml php8.2-xmlrpc php8.2-xsl php8.2-zip php8.2-curl
```

## 3. ตั้งค่า NGINX ให้ใช้งานกับ PHP ได้แต่ละ Version
สร้าง folder เฉพาะของ website ที่คุณต้องการ
```bash
mkdir /var/www/your_domain
```
ทำการตั้งค่าสิทธิ์การเข้าถึงให้ folder ที่เราสร้างขึ้นมา
```bash
chown -R $USER:$USER /var/www/your_domain
```
สร้างไฟล์ตั้งค่าของ website ของคุณ
```bash
nano /etc/nginx/sites-available/your_domain.conf
```
ทำการตั้งค่า php socket ตาม version ที่คุณต้องการ โดยไฟล์ socket จะเก็บใน `/var/run/php/` สามารถตั้งค่าตามตัวอย่างด้านล่าง
```
# Fiile: /etc/nginx/sites-available/your_domain.conf

server {
  listen 80;
  server_name www.mywebsite.com subdomain.mywebsite.com;
  client_max_body_size 100M;

  location / {
    root /var/www/mywebsite;
    index index.php;
    try_files $uri $uri/ /index.php;

    location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
      fastcgi_param SCRIPT_FILENAME $document_root/index.php;
    }

    location ~ /\.ht {
      deny all;
    }
  }
}
```
ทำการสร้าง shortcut link ไว้ใน `/etc/nginx/sites-enabled/` ด้วยคำสั่ง
```bash
ln -s /etc/nginx/sites-available/your_domain.conf /etc/nginx/sites-enabled/
```
ตรวจสอบความถูกต้องของการตั้งค่า NGINX
```bash
nginx -t
```
ทำการ Restart service ของ NGINX
```bash
systemctl reload nginx
```

### ตรวจสอบ PHP Extension
สามารถใช้ `php --ri [ชื่อ Extension]` ตามตัวอย่าง
```bash
php --ri mcrypt
```
หรือ

ทดสอบการทำงานของ PHP ด้วย
```bash
nano /var/www/your_domain/info.php
```
ผลลัพธ์
```
## File: /var/www/your_domain/info.php

<?php
phpinfo();
```
ทดสอบด้วย URL: `http://99.38.118.190/info.php` ระบบจะแสดงผลดังต่อไปนี้

![PHP Info](https://storage.kaikannook.com/image/showimage/common/blog/ddcc10a672fce76c09218f6579882f47d.jpeg)


## ที่มาของแหล่งความรู้และข้อมูล
- [How to Run Multiple PHP Versions with Apache on Ubuntu 20.04 / 18.04 / 16.04](https://devanswers.co/run-multiple-php-versions-on-apache/)
- [How To Install Linux, Nginx, MySQL, PHP (LEMP stack) on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04)
- [How to Install PHP 8.3 on Ubuntu 23.10](https://wiki.crowncloud.net/?How_to_Install_PHP_8_3_on_Ubuntu_23_10)

