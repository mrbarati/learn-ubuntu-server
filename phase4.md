# 🟠 فاز ۴: پیشرفته — امنیت و بهینه‌سازی
## برنامه آموزشی جلسات ۲۵ تا ۳۲ — هر جلسه ۳۰ دقیقه

---

## 📌 جلسه ۲۵: SELinux و AppArmor

### هدف
درک و مدیریت Mandatory Access Control (MAC) برای امنیت بیشتر

### AppArmor (پیش‌فرض Ubuntu)

```bash
# وضعیت AppArmor
sudo aa-status

# لیست پروفایل‌ها
ls /etc/apparmor.d/

# حالت‌های پروفایل
# enforce = اجباری (پیش‌فرض)
# complain = فقط گزارش
# disabled = غیرفعال

# تغییر حالت پروفایل
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx

# دیدن لاگ AppArmor
sudo dmesg | grep apparmor
sudo journalctl | grep apparmor

# نمونه پروفایل سفارشی
sudo nano /etc/apparmor.d/usr.local.myapp
```

```apparmor
#include <tunables/global>

/usr/local/bin/myapp {
  #include <abstractions/base>

  /usr/local/bin/myapp r,
  /var/log/myapp.log w,
  /etc/myapp/** r,

  deny /etc/shadow r,
}
```

```bash
# بارگذاری پروفایل
sudo apparmor_parser -r /etc/apparmor.d/usr.local.myapp

# ری‌استارت AppArmor
sudo systemctl restart apparmor
```

### SELinux (در Ubuntu نیاز به نصب)

```bash
# نصب (اختیاری — AppArmor پیش‌فرض است)
sudo apt install selinux-basics selinux-policy-default

# وضعیت
sestatus
getenforce

# تغییر حالت
sudo setenforce 0   # permissive
sudo setenforce 1   # enforcing

# دیدن کانتکست‌ها
ls -Z
ps -eZ

# تغییر کانتکست
sudo chcon -t httpd_sys_content_t /var/www/html/index.html
sudo restorecon -v /var/www/html/index.html
```

### 🎯 تمرین عملی
1. `sudo aa-status` را اجرا کنید — چه پروفایل‌هایی فعال‌اند؟
2. یک پروفایل AppArmor را به complain تغییر دهید
3. دوباره enforce کنید
4. لاگ AppArmor را ببینید
5. تفاوت AppArmor و SELinux را در یک جمله بنویسید

---

## 📌 جلسه ۲۶: Fail2Ban — محافظت از حملات

### هدف
جلوگیری از حملات brute-force و اسکن

### دستورات

```bash
# نصب
sudo apt install fail2ban

# وضعیت
sudo systemctl status fail2ban
sudo systemctl enable failban

# فایل‌های تنظیمات
/etc/fail2ban/
├── fail2ban.conf       # تنظیمات عمومی
├── jail.conf           # jail پیش‌فرض (تغییر ندهید!)
└── jail.local          # تنظیمات شما ✅

# ساخت jail.local
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

### تنظیمات jail.local

```ini
[DEFAULT]
# زمان بن (ثانیه) = ۱ ساعت
bantime = 3600

# تعداد تلاش قبل از بن
maxretry = 3

# فاصله زمانی برای شمارش (ثانیه) = ۱۰ دقیقه
findtime = 600

# ارسال ایمیل (اختیاری)
# destemail = admin@example.com
# sender = fail2ban@example.com
# mta = sendmail
# action = %(action_mwl)s

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log

[apache-auth]
enabled = true
port = http,https
logpath = /var/log/apache2/error.log
```

```bash
# ری‌استارت
sudo systemctl restart fail2ban

# وضعیت jailها
sudo fail2ban-client status
sudo fail2ban-client status sshd

# دیدن IPهای بن‌شده
sudo fail2ban-client status sshd | grep "Banned IP list"
sudo iptables -L -n | grep DROP

