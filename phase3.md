# 🟡 فاز ۳: سرورهای وب و پایگاه داده
## برنامه آموزشی جلسات ۱۷ تا ۲۴ — هر جلسه ۳۰ دقیقه

---

## 📌 جلسه ۱۷: نصب و پیکربندی Apache

### هدف
راه‌اندازی وب‌سرور Apache و درک ساختار آن

### دستورات

```bash
# نصب Apache
sudo apt update
sudo apt install apache2

# وضعیت سرویس
sudo systemctl status apache2
sudo systemctl enable apache2

# تست در مرورگر
# http://server-ip
# باید صفحه "It works!" را ببینید

# ساختار دایرکتوری Apache
/etc/apache2/
├── apache2.conf          # فایل اصلی
├── ports.conf            # پورت‌ها
├── sites-available/      # سایت‌های موجود
│   ├── 000-default.conf
│   └── default-ssl.conf
├── sites-enabled/        # سایت‌های فعال (symlink)
├── mods-available/       # ماژول‌های موجود
├── mods-enabled/         # ماژول‌های فعال
├── conf-available/
└── conf-enabled/

# فایل پیش‌فرض
sudo nano /etc/apache2/sites-available/000-default.conf
```

### نمونه Virtual Host

```bash
sudo nano /etc/apache2/sites-available/mysite.conf
```

```apache
<VirtualHost *:80>
    ServerName mysite.local
    ServerAlias www.mysite.local
    DocumentRoot /var/www/mysite

    <Directory /var/www/mysite>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mysite-error.log
    CustomLog ${APACHE_LOG_DIR}/mysite-access.log combined
</VirtualHost>
```

```bash
# ساخت دایرکتوری سایت
sudo mkdir -p /var/www/mysite
sudo chown -R $USER:$USER /var/www/mysite

# فایل تست
echo "<h1>Welcome to My Site</h1>" > /var/www/mysite/index.html

# فعال‌سازی سایت
sudo a2ensite mysite.conf
sudo a2dissite 000-default.conf

# فعال‌سازی ماژول‌ها
sudo a2enmod rewrite
sudo a2enmod ssl

# ری‌استارت
sudo systemctl restart apache2
```

### 🎯 تمرین عملی
1. Apache را نصب و start کنید
2. در مرورگر `http://server-ip` را باز کنید
3. یک Virtual Host جدید برای `test.local` بسازید
4. فایل `index.html` ساده بسازید
5. سایت را enable کنید و تست کنید
6. لاگ‌های دسترسی را ببینید (`tail /var/log/apache2/access.log`)

---

## 📌 جلسه ۱۸: نصب و پیکربندی Nginx

### هدف
راه‌اندازی وب‌سرور Nginx و مقایسه با Apache

### دستورات

```bash
# نصب Nginx
sudo apt install nginx

# وضعیت
sudo systemctl status nginx
sudo systemctl enable nginx

# تست
# http://server-ip
# باید "Welcome to nginx!" را ببینید

# ساختار فایل‌ها
/etc/nginx/
├── nginx.conf            # فایل اصلی
├── sites-available/      # سایت‌های موجود
│   └── default
├── sites-enabled/        # سایت‌های فعال (symlink)
├── snippets/             # قطعات قابل استفاده مجدد
└── conf.d/               # فایل‌های اضافی

# فایل پیش‌فرض
sudo nano /etc/nginx/sites-available/default
```

### نمونه Server Block

```bash
sudo nano /etc/nginx/sites-available/mysite
```

```nginx
server {
    listen 80;
    server_name mysite.local www.mysite.local;

    root /var/www/mysite;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    error_log /var/log/nginx/mysite-error.log;
    access_log /var/log/nginx/mysite-access.log;
}
```

```bash
# فعال‌سازی
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

# تست تنظیمات
sudo nginx -t

# ری‌لود
sudo systemctl reload nginx
```

### مقایسه Apache vs Nginx

| ویژگی | Apache | Nginx |
|:---|:---|:---|
| معماری | Process-based | Event-driven |
| مصرف RAM | بیشتر | کمتر |
| فایل استاتیک | خوب | عالی |
| Reverse Proxy | با ماژول | بومی |
| پیکربندی | .htaccess | فقط ادمین |
| پوپولاریتی | کلاسیک | مدرن |

