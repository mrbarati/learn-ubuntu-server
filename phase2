# 🔵 فاز ۲: شبکه و امنیت پایه
## برنامه آموزشی جلسات ۹ تا ۱۶ — هر جلسه ۳۰ دقیقه

---

## 📌 جلسه ۹: تنظیمات شبکه

### هدف
درک و مدیریت تنظیمات شبکه در Ubuntu Server

### دستورات

```bash
# دیدن IP و وضعیت کارت شبکه
ip addr show
ip addr show eth0

# دیدن جدول مسیریابی (routing)
ip route show
ip route | grep default

# دستورات کلاسیک (هنوز کار می‌کنند)
ifconfig          # نیاز به: sudo apt install net-tools
ping -c 4 google.com

# تست اتصال به پورت
nc -zv google.com 80
# یا
curl -I http://google.com

# دیدن اتصالات فعال
ss -tuln          # TCP/UDP listening ports
netstat -tuln     # نیاز به net-tools

# ویرایش تنظیمات شبکه (Netplan)
sudo nano /etc/netplan/00-installer-config.yaml
```

### نمونه فایل Netplan (Static IP)

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

```bash
# اعمال تنظیمات
sudo netplan apply
sudo netplan try    # با تایم‌اوت ۱۲۰ ثانیه (اگر مشکل داشت برمی‌گردد)
```

### 🎯 تمرین عملی
1. `ip addr show` را اجرا کنید و IP سرور خود را یادداشت کنید
2. `ip route show` را ببینید — Gateway چیست؟
3. `ping 8.8.8.8` و `ping google.com` را تست کنید
4. `ss -tuln` — چه پورت‌هایی باز هستند؟
5. فایل Netplan را ببینید (`cat /etc/netplan/*.yaml`)
6. (اختیاری) IP استاتیک تنظیم کنید

---

## 📌 جلسه ۱۰: DNS و Hostname

### هدف
مدیریت نام سرور و تنظیمات DNS

### دستورات

```bash
# دیدن Hostname
hostname
hostnamectl

# تغییر Hostname
sudo hostnamectl set-hostname myserver

# فایل hosts
cat /etc/hosts
# ساختار:
# 127.0.0.1   localhost
# 127.0.1.1   myserver
# 192.168.1.5   other-server.local

# DNS Resolver
cat /etc/resolv.conf
# یا در Ubuntu جدید:
systemd-resolve --status
resolvectl status

# جستجوی DNS
nslookup google.com
dig google.com
dig @8.8.8.8 google.com    # استفاده از DNS خاص

# رکورد خاص
dig MX gmail.com            # Mail Exchange
dig NS google.com           # Name Server

# Reverse DNS
dig -x 8.8.8.8
```

### 🎯 تمرین عملی
1. Hostname فعلی را ببینید
2. Hostname را به `ubuntu-lab` تغییر دهید
3. `/etc/hosts` را ببینید — چه ورودی‌هایی دارد؟
4. `nslookup` و `dig` برای ۳ دامنه مختلف اجرا کنید
5. `/etc/resolv.conf` را بررسی کنید
6. یک ورودی به `/etc/hosts` اضافه کنید: `192.168.1.1 myrouter.local`

---

## 📌 جلسه ۱۱: SSH — اتصال از راه دور

### هدف
نصب، پیکربندی و استفاده امن از SSH

### دستورات

```bash
# نصب SSH Server (معمولاً نصب است)
sudo apt install openssh-server

# وضعیت سرویس
sudo systemctl status ssh

# اتصال از کلاینت (از کامپیوتر دیگر)
ssh username@server-ip
ssh -p 2222 username@server-ip    # اگر پورت تغییر کرده

# کپی فایل با SCP
scp file.txt username@server-ip:/home/username/
scp -r folder/ username@server-ip:/home/username/
scp username@server-ip:/etc/nginx/nginx.conf ./

# ساخت کلید SSH
ssh-keygen -t ed25519 -C "my-key"
# یا کلاسیک:
ssh-keygen -t rsa -b 4096

# کپی کلید عمومی به سرور
ssh-copy-id username@server-ip

# اتصال بدون پسورد (بعد از ssh-copy-id)
ssh username@server-ip

# تنظیمات SSH Server
sudo nano /etc/ssh/sshd_config
```

