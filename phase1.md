# 🟢 فاز ۱: مبانی و راه‌اندازی Ubuntu Server
## برنامه آموزشی جلسات ۱ تا ۸ — هر جلسه ۳۰ دقیقه

---

## 📌 جلسه ۱: نصب Ubuntu Server

### هدف
نصب Ubuntu Server روی کامپیوتر شخصی (یا ماشین مجازی)

### مراحل

#### ۱. دانلود ISO
```bash
# از سایت رسمی Ubuntu دانلود کنید:
# https://ubuntu.com/download/server
```

#### ۲. ساختن Bootable USB (اختیاری — اگر روی سخت‌افزار واقعی نصب می‌کنید)
```bash
# در لینوکس:
sudo dd if=ubuntu-server.iso of=/dev/sdX bs=4M status=progress

# در ویندوز از Rufus استفاده کنید
```

#### ۳. نصب
- بوت از USB/DVD
- انتخاب زبان: English (یا Persian)
- Keyboard layout: Iranian (اگر فارسی می‌خواهید)
- نوع نصب: Ubuntu Server
- Network: DHCP (معمولاً خودکار)
- Proxy: خالی بگذارید (اگر نیاز ندارید)
- Mirror: پیش‌فرض
- Disk: Use entire disk
- Profile: نام کاربری و پسورد خود را وارد کنید
- SSH: ✅ تیک SSH server را بزنید
- Featured Server Snaps: خالی بگذارید

#### ۴. اولین ورود
```bash
# پس از ری‌استارت، با یوزری که ساختید لاگین کنید
username@hostname:~$ 
```

### 🎯 تمرین عملی
1. Ubuntu Server را نصب کنید
2. یک بار ری‌استارت کنید و مطمئن شوید بالا می‌آید
3. IP سرور را پیدا کنید: `ip addr show`

---

## 📌 جلسه ۲: دستورات پایه فایل‌سیستم

### هدف
آشنایی با دستورات ضروری برای حرکت در فایل‌سیستم

### دستورات

```bash
# محل فعلی
pwd

# لیست فایل‌ها
ls
ls -l          # جزئیات بیشتر
ls -la         # شامل فایل‌های مخفی
ls -lh         # سایز قابل خواندن

# تغییر دایرکتوری
cd /           # ریشه
cd ~           # هوم
cd ..          # یکی بالاتر
cd -           # قبلی

# ساخت دایرکتوری
mkdir test
mkdir -p a/b/c # ساخت تو در تو

# ساخت فایل خالی
touch file.txt

# کپی
cp file.txt file2.txt
cp -r dir1 dir2

# انتقال/تغییر نام
mv file.txt newname.txt
mv file.txt /tmp/

# حذف
rm file.txt
rm -r directory/
rm -rf directory/  # با احتیاط!

# پاک کردن ترمینال
clear
```

### 🎯 تمرین عملی
1. دایرکتوری `~/projects` بسازید
2. داخلش فایل `README.md` بسازید
3. یک دایرکتوری `backup` هم بسازید
4. `README.md` را کپی کنید به `backup/`
5. فایل اصلی را حذف کنید
6. با `ls -la` بررسی کنید

---

## 📌 جلسه ۳: مدیریت فایل‌ها و جستجو

### هدف
خواندن، جستجو و پیدا کردن فایل‌ها

### دستورات

```bash
# نمایش محتوای فایل
cat file.txt
cat -n file.txt      # با شماره خط

# نمایش صفحه به صفحه
less /var/log/syslog
# داخل less: q=خروج, /جستجو, n=بعدی, N=قبلی

# سر و ته فایل
head -20 file.txt    # ۲۰ خط اول
tail -20 file.txt    # ۲۰ خط آخر
tail -f /var/log/syslog  # دنبال کردن زنده (live)

# جستجو در فایل
grep "error" file.txt
grep -i "error" file.txt     # بدون حساسیت به بزرگی/کوچکی
grep -r "pattern" /etc/      # بازگشتی

# پیدا کردن فایل
find / -name "*.conf"
find /home -type f -size +100M
find . -name "*.log" -mtime +7  # فایل‌های لاگ قدیمی‌تر از ۷ روز

# جستجوی سریع (نیاز به updatedb)
updatedb
locate sshd_config
```

