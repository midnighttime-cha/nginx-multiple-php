# วิธีติดตั้ง PHP หลาย Version และ NGINX บน Ubuntu 20.04
ความรู้ความเข้าใจพื้นฐาน
- สามารถใช้งาน Linux command line ได้
- สามารถตั้งค่า NGINX ขั้นพื้นฐานได้

## ติดตั้ง NGINX
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
sudo ufw allow 'Nginx OpenSSH'
```
ตรวจสอบว่า application ที่เราทำการ allow ไว้ด้วยคำสั่ง
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