### تنظیمات امنیتی SSH (مهم!)

```bash
# در /etc/ssh/sshd_config:
Port 2222                    # تغییر پورت پیش‌فرض
PermitRootLogin no           # غیرفعال کردن لاگین روت
PasswordAuthentication no    # فقط کلید (بعد از تست کلید!)
PubkeyAuthentication yes
MaxAuthTries 3

# اعمال تغییرات
sudo systemctl restart ssh
```

### 🎯 تمرین عملی
1. مطمئن شوید SSH نصب و فعال است
2. از کامپیوتر/موبایل دیگر با `ssh` وصل شوید
3. یک فایل با `scp` به سرور کپی کنید
4. کلید SSH بسازید (`ssh-keygen`)
5. کلید را به سرور کپی کنید (`ssh-copy-id`)
6. بدون پسورد وارد شوید
7. (اختیاری) پورت SSH را تغییر دهید

---

## 📌 جلسه ۱۲: فایروال UFW

### هدف
محافظت از سرور با فایروال ساده Ubuntu

### دستورات

```bash
# نصب (معمولاً نصب است)
sudo apt install ufw

# وضعیت
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# سیاست پیش‌فرض
sudo ufw default deny incoming
sudo ufw default allow outgoing

# باز کردن پورت‌ها
sudo ufw allow 22/tcp        # SSH
sudo ufw allow 80/tcp        # HTTP
sudo ufw allow 443/tcp       # HTTPS
sudo ufw allow 2222/tcp      # SSH روی پورت دیگر

# باز کردن سرویس (با نام)
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# محدود کردن به IP خاص
sudo ufw allow from 192.168.1.0/24 to any port 22

# حذف قانون
sudo ufw delete allow 80/tcp
sudo ufw delete 3            # حذف بر اساس شماره

# فعال‌سازی
sudo ufw enable
# ⚠️ قبلش مطمئن شوید SSH را allow کردید!

# غیرفعال کردن
sudo ufw disable

# ریست
sudo ufw reset

# لاگ فایروال
sudo ufw logging on
```

### 🎯 تمرین عملی
1. `sudo ufw status` را ببینید
2. سیاست پیش‌فرض را deny incoming تنظیم کنید
3. SSH را allow کنید
4. پورت ۸۰ و ۴۴۳ را allow کنید
5. UFW را enable کنید
6. از کلاینت دیگر SSH بزنید — کار می‌کند؟
7. پورت ۲۳ (telnet) را allow کنید، سپس delete کنید

---

## 📌 جلسه ۱۳: مدیریت سرویس‌ها با systemd

### هدف
کنترل سرویس‌ها، بوت و وضعیت سیستم

### دستورات

```bash
# وضعیت سرویس
sudo systemctl status ssh
sudo systemctl status nginx

# استارت/استاپ/ری‌استارت
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx     # بدون قطع سرویس (تنظیمات جدید)

# enable/disable (بوت خودکار)
sudo systemctl enable nginx     # فعال در بوت
sudo systemctl disable nginx    # غیرفعال در بوت

# لیست سرویس‌ها
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-unit-files       # همه فایل‌ها

# سرویس‌های failed
systemctl --failed

# reboot/shutdown
sudo systemctl reboot
sudo systemctl poweroff

# Target‌ها (مشابه runlevel)
systemctl get-default           # graphical یا multi-user
sudo systemctl set-default multi-user.target

# Timerها (جایگزین cron در systemd)
systemctl list-timers
```