### 🎯 تمرین عملی
1. Nginx را نصب کنید
2. صفحه خوش‌آمدگویی را در مرورگر ببینید
3. یک Server Block جدید بسازید
4. `nginx -t` را برای تست تنظیمات اجرا کنید
5. Apache را stop کنید، Nginx را روی پورت ۸۰ اجرا کنید
6. لاگ‌های Nginx را بررسی کنید

---

## 📌 جلسه ۱۹: SSL/TLS با Certbot (Let's Encrypt)

### هدف
فعال‌سازی HTTPS رایگان با گواهی Let's Encrypt

### پیش‌نیاز
- یک دامنه واقعی که به IP سرور اشاره کند
- پورت ۸۰ و ۴۴۳ باز باشد

### دستورات

```bash
# نصب Certbot
sudo apt install certbot python3-certbot-nginx
# یا برای Apache:
# sudo apt install certbot python3-certbot-apache

# دریافت گواهی با Nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# یا با Apache
sudo certbot --apache -d yourdomain.com

# فقط دریافت گواهی (بدون تغییر تنظیمات وب‌سرور)
sudo certbot certonly --standalone -d yourdomain.com

# گواهی‌ها کجا ذخیره می‌شوند
ls /etc/letsencrypt/live/yourdomain.com/
# ├── cert.pem
# ├── chain.pem
# ├── fullchain.pem
# └── privkey.pem

# تمدید خودکار (معمولاً cron تنظیم می‌شود)
sudo certbot renew --dry-run

# تمدیر دستی
sudo certbot renew

# لیست گواهی‌ها
sudo certbot certificates

# حذف گواهی
sudo certbot delete --cert-name yourdomain.com
```

### تنظیم دستی SSL در Nginx

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    root /var/www/yourdomain;
    index index.html;
}

# ریدایرکت HTTP به HTTPS
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

### 🎯 تمرین عملی
1. Certbot را نصب کنید
2. اگر دامنه دارید، گواهی بگیرید
3. اگر نه، با `--staging` تست کنید
4. `sudo certbot certificates` را ببینید
5. تمدید خودکار را تست کنید (`--dry-run`)
6. تنظیمات SSL در فایل Nginx/Apache را ببینید

---

## 📌 جلسه ۲۰: نصب MySQL/MariaDB

### هدف
راه‌اندازی پایگاه داده رابطه‌ای

### دستورات

```bash
# نصب MariaDB (توصیه شده — سازگار با MySQL)
sudo apt install mariadb-server mariadb-client

# یا MySQL
# sudo apt install mysql-server mysql-client

# وضعیت
sudo systemctl status mariadb
sudo systemctl enable mariadb

# تنظیمات اولیه امنیتی
sudo mysql_secure_installation
# مراحل:
# - Set root password? Y
# - Remove anonymous users? Y
# - Disallow root login remotely? Y
# - Remove test database? Y
# - Reload privilege tables? Y

# ورود به MySQL
sudo mysql -u root -p

# یا بدون پسورد (sudo)
sudo mysql
```

### دستورات SQL پایه

```sql
-- دیدن دیتابیس‌ها
SHOW DATABASES;

-- ساخت دیتابیس
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- ساخت کاربر
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';

-- دادن دسترسی
GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;

-- دیدن کاربرها
SELECT user, host FROM mysql.user;

-- استفاده از دیتابیس
USE mydb;

-- ساخت جدول
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- درج داده
INSERT INTO users (username, email) VALUES ('ali', 'ali@example.com');

-- خواندن
SELECT * FROM users;

-- خروج
EXIT;
```

### بک‌آپ و ری‌استور

```bash
# بک‌آپ
mysqldump -u root -p mydb > mydb_backup.sql

# ری‌استور
mysql -u root -p mydb < mydb_backup.sql

# بک‌آپ همه دیتابیس‌ها
mysqldump -u root -p --all-databases > all_databases.sql
```

