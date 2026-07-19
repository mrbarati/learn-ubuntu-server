<div align="center">

# 🐧 Learn Ubuntu Server
### از صفر تا قهرمان — آموزش جامع Ubuntu Server به فارسی

[![Ubuntu](https://img.shields.io/badge/Ubuntu_Server-24.04-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/server)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Progress](https://img.shields.io/badge/Progress-5%2F5%20Phases-brightgreen)]()

</div>

---

## 📖 معرفی

این مخزن یک برنامه آموزشی جامع و عملی برای یادگیری **Ubuntu Server** از سطح صفر (نصب) تا سطح فوق حرفه‌ای (DevOps & Infrastructure) است. هر جلسه حدود **۳۰ دقیقه** طراحی شده و کاملاً عملی است — یعنی دستورات را روی سرور خودتان اجرا می‌کنید و یاد می‌گیرید.

> 🎯 **هدف:** بعد از اتمام این دوره، شما قادر خواهید بود یک سرور لینوکس را به‌طور کامل نصب، پیکربندی، ایمن‌سازی، مانیتور و اتوماتیک کنید.

---

## 🗺️ نقشه راه

| فاز | نام | جلسات | سطح | فایل |
|:---:|:---|:---:|:---:|:---|
| 🟢 | **مبانی و راه‌اندازی** | ۱ تا ۸ | مبتدی | [phase1.md](phase1.md) |
| 🔵 | **شبکه و امنیت پایه** | ۹ تا ۱۶ | مبتدی-متوسط | [phase2.md](phase2.md) |
| 🟡 | **سرورهای وب و پایگاه داده** | ۱۷ تا ۲۴ | متوسط | [phase3.md](phase3.md) |
| 🟠 | **پیشرفته — امنیت و بهینه‌سازی** | ۲۵ تا ۳۲ | پیشرفته | [phase4.md](phase4.md) |
| 🔴 | **فوق حرفه‌ای — DevOps و زیرساخت** | ۳۳ تا ۴۰ | فوق حرفه‌ای | [phase5.md](phase5.md) |

**کل دوره:** ۴۰ جلسه × ۳۰ دقیقه = **۲۰ ساعت آموزش عملی**

---

## 📚 سرفصل‌ها

### 🟢 فاز ۱: مبانی و راه‌اندازی
<details>
<summary>مشاهده جلسات</summary>

| جلسه | موضوع |
|:---:|:---|
| ۱ | نصب Ubuntu Server |
| ۲ | دستورات پایه فایل‌سیستم (`ls`, `cd`, `mkdir`, `cp`, `mv`, `rm`) |
| ۳ | مدیریت فایل‌ها و جستجو (`cat`, `grep`, `find`, `locate`) |
| ۴ | مجوزها و مالکیت (`chmod`, `chown`) |
| ۵ | کاربران، گروه‌ها و Sudo |
| ۶ | ویرایشگر متن nano و vim |
| ۷ | مدیریت بسته‌ها با APT |
| ۸ | سیستم فایل و دیسک (`df`, `du`, `fdisk`, `mount`) |

</details>

### 🔵 فاز ۲: شبکه و امنیت پایه
<details>
<summary>مشاهده جلسات</summary>

| جلسه | موضوع |
|:---:|:---|
| ۹ | تنظیمات شبکه و Netplan |
| ۱۰ | DNS و Hostname |
| ۱۱ | SSH — اتصال از راه دور و کلیدها |
| ۱۲ | فایروال UFW |
| ۱۳ | مدیریت سرویس‌ها با systemd |
| ۱۴ | لاگ‌ها و عیب‌یابی (`journalctl`, `dmesg`) |
| ۱۵ | Cron Jobs — زمان‌بندی وظایف |
| ۱۶ | پشتیبان‌گیری با `tar` و `rsync` |

</details>

### 🟡 فاز ۳: سرورهای وب و پایگاه داده
<details>
<summary>مشاهده جلسات</summary>

| جلسه | موضوع |
|:---:|:---|
| ۱۷ | نصب و پیکربندی Apache (Virtual Host) |
| ۱۸ | نصب و پیکربندی Nginx (Server Block) |
| ۱۹ | SSL/TLS با Certbot (Let's Encrypt) |
| ۲۰ | MySQL/MariaDB |
| ۲۱ | PHP و LAMP Stack |
| ۲۲ | Docker — مقدمات |
| ۲۳ | Docker Compose |
| ۲۴ | **پروژه عملی:** راه‌اندازی وب‌سایت کامل با Docker |

</details>

### 🟠 فاز ۴: پیشرفته — امنیت و بهینه‌سازی
<details>
<summary>مشاهده جلسات</summary>

| جلسه | موضوع |
|:---:|:---|
| ۲۵ | SELinux و AppArmor (MAC) |
| ۲۶ | Fail2Ban — محافظت از حملات |
| ۲۷ | مانیتورینگ با htop, glances, Netdata |
| ۲۸ | RAID و LVM |
| ۲۹ | NFS — اشتراک فایل در شبکه |
| ۳۰ | Samba — اشتراک با ویندوز |
| ۳۱ | Bash Scripting — مقدمات |
| ۳۲ | Bash Scripting — پیشرفته + اتوماسیون |

</details>

### 🔴 فاز ۵: فوق حرفه‌ای — DevOps و زیرساخت
<details>
<summary>مشاهده جلسات</summary>

| جلسه | موضوع |
|:---:|:---|
| ۳۳ | Ansible — اتوماسیون چند سرور |
| ۳۴ | Kubernetes (k3s/minikube) |
| ۳۵ | CI/CD با GitHub Actions |
| ۳۶ | Prometheus + Grafana |
| ۳۷ | Nginx Reverse Proxy + Load Balancer |
| ۳۸ | VPN با WireGuard |
| ۳۹ | Hardening پیشرفته (sysctl, SSH, Lynis) |
| ۴۰ | **پروژه نهایی:** زیرساخت کامل یک استارتاپ |

</details>

---

## 🚀 شروع سریع

### پیش‌نیازها

- یک کامپیوتر یا ماشین مجازی (VM)
- Ubuntu Server ISO ([دانلود از سایت رسمی](https://ubuntu.com/download/server))
- اتصال اینترنت
- کمی صبر و حوصله! 😊

### نحوه استفاده

1. مخزن را clone کنید:
   ```bash
   git clone https://github.com/mrbarati/learn-ubuntu-server.git
   cd learn-ubuntu-server
   ```

2. Ubuntu Server را روی کامپیوتر یا VM نصب کنید

3. از [phase1.md](phase1.md) شروع کنید و جلسه به جلسه پیش بروید

4. هر دستور را روی سرور خودتان اجرا کنید — **یادگیری عملی** کلید موفقیت است!

### پیشنهاد زمان‌بندی

- **۳ جلسه در هفته** (مثلاً شنبه، دوشنبه، چهارشنبه)
- هر جلسه **۳۰ دقیقه** تمرکز
- **جمعه‌ها مرور** هفته
- **مدت کل:** حدود ۱۳ هفته (کمتر از ۴ ماه!)

---

## ✅ چک‌لیست پیشرفت

### 🟢 فاز ۱ — مبانی
- [ ] Ubuntu Server نصب و راه‌اندازی شده
- [ ] در فایل‌سیستم حرکت می‌کنم
- [ ] مجوزها را می‌فهمم و تغییر می‌دهم
- [ ] کاربر و گروه می‌سازم
- [ ] با nano و vim کار می‌کنم
- [ ] بسته نصب/حذف/به‌روز می‌کنم (APT)
- [ ] فضای دیسک را بررسی می‌کنم

### 🔵 فاز ۲ — شبکه و امنیت پایه
- [ ] IP و Netplan را تنظیم می‌کنم
- [ ] Hostname و DNS را مدیریت می‌کنم
- [ ] با SSH از راه دور وصل می‌شوم
- [ ] کلید SSH ساخته و استفاده می‌کنم
- [ ] UFW را پیکربندی و فعال می‌کنم
- [ ] سرویس‌ها را با systemctl مدیریت می‌کنم
- [ ] لاگ‌ها را می‌خوانم و عیب‌یابی می‌کنم
- [ ] Cron Job می‌سازم
- [ ] با tar و rsync بک‌آپ می‌گیرم

### 🟡 فاز ۳ — وب و دیتابیس
- [ ] Apache را با Virtual Host پیکربندی می‌کنم
- [ ] Nginx را با Server Block پیکربندی می‌کنم
- [ ] SSL با Certbot فعال می‌کنم
- [ ] MariaDB/MySQL را نصب و مدیریت می‌کنم
- [ ] LAMP Stack را راه‌اندازی می‌کنم
- [ ] Docker را نصب و کانتینر اجرا می‌کنم
- [ ] Docker Compose می‌نویسم
- [ ] پروژه وب کامل با Docker راه‌اندازی می‌کنم

### 🟠 فاز ۴ — پیشرفته
- [ ] AppArmor/SELinux را پیکربندی می‌کنم
- [ ] Fail2Ban را تنظیم می‌کنم
- [ ] سرور را مانیتور می‌کنم (Netdata/Grafana)
- [ ] RAID یا LVM می‌سازم
- [ ] NFS Server/Client راه‌اندازی می‌کنم
- [ ] Samba Share برای ویندوز می‌سازم
- [ ] اسکریپت Bash با توابع می‌نویسم
- [ ] اسکریپت اتوماسیون (بک‌آپ/مانیتورینگ) می‌سازم

### 🔴 فاز ۵ — DevOps
- [ ] Ansible Playbook می‌نویسم و اجرا می‌کنم
- [ ] Kubernetes/k3s را راه‌اندازی می‌کنم
- [ ] CI/CD Pipeline با GitHub Actions می‌سازم
- [ ] Prometheus + Grafana را راه‌اندازی می‌کنم
- [ ] Nginx Reverse Proxy + Load Balancer پیکربندی می‌کنم
- [ ] WireGuard VPN شخصی راه‌اندازی می‌کنم
- [ ] سرور را با Lynis harden می‌کنم
- [ ] پروژه نهایی (زیرساخت کامل) را می‌سازم

---

## 🛠️ تکنولوژی‌ها و ابزارها

<div align="center">

![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)

</div>

---

## 💡 نکات طلایی

| نکته | توضیح |
|:---|:---|
| 📝 **یادداشت‌برداری** | هر دستور جدید را در یک فایل Markdown شخصی بنویسید |
| 🧪 **تمرین عملی** | هر جلسه را روی سرور واقعی خودتان اجرا کنید |
| 🔁 **مرور هفتگی** | جمعه‌ها ۱۵ دقیقه مرور جلسات هفته قبل |
| 🐛 **عیب‌یابی = یادگیری** | وقتی خطا می‌گیرید، اول خودتان جستجو کنید |
| 🔒 **احتیاط** | قبل از `rm -rf` یا `ufw enable` دو بار فکر کنید! |

---

## 📖 منابع تکمیلی

- [Ubuntu Server Official Documentation](https://ubuntu.com/server/docs)
- [The Linux Documentation Project](https://tldp.org/)
- [DigitalOcean Tutorials](https://www.digitalocean.com/community/tutorials)
- [Linuxize](https://linuxize.com/)

---

## 🤝 مشارکت

اگر اشتباهی پیدا کردید یا پیشنهادی دارید، خوشحال می‌شوم Pull Request بزنید یا Issue باز کنید!

---

<div align="center">

### ⭐ اگر این مخزن مفید بود، ستاره بدید!

**ساخته‌شده با ❤️ برای یادگیری Ubuntu Server**

</div>