### 🎯 تمرین عملی
1. محتوای `/etc/os-release` را ببینید
2. ۱۰ خط آخر `/var/log/syslog` را ببینید
3. کلمه "failed" را در `/var/log/auth.log` جستجو کنید
4. همه فایل‌های `.conf` در `/etc` را پیدا کنید
5. فایل `sshd_config` را پیدا کنید (با locate یا find)

---

## 📌 جلسه ۴: مجوزها و مالکیت فایل‌ها

### هدف
درک و مدیریت دسترسی‌ها

### مفاهیم
```
-rwxr-xr--  1 ali users  1234 Jul 19 10:00 script.sh
 |  |  |
 |  |  └── others (سایر)
 |  └───── group (گروه)
 └──────── owner (مالک)

r = read (4)    | 7 = rwx
w = write (2)   | 6 = rw-
x = execute (1) | 5 = r-x
                | 4 = r--
```

### دستورات

```bash
# دیدن مجوزها
ls -l
ls -ld /var/www  # خود دایرکتوری

# تغییر مجوزها
chmod 755 script.sh
chmod u+x script.sh      # اضافه کردن execute برای مالک
chmod g-w file.txt       # حذف write از گروه
chmod a+r file.txt       # اضافه کردن read برای همه
chmod -R 755 mydir/      # بازگشتی

# تغییر مالک
sudo chown ali file.txt
sudo chown ali:users file.txt
sudo chown -R ali:ali myproject/

# تغییر گروه
sudo chgrp developers file.txt
```

### 🎯 تمرین عملی
1. فایل `test.sh` بسازید و بنویسید: `#!/bin/bash` و `echo "Hello"`
2. سعی کنید اجرا کنید: `./test.sh` → باید Permission denied بدهد
3. مجوز execute بدهید: `chmod +x test.sh`
4. دوباره اجرا کنید
5. مالک را به root تغییر دهید و سعی کنید ویرایش کنید

---

## 📌 جلسه ۵: کاربران، گروه‌ها و Sudo

### هدف
مدیریت کاربران و دسترسی‌های مدیریتی

### دستورات

```bash
# دیدن کاربر فعلی
whoami
id

# ساخت کاربر جدید
sudo useradd -m -s /bin/bash newuser
# -m = ساخت home directory
# -s = shell پیش‌فرض

# تنظیم پسورد
sudo passwd newuser

# اضافه کردن به گروه sudo (دسترسی ادمین)
sudo usermod -aG sudo newuser

# ساخت گروه
sudo groupadd developers
sudo usermod -aG developers newuser

# حذف کاربر
sudo userdel -r newuser  # -r = حذف home هم

# سوئیچ کاربر
su - newuser

# اجرای دستور با sudo
sudo apt update
sudo !!                  # اجرای دستور قبلی با sudo

# ویرایش sudoers (با احتیاط!)
sudo visudo
```

### 🎯 تمرین عملی
1. کاربری به نام `testuser` بسازید
2. پسورد برایش تنظیم کنید
3. او را به گروه `sudo` اضافه کنید
4. با `su - testuser` سوئیچ کنید
5. دستور `sudo whoami` را اجرا کنید → باید `root` برگرداند
6. کاربر را حذف کنید

---

## 📌 جلسه ۶: ویرایشگر متن — nano و vim

### هدف
تسلط بر ویرایش فایل‌های متنی در ترمینال

### Nano (ساده‌تر)

```bash
nano file.txt

# میانبرهای nano:
# Ctrl+O = ذخیره
# Ctrl+X = خروج
# Ctrl+K = برش خط
# Ctrl+U = پیست
# Ctrl+W = جستجو
# Ctrl+G = راهنما
```

### Vim (قدرتمندتر)