### 🎯 تمرین عملی
1. MariaDB را نصب کنید
2. `mysql_secure_installation` را اجرا کنید
3. یک دیتابیس و یک کاربر بسازید
4. یک جدول ساده بسازید و داده وارد کنید
5. از دیتابیس بک‌آپ بگیرید
6. دیتابیس را حذف کنید و از بک‌آپ برگردانید

---

## 📌 جلسه ۲۱: PHP و LAMP Stack

### هدف
راه‌اندازی پشته LAMP (Linux + Apache + MySQL + PHP)

### دستورات

```bash
# نصب PHP و ماژول‌های رایج
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-bcmath

# نسخه PHP
php -v

# فایل تست PHP
sudo nano /var/www/html/info.php
```

```php
<?php
phpinfo();
?>
```

```bash
# تست در مرورگر
# http://server-ip/info.php

# فایل php.ini
sudo nano /etc/php/8.3/apache2/php.ini
# تنظیمات مهم:
# upload_max_filesize = 64M
# post_max_size = 64M
# max_execution_time = 300
# memory_limit = 256M

# ری‌استارت Apache بعد از تغییر
sudo systemctl restart apache2
```

### نمونه اتصال PHP به MySQL

```php
<?php
$host = 'localhost';
$db   = 'mydb';
$user = 'myuser';
$pass = 'StrongPassword123!';

$dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";

try {
    $pdo = new PDO($dsn, $user, $pass);
    echo "Connected successfully!";
} catch (PDOException $e) {
    echo "Connection failed: " . $e->getMessage();
}
?>
```

### 🎯 تمرین عملی
1. PHP و ماژول‌ها را نصب کنید
2. `info.php` را بسازید و در مرورگر ببینید
3. یک اسکریپت PHP بنویسید که به MariaDB وصل شود
4. `php.ini` را بررسی کنید
5. یک فرم ساده HTML+PHP بسازید که داده را در دیتابیس ذخیره کند

---

## 📌 جلسه ۲۲: Docker — مقدمات

### هدف
آشنایی با کانتینرها و Docker

### دستورات

```bash
# نصب Docker
sudo apt install docker.io

# یا با اسکریپت رسمی:
curl -fsSL https://get.docker.com | sh

# اضافه کردن کاربر به گروه docker
sudo usermod -aG docker $USER
# بعد logout/login کنید

# وضعیت
sudo systemctl status docker
sudo systemctl enable docker

# نسخه
docker --version
docker info
```

### دستورات پایه Docker

```bash
# دانلود و اجرای image
docker run hello-world

# اجرای Ubuntu در کانتینر
docker run -it ubuntu bash
# -i = interactive, -t = tty

# اجرای در پس‌زمینه
docker run -d nginx

# اجرا با پورت
docker run -d -p 8080:80 nginx
# -p host:container

# اجرا با نام
docker run -d --name my-nginx -p 8080:80 nginx

# اجرا با volume
docker run -d -v /host/data:/data nginx

# لیست کانتینرهای فعال
docker ps

# لیست همه کانتینرها
docker ps -a

# لیست imageها
docker images

# stop/start/restart
docker stop my-nginx
docker start my-nginx
docker restart my-nginx

# لاگ‌ها
docker logs my-nginx
docker logs -f my-nginx

# اجرای دستور در کانتینر
docker exec -it my-nginx bash

# حذف کانتینر
docker rm my-nginx
docker rm -f my-nginx  # force

# حذف image
docker rmi nginx

# پاک کردن همه کانتینرهای متوقف‌شده
docker container prune

# پاک کردن همه imageهای استفاده‌نشده
docker image prune -a
```

### 🎯 تمرین عملی
1. Docker را نصب کنید
2. `docker run hello-world` را اجرا کنید
3. یک کانتینر nginx روی پورت ۸۰۸۰ اجرا کنید
4. در مرورگر `http://server-ip:8080` را باز کنید
5. با `docker exec` وارد کانتینر شوید
6. کانتینر را stop و حذف کنید
7. `docker ps -a` و `docker images` را ببینید

---

## 📌 جلسه ۲۳: Docker Compose

### هدف
مدیریت چند کانتینر با یک فایل