# آزاد کردن دستی IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# دیدن لاگ
sudo tail -f /var/log/fail2ban.log
```

### 🎯 تمرین عملی
1. Fail2Ban را نصب کنید
2. `jail.local` بسازید و SSH را پیکربندی کنید (maxretry=3)
3. از کامپیوتر دیگر ۴ بار SSH با پسورد اشتباه بزنید
4. `sudo fail2ban-client status sshd` را ببینید — IP شما بن شده؟
5. IP را unban کنید
6. لاگ Fail2Ban را بررسی کنید

---

## 📌 جلسه ۲۷: مانیتورینگ سرور

### هدف
نظارت بر منابع و عملکرد سرور

### htop (پیشنهادی)

```bash
sudo apt install htop
htop

# میانبرها:
# F1 = Help
# F2 = Setup (رنگ، ستون‌ها)
# F3 = Search
# F4 = Filter
# F5 = Tree view
# F6 = Sort by
# F9 = Kill process
# F10 = Quit
# / = Search
# t = Tree mode
# Space = Tag process
```

### glances

```bash
sudo apt install glances
glances

# میانبرها:
# h = Help
# q = Quit
# d = Show/hide disk I/O
# n = Show/hide network
# s = Show/hide sensors
# 1 = Per-CPU stats
# b = Bytes or bits for network
# l = Alert logs
# w = Delete warning alerts
```

### Netdata (مانیتورینگ وب)

```bash
# نصب
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# یا با apt
sudo apt install netdata

# تنظیمات
sudo nano /etc/netdata/netdata.conf
# bind to = 0.0.0.0

# ری‌استارت
sudo systemctl restart netdata

# دسترسی
# http://server-ip:19999
```

### دستورات CLI مفید

```bash
# مصرف RAM
free -h

# مصرف CPU (هر ۲ ثانیه)
watch -n 2 'cat /proc/loadavg'

# پروسس‌ها
ps aux --sort=-%mem | head -20    # ۲۰ پروسس پرمصرف RAM
ps aux --sort=-%cpu | head -20    # ۲۰ پروسس پرمصرف CPU

# I/O دیسک
iotop    # نیاز به sudo apt install iotop

# شبکه
nload    # نیاز به sudo apt install nload
iftop    # نیاز به sudo apt install iftop

# دما (اگر سنسور دارید)
sensors  # نیاز به sudo apt install lm-sensors
```

### 🎯 تمرین عملی
1. `htop` را نصب و اجرا کنید — پروسس‌های فعال را ببینید
2. `glances` را نصب کنید و شبکه/دیسک را بررسی کنید
3. Netdata را نصب کنید و داشبورد وب را ببینید
4. `ps aux --sort=-%mem | head -10` — ۱۰ پروسس پرمصرف RAM
5. `free -h` — وضعیت RAM را بررسی کنید
6. یک پروسس مصرف‌کننده را در htop پیدا و kill کنید

---

## 📌 جلسه ۲۸: RAID و LVM

### هدف
مدیریت پیشرفته دیسک‌ها

### RAID با mdadm

```bash
# نصب
sudo apt install mdadm

# دیدن دیسک‌ها
lsblk
sudo fdisk -l

# ساخت RAID 1 (Mirror) — نیاز به ۲ دیسک
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# ساخت RAID 5 — نیاز به ۳+ دیسک
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# وضعیت RAID
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# فرمت کردن
sudo mkfs.ext4 /dev/md0

# mount
sudo mkdir /mnt/raid
sudo mount /dev/md0 /mnt/raid

# اضافه به fstab
echo '/dev/md0 /mnt/raid ext4 defaults 0 0' | sudo tee -a /etc/fstab

# ذخیره تنظیمات
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

### LVM (Logical Volume Manager)

```bash
# نصب (معمولاً نصب است)
sudo apt install lvm2

# ساختار: PV → VG → LV
# Physical Volume → Volume Group → Logical Volume

# ۱. ساخت Physical Volume
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc

# دیدن PV
sudo pvdisplay
sudo pvs

# ۲. ساخت Volume Group
sudo vgcreate vg_data /dev/sdb /dev/sdc

# دیدن VG
sudo vgdisplay
sudo vgs

# ۳. ساخت Logical Volume
sudo lvcreate -L 10G -n lv_www vg_data
sudo lvcreate -l 100%FREE -n lv_backup vg_data

# دیدن LV
sudo lvdisplay
sudo lvs

# فرمت و mount
sudo mkfs.ext4 /dev/vg_data/lv_www
sudo mkdir /mnt/www
sudo mount /dev/vg_data/lv_www /mnt/www

# افزایش سایز LV (آنلاین!)
sudo lvextend -L +5G /dev/vg_data/lv_www
sudo resize2fs /dev/vg_data/lv_www

# snapshot
sudo lvcreate -L 2G -s -n lv_www_snap /dev/vg_data/lv_www

# حذف snapshot
sudo lvremove /dev/vg_data/lv_www_snap
```