```bash
vim file.txt

# حالت‌ها:
# Normal (ESC) = حالت پیش‌فرض، حرکت و دستور
# Insert (i)   = حالت نوشتن
# Command (:)  = دستورات

# حرکت در Normal:
h j k l        # چپ، پایین، بالا، راست
w b            # کلمه بعدی/قبلی
gg G           # اول/آخر فایل
:10            # رفتن به خط ۱۰

# ویرایش:
i              # insert قبل مکان‌نما
a              # append بعد مکان‌نما
o              # خط جدید زیر
O              # خط جدید بالا
dd             # حذف خط
yy             # کپی خط
p              # پیست
u              # undo
Ctrl+r         # redo

# ذخیره و خروج:
:w             # ذخیره
:q             # خروج
:wq            # ذخیره و خروج
:q!            # خروج بدون ذخیره
```

### 🎯 تمرین عملی
1. با nano فایل `~/notes.txt` بسازید و چند خط بنویسید
2. ذخیره و خروج کنید
3. با vim همان فایل را باز کنید
4. چند خط اضافه کنید (حالت Insert)
5. یک خط را حذف کنید (`dd`)
6. ذخیره و خروج کنید (`:wq`)

---

## 📌 جلسه ۷: مدیریت بسته‌ها با APT

### هدف
نصب، به‌روزرسانی و حذف نرم‌افزارها

### دستورات

```bash
# به‌روزرسانی لیست بسته‌ها
sudo apt update

# ارتقای بسته‌های نصب‌شده
sudo apt upgrade
sudo apt full-upgrade  # با حذف/نصب وابستگی‌ها

# نصب بسته
sudo apt install nginx
sudo apt install -y htop tree curl

# حذف بسته
sudo apt remove nginx       # فقط بسته
sudo apt purge nginx        # بسته + فایل‌های تنظیمات

# پاک کردن وابستگی‌های اضافی
sudo apt autoremove

# جستجوی بسته
apt search web server
apt show nginx

# دیدن بسته‌های نصب‌شده
apt list --installed

# اطلاع از نسخه
nginx -v

# dpkg (سطح پایین‌تر)
dpkg -l | grep nginx
dpkg -L nginx    # فایل‌های نصب‌شده توسط nginx
```

### 🎯 تمرین عملی
1. `sudo apt update` و `sudo apt upgrade` اجرا کنید
2. بسته‌های `htop`, `tree`, `curl` را نصب کنید
3. `htop` را اجرا کنید و ببینید
4. `tree` را روی دایرکتوری home اجرا کنید
5. `nginx` را نصب کنید، سپس `purge` کنید
6. `apt autoremove` اجرا کنید

---

## 📌 جلسه ۸: سیستم فایل و دیسک

### هدف
مدیریت فضای دیسک و پارتیشن‌ها

### دستورات

```bash
# فضای دیسک
 df -h              # human-readable
 df -h /            # فقط ریشه

# سایز دایرکتوری‌ها
 du -sh /var/log    # سایز کل
 du -h --max-depth=1 /var  # سایز زیرشاخه‌ها

# دیدن پارتیشن‌ها
 lsblk
 fdisk -l           # نیاز به sudo

# mount/umount
 mount              # لیست mount شده‌ها
 sudo mount /dev/sdb1 /mnt
 sudo umount /mnt

# fstab — mount خودکار در بوت
cat /etc/fstab

# UUID دیسک‌ها
 sudo blkid

# LVM (مقدماتی)
 sudo pvdisplay     # Physical Volume
 sudo vgdisplay     # Volume Group
 sudo lvdisplay     # Logical Volume
```

### 🎯 تمرین عملی
1. `df -h` اجرا کنید — فضای باقی‌مانده را ببینید
2. `du -sh /var/log` — سایز لاگ‌ها
3. `lsblk` — ساختار دیسک‌ها
4. `sudo fdisk -l` — پارتیشن‌ها
5. `/etc/fstab` را ببینید
6. اگر USB دارید، آن را mount کنید به `/mnt`

---

## ✅ چک‌لیست پایان فاز ۱

قبل از رفتن به فاز ۲، مطمئن شوید می‌توانید:

- [ ] Ubuntu Server را نصب و راه‌اندازی کنید
- [ ] در فایل‌سیستم حرکت کنید و فایل مدیریت کنید
- [ ] مجوزها را بخوانید و تغییر دهید
- [ ] کاربر جدید بسازید و sudo بدهید
- [ ] با nano و vim فایل ویرایش کنید
- [ ] بسته نصب/حذف/به‌روز کنید
- [ ] فضای دیسک را بررسی کنید

---