### 🎯 تمرین عملی
1. وضعیت سرویس `ssh` را ببینید
2. `nginx` را نصب کنید، start و enable کنید
3. `systemctl status nginx` — وضعیت active/running را ببینید
4. `systemctl list-units --type=service --state=running` — چه سرویس‌هایی فعال‌اند؟
5. nginx را stop و disable کنید
6. سرویس‌های failed را ببینید (`systemctl --failed`)

---

## 📌 جلسه ۱۴: لاگ‌ها و عیب‌یابی

### هدف
خواندن و تحلیل لاگ‌ها برای پیدا کردن مشکلات

### دستورات

```bash
# journalctl — لاگ‌های systemd
sudo journalctl
sudo journalctl -u ssh          # لاگ یک سرویس خاص
sudo journalctl -u nginx --since "1 hour ago"
sudo journalctl --since "2026-07-19 10:00" --until "2026-07-19 12:00"
sudo journalctl -f              # دنبال کردن زنده (مثل tail -f)
sudo journalctl -p err          # فقط خطاها
sudo journalctl --disk-usage    # فضای مصرفی
sudo journalctl --vacuum-time=7d # حذف لاگ‌های قدیمی‌تر از ۷ روز

# فایل‌های لاگ کلاسیک
cat /var/log/syslog
cat /var/log/auth.log           # لاگین‌ها و SSH
cat /var/log/kern.log           # کرنل
cat /var/log/dmesg              # بوت و سخت‌افزار

# dmesg
sudo dmesg | less
sudo dmesg | grep -i error
sudo dmesg --level=err,warn

# لاگ‌های برنامه‌های خاص
cat /var/log/nginx/access.log
cat /var/log/nginx/error.log

# دنبال کردن زنده
sudo tail -f /var/log/syslog
sudo tail -f /var/log/auth.log
```

### 🎯 تمرین عملی
1. `sudo journalctl -n 50` — ۵۰ لاگ آخر
2. `sudo journalctl -u ssh` — لاگ SSH
3. `sudo tail -f /var/log/auth.log` — در پنجره دیگر SSH بزنید و ببینید
4. `sudo dmesg | grep -i error` — خطاهای کرنل
5. `sudo journalctl --since "30 min ago"` — لاگ نیم ساعت اخیر
6. یک لاگ failed را پیدا کنید و سعی کنید علت را حدس بزنید

---

## 📌 جلسه ۱۵: Cron Jobs — زمان‌بندی وظایف

### هدف
اتوماتیک کردن وظایف دوره‌ای

### دستورات

```bash
# ویرایش crontab کاربر فعلی
crontab -e

# دیدن cron jobs
crontab -l

# حذف همه
crontab -r

# cron کاربر root
sudo crontab -e

# cron سیستمی
ls /etc/cron.d/
cat /etc/crontab
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.hourly/
```

### سینتکس Cron

```
* * * * *  command
│ │ │ │ │
│ │ │ │ └── day of week (0-7, 0=Sunday)
│ │ │ └──── month (1-12)
│ │ └────── day of month (1-31)
│ └──────── hour (0-23)
└────────── minute (0-59)
```

### نمونه Cron Jobs

```bash
# هر دقیقه
* * * * * echo "Hello" >> /tmp/cron.log

# هر ۵ دقیقه
*/5 * * * * /home/user/script.sh

# هر روز ساعت ۳ صبح
0 3 * * * /usr/bin/apt update

# هر یکشنبه ساعت ۲ صبح
0 2 * * 0 /home/user/backup.sh

# هر اول ماه ساعت نیمه‌شب
0 0 1 * * /home/user/monthly-report.sh

# هر ۱۰ دقیقه فقط در ساعات کاری
*/10 9-17 * * 1-5 /home/user/work-script.sh
```

### at — یک‌بار اجرا

```bash
# نصب
sudo apt install at

# زمان‌بندی یک دستور
at now + 5 minutes
at> echo "Done" | mail -s "Alert" user@localhost
at> Ctrl+D

# لیست
atq

# حذف
atrm 1
```