### 🎯 تمرین عملی
1. `lsblk` و `sudo fdisk -l` را ببینید
2. اگر ۲ دیسک مجازی دارید، RAID 1 بسازید
3. یا LVM بسازید: PV → VG → LV
4. LV را فرمت و mount کنید
5. سایز LV را افزایش دهید
6. `/etc/fstab` را آپدیت کنید

---

## 📌 جلسه ۲۹: NFS — اشتراک فایل در شبکه

### هدف
اشتراک دایرکتوری بین سرورهای لینوکس

### سرور NFS

```bash
# نصب
sudo apt install nfs-kernel-server

# ساخت دایرکتوری اشتراکی
sudo mkdir -p /srv/nfs/share
sudo chown nobody:nogroup /srv/nfs/share
sudo chmod 777 /srv/nfs/share

# تنظیمات export
sudo nano /etc/exports
```

```
# /etc/exports
/srv/nfs/share  192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/backup 192.168.1.10(ro,sync,no_subtree_check)
```

```bash
# اعمال exports
sudo exportfs -a
sudo exportfs -v    # دیدن exports فعال

# ری‌استارت
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server

# وضعیت
showmount -e localhost
```

### کلاینت NFS

```bash
# نصب
sudo apt install nfs-common

# دیدن exports سرور
showmount -e 192.168.1.100

# mount
sudo mkdir -p /mnt/nfs-share
sudo mount -t nfs 192.168.1.100:/srv/nfs/share /mnt/nfs-share

# mount خودکار در fstab
echo '192.168.1.100:/srv/nfs/share /mnt/nfs-share nfs defaults 0 0' | sudo tee -a /etc/fstab

# umount
sudo umount /mnt/nfs-share
```

### 🎯 تمرین عملی
1. NFS Server را نصب کنید
2. یک دایرکتوری export کنید
3. از خود سرور `showmount -e localhost` را ببینید
4. روی همان سرور (یا VM دیگر) کلاینت NFS را mount کنید
5. یک فایل در `/srv/nfs/share` بسازید و در `/mnt/nfs-share` ببینید
6. fstab را تنظیم کنید

---

## 📌 جلسه ۳۰: Samba — اشتراک با ویندوز

### هدف
اشتراک فایل با کامپیوترهای ویندوز

### دستورات

```bash
# نصب
sudo apt install samba samba-common-bin

# پشتیبان از فایل اصلی
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak

# تنظیمات
sudo nano /etc/samba/smb.conf
```

```ini
[global]
   workgroup = WORKGROUP
   server string = Ubuntu Server
   security = user
   map to guest = bad user
   dns proxy = no

[share]
   comment = Shared Folder
   path = /srv/samba/share
   browseable = yes
   read only = no
   guest ok = yes
   create mask = 0775
   directory mask = 0775

[private]
   comment = Private Folder
   path = /srv/samba/private
   valid users = @smbusers
   browseable = yes
   read only = no
   create mask = 0770
   directory mask = 0770
```

```bash
# ساخت دایرکتوری‌ها
sudo mkdir -p /srv/samba/share /srv/samba/private
sudo chmod 777 /srv/samba/share
sudo chmod 770 /srv/samba/private

# ساخت گروه و کاربر
sudo groupadd smbusers
sudo useradd -M -s /sbin/nologin smbuser
sudo usermod -aG smbusers smbuser

# تنظیم پسورد Samba (جدا از سیستم)
sudo smbpasswd -a smbuser

# تست تنظیمات
sudo testparm

# ری‌استارت
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd nmbd

# وضعیت
sudo systemctl status smbd

# دیدن shares
smbclient -L localhost -U%
```

