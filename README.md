# วิธีติดตั้ง PHP หลาย Version ให้ใช้งานกับ NGINX บน Ubuntu 20.04
ความรู้ความเข้าใจพื้นฐาน
- สามารถใช้งาน Linux command line ได้
- สามารถตั้งค่า NGINX ขั้นพื้นฐานได้

## 1. ติดตั้ง NGINX
ทำการอัพเทเวอร์ชั่นของ Ubuntu และติดตั้ง NGINX ได้เลย
```bash
sudo apt update
sudo apt install nginx
```
จากนั้นทำการตรวจสอบรายการ Application ใน Firewall ด้วย
```bash
sudo ufw app list
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
sudo ufw allow 'Nginx Full'
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
sudo ufw allow 'OpenSSH'
```
อย่าลืม Allow OpenSSH ด้วยนะจ๊ะเดี่ยวจะ Remote ไม่ได้ถ้าเราเปิดใช้งาน Firewall ตรวจสอบว่า application ที่เราทำการ allow ไว้ด้วยคำสั่ง
```bash
sudo ufw status
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
sudo ufw enable
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
sudo apt update
```
จากนั้นตั้งค่า Repository ของ PHP กันครับด้วยคำสั่ง
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php && sudo apt update
```
จากนั้นติดตั้ง PHP กันได้เลย เริ่มจาก 5.6 ถึง 7.4 กันเลย
```bash
sudo apt install -y php5.6 php5.6-fpm
sudo apt install -y php7.0 php7.0-fpm
sudo apt install -y php7.1 php7.1-fpm
sudo apt install -y php7.2 php7.2-fpm
sudo apt install -y php7.4 php7.4-fpm
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
sudo apt install php5.6-mysql # mysql extension
sudo apt install php7.x-mysql # mysql extension ของ php7.xxx version ต่างๆ

#Install basic extension.

# php5.6
sudo apt -y install php5.6-mysql php5.6-curl php5.6-gd php5.6-intl \
  php-pear php-imagick php5.6-imap php5.6-mcrypt php5.6-memcache  \
  php5.6-pspell php5.6-recode php5.6-sqlite3 php5.6-tidy \
  php5.6-xmlrpc php5.6-xsl php5.6-mbstring php5.6-gettext

# php7.1
sudo apt -y install php7.1-mysql php7.1-bcmath \
  php7.1-gd libgd-dev php7.1-imagick php7.1-imap \
  php7.1-intl php7.1-json php7.1-mbstring php7.1-mcrypt \
  php7.1-opcache php7.1-common php7.1-phalcon3 php7.1-soap \
  php7.1-tidy php7.1-xml php7.1-xmlrpc php7.1-xsl php7.1-zip
```

## 3. ตั้งค่า NGINX ให้ใช้งานกับ PHP ได้แต่ละ Version
สร้าง folder เฉพาะของ website ที่คุณต้องการ
```bash
sudo mkdir /var/www/your_domain
```
ทำการตั้งค่าสิทธิ์การเข้าถึงให้ folder ที่เราสร้างขึ้นมา
```bash
sudo chown -R $USER:$USER /var/www/your_domain
```
สร้างไฟล์ตั้งค่าของ website ของคุณ
```bash
sudo nano /etc/nginx/sites-available/your_domain.conf
```
ทำการตั้งค่า php socket ตาม version ที่คุณต้องการ โดยไฟล์ socket จะเก็บใน `/var/run/php/` สามารถตั้งค่าตามตัวอย่างด้านล่าง
```
# Fiile: /etc/nginx/sites-available/your_domain.conf

server {
    listen 80;
    server_name your_domain www.your_domain;
    root /var/www/your_domain;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock; # ตั้งค่า php socket ตาม version ที่คุณต้องการ
     }

    location ~ /\.ht {
        deny all;
    }

}
```
ทำการสร้าง shortcut link ไว้ใน `/etc/nginx/sites-enabled/` ด้วยคำสั่ง
```bash
sudo ln -s /etc/nginx/sites-available/your_domain.conf /etc/nginx/sites-enabled/
```
ตรวจสอบความถูกต้องของการตั้งค่า NGINX
```bash
sudo nginx -t
```
ทำการ Restart service ของ NGINX
```bash
sudo systemctl reload nginx
```
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