### دستورات

```bash
# نصل Docker Compose
sudo apt install docker-compose
# یا نسخه جدیدتر:
# sudo apt install docker-compose-plugin

# نسخه
docker-compose --version
# یا
docker compose version
```

### نمونه docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: myapp
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppass
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      PMA_PORT: 3306

volumes:
  db_data:
```

```bash
# اجرا
docker-compose up -d

# دیدن وضعیت
docker-compose ps

# لاگ‌ها
docker-compose logs
docker-compose logs -f web

# اجرای دستور در سرویس
docker-compose exec db mysql -u root -p

# ری‌استارت
docker-compose restart

# stop
docker-compose down

# stop + حذف volumeها
docker-compose down -v

# rebuild
docker-compose up -d --build
```

### 🎯 تمرین عملی
1. `docker-compose.yml` بالا را بسازید
2. `docker-compose up -d` را اجرا کنید
3. سه سرویس را در `docker-compose ps` ببینید
4. `http://server-ip` (nginx) و `http://server-ip:8080` (phpMyAdmin) را تست کنید
5. با `docker-compose exec db bash` وارد MariaDB شوید
6. `docker-compose down -v` را اجرا کنید

---

## 📌 جلسه ۲۴: پروژه عملی — راه‌اندازی یک وب‌سایت کامل

### هدف
ترکیب همه چیزهایی که یاد گرفتید

### سناریو
راه‌اندازی یک وبلاگ ساده با:
- Nginx (Reverse Proxy + SSL)
- PHP-FPM
- MariaDB
- Docker Compose

### ساختار پروژه

```
~/myblog/
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── php/
│   └── Dockerfile
└── html/
    └── index.php
```

### فایل‌ها

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php
      - db

  php:
    build: ./php
    volumes:
      - ./html:/var/www/html

  db:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: blog
      MYSQL_USER: bloguser
      MYSQL_PASSWORD: blogpass
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

**php/Dockerfile:**
```dockerfile
FROM php:8.2-fpm
RUN docker-php-ext-install mysqli pdo pdo_mysql
```

**nginx/default.conf:**
```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

**html/index.php:**
```php
<?php
$host = 'db';
$db   = 'blog';
$user = 'bloguser';
$pass = 'blogpass';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8mb4", $user, $pass);
    echo "<h1>🎉 وبلاگ من راه افتاد!</h1>";
    echo "<p>اتصال به دیتابیس موفقیت‌آمیز بود.</p>";
} catch (PDOException $e) {
    echo "<h1>❌ خطا در اتصال</h1>";
    echo $e->getMessage();
}
?>
```

### اجرا

```bash
cd ~/myblog
docker-compose up -d --build

# تست
curl http://localhost
docker-compose logs
```

### 🎯 تمرین عملی
1. ساختار پروژه بالا را بسازید
2. `docker-compose up -d --build` را اجرا کنید
3. در مرورگر `http://server-ip` را باز کنید
4. یک جدول `posts` در دیتابیس `blog` بسازید
5. یک صفحه PHP بنویسید که پست‌ها را از دیتابیس بخواند
6. همه چیز را با `docker-compose down -v` پاک کنید

---

## ✅ چک‌لیست پایان فاز ۳

قبل از رفتن به فاز ۴، مطمئن شوید می‌توانید:

- [ ] Apache را نصب و پیکربندی کنید (Virtual Host)
- [ ] Nginx را نصب و پیکربندی کنید (Server Block)
- [ ] SSL با Certbot فعال کنید
- [ ] MariaDB/MySQL را نصب و مدیریت کنید
- [ ] LAMP Stack را راه‌اندازی کنید
- [ ] Docker را نصب و کانتینر اجرا کنید
- [ ] Docker Compose بنویسید و چند سرویس مدیریت کنید
- [ ] یک پروژه وب کامل با Docker راه‌اندازی کنید

---

> 🐳 **نکته Docker:** همیشه از `docker-compose down -v` با احتیاط استفاده کنید — volumeها حذف می‌شوند و داده‌ها از بین می‌روند!

**موفق باشید! 🟡**