### اتصال از ویندوز
```
# در File Explorer:
\\server-ip\share

# یا map network drive:
\\server-ip\share
```

### 🎯 تمرین عملی
1. Samba را نصب کنید
2. یک share public و یک share private بسازید
3. کاربر Samba بسازید و پسورد تنظیم کنید
4. `testparm` را برای تست اجرا کنید
5. از ویندوز یا دستور `smbclient` به share وصل شوید
6. یک فایل در share بسازید

---

## 📌 جلسه ۳۱: Bash Scripting — مقدمات

### هدف
نوشتن اسکریپت‌های کاربردی

### مبانی

```bash
#!/bin/bash
# اولین خط: shebang

# متغیرها
NAME="Ali"
AGE=25
echo "Hello, $NAME! You are $AGE years old."
echo "Hello, ${NAME}!"   # استفاده از {}

# متغیرهای محیطی
echo "User: $USER"
echo "Home: $HOME"
echo "PWD: $PWD"

# خواندن ورودی
read -p "Enter your name: " USERNAME
echo "Welcome, $USERNAME!"

# آرگومان‌های اسکریپت
echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Number of args: $#"

# مثال: ./script.sh arg1 arg2
```

### شرط‌ها

```bash
#!/bin/bash

# if-else
if [ "$1" == "start" ]; then
    echo "Starting..."
elif [ "$1" == "stop" ]; then
    echo "Stopping..."
else
    echo "Usage: $0 {start|stop}"
fi

# مقایسه اعداد
NUM=10
if [ $NUM -eq 10 ]; then
    echo "Equal"
fi

# -eq, -ne, -gt, -ge, -lt, -le
# =, !=, -z (empty), -n (not empty)

# فایل‌ها
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

# -f (file), -d (directory), -r (readable), -w (writable), -x (executable)
# -e (exists), -s (not empty)

# AND / OR
if [ -f "$1" ] && [ -r "$1" ]; then
    echo "File exists and readable"
fi

# case
case "$1" in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    *)
        echo "Unknown command"
        ;;
esac
```

### حلقه‌ها

```bash
#!/bin/bash

# for loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# range
for i in {1..10}; do
    echo "$i"
done

# with step
for i in {0..10..2}; do
    echo "$i"
done

# C-style
for ((i=0; i<5; i++)); do
    echo "Index: $i"
done

# loop on files
for file in *.txt; do
    echo "Processing: $file"
done

# while loop
COUNTER=0
while [ $COUNTER -lt 5 ]; do
    echo "Counter: $COUNTER"
    COUNTER=$((COUNTER + 1))
done

# until loop
COUNTER=0
until [ $COUNTER -ge 5 ]; do
    echo "Counter: $COUNTER"
    COUNTER=$((COUNTER + 1))
done

# read file line by line
while read line; do
    echo "$line"
done < file.txt
```

### 🎯 تمرین عملی
1. اسکریپتی بنویسید که نام کاربر را بگیرد و سلام کند
2. اسکریپتی بنویسید که چک کند آیا فایل داده‌شده وجود دارد
3. اسکریپتی بنویسید که ۱ تا ۱۰۰ را چاپ کند
4. اسکریپتی بنویسید که همه فایل‌های `.log` در `/var/log` را لیست کند
5. اسکریپتی با case بنویسید: start/stop/restart

---

## 📌 جلسه ۳۲: Bash Scripting — پیشرفته

### هدف
اسکریپت‌های حرفه‌ای با توابع و اتوماسیون

### توابع

```bash
#!/bin/bash

# تعریف تابع
my_function() {
    echo "Hello from function!"
}

# با آرگومان
greet() {
    local name=$1   # local = فقط داخل تابع
    echo "Hello, $name!"
}

greet "Ali"
greet "Reza"

# بازگشت مقدار
add() {
    local a=$1
    local b=$2
    echo $((a + b))
}

RESULT=$(add 5 3)
echo "Result: $RESULT"

# بررسی خطا
backup_file() {
    local file=$1
    if [ ! -f "$file" ]; then
        echo "Error: File not found" >&2
        return 1
    fi
    cp "$file" "$file.bak"
    return 0
}

backup_file "/etc/passwd"
if [ $? -eq 0 ]; then
    echo "Backup successful"
else
    echo "Backup failed"
fi
```