### 🎯 تمرین عملی
1. `crontab -e` را باز کنید
2. هر دقیقه یک پیام به `/tmp/test-cron.log` بنویسید
3. یک دقیقه صبر کنید و فایل را ببینید
4. آن job را حذف کنید
5. یک اسکریپت ساده بسازید که تاریخ را چاپ کند
6. آن را هر ۵ دقیقه در cron قرار دهید

---

## 📌 جلسه ۱۶: پشتیبان‌گیری با tar و rsync

### هدف
ساختن بک‌آپ و همگام‌سازی فایل‌ها

### دستورات tar

```bash
# ساخت آرشیو
tar -cvf backup.tar /home/user/documents
# -c = create, -v = verbose, -f = file

# آرشیو فشرده
tar -czvf backup.tar.gz /home/user/documents
# -z = gzip

tar -cjvf backup.tar.bz2 /home/user/documents
# -j = bzip2 (فشرده‌تر ولی کندتر)

# استخراج
tar -xvf backup.tar
tar -xzvf backup.tar.gz
tar -xjvf backup.tar.bz2 -C /tmp/restore

# دیدن محتوای آرشیو بدون استخراج
tar -tvf backup.tar.gz

# آرشیو کردن با exclude
tar -czvf backup.tar.gz /home/user --exclude='*.tmp' --exclude='Downloads'
```

### دستورات rsync

```bash
# همگام‌سازی محلی
rsync -av /home/user/documents/ /backup/documents/
# -a = archive, -v = verbose
# ⚠️ اسلش آخر مهم است! /source/ vs /source

# حذف فایل‌های مقصد که در مبدا نیستند
rsync -av --delete /home/user/documents/ /backup/documents/

# فشرده‌سازی در انتقال شبکه
rsync -avz /home/user/documents/ user@server:/backup/

# نمایش پیشرفت
rsync -av --progress /large-file /backup/

# dry-run (فقط نشان بده، انجام نده)
rsync -av --dry-run /source/ /dest/

# exclude
rsync -av --exclude='*.log' /source/ /dest/
```

### اسکریپت بک‌آپ ساده

```bash
#!/bin/bash
# /home/user/backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/$DATE"
SOURCE="/home/user/projects"

mkdir -p $BACKUP_DIR
tar -czvf "$BACKUP_DIR/projects_$DATE.tar.gz" $SOURCE
echo "Backup completed: $BACKUP_DIR"
```

```bash
# اجرای خودکار با cron
chmod +x /home/user/backup.sh
crontab -e
# اضافه کنید:
0 2 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1
```

### 🎯 تمرین عملی
1. دایرکتوری `~/test-backup` بسازید و چند فایل در آن بسازید
2. یک tar.gz از آن بسازید
3. tar.gz را استخراج کنید در `/tmp/restore-test`
4. `rsync -av ~/test-backup/ /tmp/rsync-test/` اجرا کنید
5. یک فایل جدید به `~/test-backup` اضافه کنید
6. دوباره rsync کنید — فقط فایل جدید کپی می‌شود
7. یک اسکریپت بک‌آپ ساده بنویسید و تست کنید

---

## ✅ چک‌لیست پایان فاز ۲

قبل از رفتن به فاز ۳، مطمئن شوید می‌توانید:

- [ ] IP سرور را ببینید و تنظیم کنید (Netplan)
- [ ] Hostname و DNS را مدیریت کنید
- [ ] با SSH از راه دور وصل شوید و کلید SSH بسازید
- [ ] فایروال UFW را پیکربندی و فعال کنید
- [ ] سرویس‌ها را با systemctl مدیریت کنید
- [ ] لاگ‌ها را بخوانید و عیب‌یابی کنید
- [ ] Cron job بسازید و زمان‌بندی کنید
- [ ] با tar و rsync بک‌آپ بگیرید

---

> 🔒 **نکته امنیتی:** همیشه قبل از فعال کردن UFW، پورت SSH را allow کنید. وگرنه خودتان را از سرور قفل می‌کنید!

**موفق باشید! 🔵**