### اسکریپت عملی: مانیتورینگ سیستم

```bash
#!/bin/bash
# /usr/local/bin/system-check.sh

LOG_FILE="/var/log/system-check.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# تابع لاگ‌گیری
log_message() {
    echo "[$DATE] $1" | tee -a "$LOG_FILE"
}

# چک دیسک
check_disk() {
    local usage=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ "$usage" -gt 80 ]; then
        log_message "WARNING: Disk usage is ${usage}%"
    else
        log_message "INFO: Disk usage is ${usage}%"
    fi
}

# چک RAM
check_memory() {
    local usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    if [ "$usage" -gt 90 ]; then
        log_message "WARNING: Memory usage is ${usage}%"
    else
        log_message "INFO: Memory usage is ${usage}%"
    fi
}

# چک Load Average
check_load() {
    local load=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
    log_message "INFO: Load average: $load"
}

# اجرا
log_message "=== System Check Started ==="
check_disk
check_memory
check_load
log_message "=== System Check Completed ==="
```

```bash
# قابل اجرا کردن
sudo chmod +x /usr/local/bin/system-check.sh

# زمان‌بندی با cron
crontab -e
# اضافه کنید:
*/30 * * * * /usr/local/bin/system-check.sh
```

### اسکریپت عملی: بک‌آپ اتوماتیک

```bash
#!/bin/bash
# /usr/local/bin/auto-backup.sh

BACKUP_DIR="/backup"
SOURCE_DIR="/home/user/projects"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"
RETENTION_DAYS=7

# ساخت دایرکتوری
mkdir -p "$BACKUP_DIR"

# بک‌آپ
tar -czf "$BACKUP_FILE" "$SOURCE_DIR" 2>/dev/null

if [ $? -eq 0 ]; then
    echo "[$DATE] Backup created: $BACKUP_FILE"
else
    echo "[$DATE] Backup FAILED!" >&2
    exit 1
fi

# حذف بک‌آپ‌های قدیمی
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "[$DATE] Old backups cleaned (>$RETENTION_DAYS days)"
```

### نکات پیشرفته

```bash
# set options
set -e    # exit on error
set -u    # exit on undefined variable
set -o pipefail  # pipe fails if any command fails

# trap — اجرای کد در خروج
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT

# heredoc
cat << EOF > /tmp/config.txt
name=Ali
age=25
city=Tehran
EOF

# subshell
(
    cd /tmp
    echo "In /tmp: $PWD"
)
echo "Back to: $PWD"
```

### 🎯 تمرین عملی
1. اسکریپت system-check.sh را بسازید و اجرا کنید
2. اسکریپت auto-backup.sh را بسازید
3. یک تابع بنویسید که پینگ کند و وضعیت را برگرداند
4. اسکریپتی بنویسید که همه سرویس‌های failed را restart کند
5. اسکریپت بک‌آپ را با cron زمان‌بندی کنید

---

## ✅ چک‌لیست پایان فاز ۴

قبل از رفتن به فاز ۵، مطمئن شوید می‌توانید:

- [ ] AppArmor/SELinux را پیکربندی کنید
- [ ] Fail2Ban را برای SSH و وب‌سرور تنظیم کنید
- [ ] سرور را با htop/glances/Netdata مانیتور کنید
- [ ] RAID یا LVM بسازید و مدیریت کنید
- [ ] NFS Server/Client راه‌اندازی کنید
- [ ] Samba Share برای ویندوز بسازید
- [ ] اسکریپت Bash با توابع بنویسید
- [ ] اسکریپت اتوماسیون (بک‌آپ/مانیتورینگ) بسازید

---

> 🛡️ **نکته امنیتی:** همیشه اسکریپت‌های bash را قبل از اجرای production تست کنید. از `set -euo pipefail` در اسکریپت‌های مهم استفاده کنید.

**موفق باشید! 🟠**
