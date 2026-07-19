# 🔴 فاز ۵: فوق حرفه‌ای — DevOps و زیرساخت
## برنامه آموزشی جلسات ۳۳ تا ۴۰ — هر جلسه ۳۰ دقیقه

---

## 📌 جلسه ۳۳: Ansible — اتوماسیون سرور

### هدف
مدیریت چند سرور با یک Playbook

### نصب و راه‌اندازی

```bash
# نصل Ansible (روی ماشین کنترل)
sudo apt install ansible

# نسخه
ansible --version

# ساختار دایرکتوری
mkdir ~/ansible-project
cd ~/ansible-project
mkdir -p inventory group_vars host_vars roles
```

### فایل Inventory

```ini
# inventory/hosts
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu
web2 ansible_host=192.168.1.11 ansible_user=ubuntu

[dbservers]
db1 ansible_host=192.168.1.20 ansible_user=ubuntu

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Playbook نمونه

```yaml
# playbook.yml
---
- name: Setup Web Servers
  hosts: webservers
  become: yes

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy index.html
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html

    - name: Allow HTTP through UFW
      ufw:
        rule: allow
        port: '80'
        proto: tcp

- name: Setup Database Servers
  hosts: dbservers
  become: yes

  tasks:
    - name: Install MariaDB
      apt:
        name: mariadb-server
        state: present

    - name: Start MariaDB
      service:
        name: mariadb
        state: started
        enabled: yes
```

### اجرا

```bash
# تست اتصال
ansible all -i inventory/hosts -m ping

# اجرای یک دستور روی همه
ansible webservers -i inventory/hosts -m shell -a "uptime"

# اجرای playbook
ansible-playbook -i inventory/hosts playbook.yml

# dry-run (check mode)
ansible-playbook -i inventory/hosts playbook.yml --check

# limit به یک هاست
ansible-playbook -i inventory/hosts playbook.yml --limit web1

# verbose
ansible-playbook -i inventory/hosts playbook.yml -vvv
```

### 🎯 تمرین عملی
1. Ansible را نصب کنید
2. یک فایل inventory با ۲ سرور (حتی VM محلی) بسازید
3. `ansible all -m ping` را تست کنید
4. یک Playbook برای نصب nginx بنویسید
5. Playbook را اجرا کنید
6. با `--check` یک تغییر را تست کنید

---

## 📌 جلسه ۳۴: Kubernetes (k3s/minikube)

### هدف
آشنایی با orchestration کانتینرها

### k3s — Kubernetes سبک

```bash
# نصب k3s (سرور)
curl -sfL https://get.k3s.io | sh -

# وضعیت
sudo systemctl status k3s

# kubectl
sudo kubectl get nodes
sudo kubectl get pods --all-namespaces

# برای استفاده بدون sudo
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config
export KUBECONFIG=~/.kube/config

# یا
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
```

### minikube (جایگزین)

```bash
# نصب
# https://minikube.sigs.k8s.io/docs/start/

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# استارت
minikube start --driver=docker

# وضعیت
minikube status
kubectl get nodes
```

### مفاهیم کلیدی Kubernetes

```bash
# Node
kubectl get nodes
kubectl describe node <node-name>

# Namespace
kubectl get namespaces
kubectl create namespace myapp
kubectl config set-context --current --namespace=myapp

# Pod
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash

# Deployment
kubectl get deployments
kubectl scale deployment <name> --replicas=3
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>

# Service
kubectl get services
kubectl get svc

# Ingress
kubectl get ingress
```

### نمونه Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

```bash
kubectl apply -f deployment.yaml
kubectl get pods
kubectl get svc

# حذف
kubectl delete -f deployment.yaml
```

### 🎯 تمرین عملی
1. k3s یا minikube را نصب کنید
2. `kubectl get nodes` را ببینید
3. Deployment بالا را اجرا کنید
4. `kubectl get pods` — ۳ replica باید باشد
5. یک Pod را حذف کنید — Kubernetes دوباره می‌سازد
6. سرویس را ببینید و در مرورگر تست کنید

---

## 📌 جلسه ۳۵: CI/CD با GitHub Actions

### هدف
اتوماتیک کردن build و deploy

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Server

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            git pull origin main
            npm install
            npm run build
            pm2 restart myapp
```

### GitLab CI (جایگزین)

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test

build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

deploy:
  stage: deploy
  image: alpine
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '
' | ssh-add -
  script:
    - ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST "cd /var/www/myapp && git pull && npm install && npm run build && pm2 restart myapp"
  only:
    - main
```

### 🎯 تمرین عملی
1. یک repo در GitHub بسازید
2. فایل `.github/workflows/deploy.yml` بسازید
3. Secrets (SERVER_HOST, SERVER_USER, SSH_PRIVATE_KEY) را در GitHub تنظیم کنید
4. یک commit بزنید و Actions را ببینید
5. لاگ‌های Actions را بررسی کنید
6. (اختیاری) GitLab CI هم تست کنید

---

## 📌 جلسه ۳۶: Prometheus + Grafana

### هدف
مانیتورینگ حرفه‌ای با داشبورد

### نصب با Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro

volumes:
  prometheus_data:
  grafana_data:
```

### تنظیمات Prometheus

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx:9113']
```

### Node Exporter روی سرور

```bash
# نصب node-exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
sudo cp node_exporter-*/node_exporter /usr/local/bin/

# سرویس systemd
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo useradd -rs /bin/false node_exporter
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
# پورت ۹۱۰۰
```

### تنظیم Grafana

```bash
# بعد از docker-compose up:
# http://server-ip:3000
# user: admin, pass: admin123

# مراحل:
# 1. Add data source → Prometheus → URL: http://prometheus:9090
# 2. Import dashboard → ID: 1860 (Node Exporter Full)
# 3. Save
```

### 🎯 تمرین عملی
1. Docker Compose بالا را اجرا کنید
2. Prometheus را در `http://server-ip:9090` ببینید
3. Targets را چک کنید (`Status → Targets`)
4. node-exporter را روی سرور نصب کنید
5. Grafana را باز کنید و Prometheus را add data source کنید
6. Dashboard ۱۸۶۰ را import کنید
7. داشبورد زیبای مانیتورینگ را ببینید! 📊

---

## 📌 جلسه ۳۷: Nginx Reverse Proxy + Load Balancer

### هدف
پیکربندی حرفه‌ای Nginx برای چند سرویس

### Reverse Proxy

```nginx
# /etc/nginx/sites-available/proxy
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Load Balancing Methods

```nginx
upstream backend {
    # Round Robin (پیش‌فرض)
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;

    # Weighted
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=1;

    # Least Connections
    least_conn;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;

    # IP Hash (session sticky)
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;

    # Health checks
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 backup;  # فقط اگر بقیه down باشند
}
```

### Rate Limiting

```nginx
# در http {} block
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        limit_conn addr 10;
        proxy_pass http://backend;
    }
}
```

### 🎯 تمرین عملی
1. ۲ سرویس ساده روی پورت ۳۰۰۰ و ۳۰۰۱ اجرا کنید (مثلاً با Python http.server)
2. Nginx را Reverse Proxy کنید
3. Load Balancing با round-robin تست کنید
4. `ip_hash` را تست کنید
5. Rate Limiting اضافه کنید
6. با `curl` چند بار بزنید و ببینید کدام backend پاسخ می‌دهد

---

## 📌 جلسه ۳۸: VPN با WireGuard

### هدف
راه‌اندازی VPN شخصی روی Ubuntu Server

### نصب WireGuard

```bash
# نصب
sudo apt install wireguard

# ساخت کلیدها
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey

# تنظیم permissions
sudo chmod 600 /etc/wireguard/privatekey
```

### تنظیمات سرور

```bash
sudo nano /etc/wireguard/wg0.conf
```

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client 1
PublicKey = <CLIENT1_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32

[Peer]
# Client 2
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.0.0.3/32
```

```bash
# فعال‌سازی IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# فایروال
sudo ufw allow 51820/udp

# استارت
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

# وضعیت
sudo wg show
sudo wg showconf wg0
```

### تنظیمات کلاینت

```ini
# client1.conf
[Interface]
Address = 10.0.0.2/32
PrivateKey = <CLIENT1_PRIVATE_KEY>
DNS = 1.1.1.1, 8.8.8.8

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = server-ip:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

```bash
# نصب روی کلاینت
# Android/iOS: اپ WireGuard
# Windows/Mac: WireGuard app
# Linux: wg-quick up client1.conf
```

### 🎯 تمرین عملی
1. WireGuard را نصب کنید
2. کلیدهای سرور و یک کلاینت بسازید
3. `wg0.conf` را پیکربندی کنید
4. IP forwarding را فعال کنید
5. `wg-quick up wg0` را اجرا کنید
6. QR Code برای کلاینت موبایل بسازید:
   `qrencode -t ansiutf8 < client1.conf`
7. از موبایل وصل شوید و `10.0.0.1` را ping کنید

---

## 📌 جلسه ۳۹: امنیت سخت‌افزاری و هسته

### هدف
Hardening پیشرفته سرور

### sysctl — تنظیمات کرنل

```bash
sudo nano /etc/sysctl.conf
```

```ini
# جلوگیری از IP Spoofing
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# غیرفعال کردن ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# غیرفعال کردن source routing
net.ipv4.conf.all.accept_source_route = 0

# محافظت در برابر SYN Flood
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# غیرفعال کردن IPv6 (اگر نیاز ندارید)
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# افزایش محدودیت فایل‌ها
fs.file-max = 2097152

# بهینه‌سازی swappiness
vm.swappiness = 10
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10
```

```bash
sudo sysctl -p
```

### limits.conf

```bash
sudo nano /etc/security/limits.conf
```

```
*    soft    nofile    65535
*    hard    nofile    65535
*    soft    nproc     65535
*    hard    nproc     65535
root soft    nofile    65535
root hard    nofile    65535
```

### SSH Hardening

```bash
sudo nano /etc/ssh/sshd_config
```

```
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 60
MaxSessions 2
AllowUsers ali reza
X11Forwarding no
AllowTcpForwarding no
```

```bash
sudo systemctl restart sshd
```

### چک‌لیست Hardening

```bash
# 1. به‌روزرسانی
sudo apt update && sudo apt upgrade -y
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# 2. فایروال
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# 3. Fail2Ban
sudo apt install fail2ban
sudo systemctl enable fail2ban

# 4. Audit
sudo apt install auditd
sudo auditctl -e 1

# 5. AIDE — فایل integrity
sudo apt install aide
sudo aideinit
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
sudo aide --check

# 6. Lynis — اسکنر امنیتی
sudo apt install lynis
sudo lynis audit system
```

### 🎯 تمرین عملی
1. `sysctl.conf` را با تنظیمات بالا آپدیت کنید
2. `limits.conf` را تنظیم کنید
3. SSH را harden کنید (Port + Key-only)
4. Lynis را نصب و اجرا کنید
5. گزارش Lynis را ببینید و هشدارها را برطرف کنید
6. AIDE را نصب و تست کنید

---

## 📌 جلسه ۴۰: پروژه نهایی — زیرساخت کامل یک استارتاپ

### هدف
ترکیب همه چیز در یک پروژه واقعی

### معماری پروژه

```
┌─────────────────────────────────────────┐
│           User (Browser/App)            │
└─────────────┬───────────────────────────┘
              │ HTTPS (443)
┌─────────────▼───────────────────────────┐
│         Cloudflare / DNS                │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│      Nginx (Reverse Proxy + SSL)        │
│    - Rate Limiting                      │
│    - Load Balancing                     │
└──────┬──────────────┬───────────────────┘
       │              │
┌──────▼──────┐  ┌────▼──────┐
│  App Node 1 │  │ App Node 2│  (Docker Swarm / K8s)
│  (Docker)   │  │ (Docker)  │
└──────┬──────┘  └────┬──────┘
       │              │
┌──────▼──────────────▼──────┐
│      Redis (Cache)         │
└─────────────┬──────────────┘
              │
┌─────────────▼──────────────┐
│    PostgreSQL / MariaDB    │
│    (Master + Replica)      │
└────────────────────────────┘
```

### Docker Compose پروژه

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
      - ./html:/var/www/html
    depends_on:
      - app
    restart: always

  app:
    build: ./app
    environment:
      - DB_HOST=db
      - DB_NAME=myapp
      - DB_USER=appuser
      - DB_PASS=${DB_PASSWORD}
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis
    deploy:
      replicas: 2
    restart: always

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./backup:/backup
    restart: always

  redis:
    image: redis:7-alpine
    restart: always

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    restart: always

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    restart: always

  backup:
    image: alpine
    volumes:
      - db_data:/data:ro
      - ./backup:/backup
    command: >
      sh -c "echo '0 2 * * * tar czf /backup/db_$$(date +\%Y\%m\%d).tar.gz /data' | crontab - && crond -f"
    restart: always

volumes:
  db_data:
  prometheus_data:
  grafana_data:
```

### اسکریپت Deploy

```bash
#!/bin/bash
# deploy.sh

set -euo pipefail

REMOTE="user@server-ip"
APP_DIR="/opt/myapp"

echo "🚀 Starting deployment..."

# بک‌آپ قبل از deploy
ssh $REMOTE "cd $APP_DIR && docker-compose exec -T db pg_dump -U appuser myapp > backup/pre-deploy-$(date +%s).sql"

# کد جدید
rsync -avz --exclude='.git' --exclude='node_modules' ./ $REMOTE:$APP_DIR/

# deploy
ssh $REMOTE "cd $APP_DIR && docker-compose pull && docker-compose up -d --build"

# health check
sleep 5
if curl -sf http://$REMOTE/health; then
    echo "✅ Deployment successful!"
else
    echo "❌ Health check failed! Rolling back..."
    ssh $REMOTE "cd $APP_DIR && docker-compose down && git reset --hard HEAD~1 && docker-compose up -d"
    exit 1
fi
```

### 🎯 تمرین عملی نهایی

1. **ساختار پروژه** را بسازید
2. **Docker Compose** را اجرا کنید
3. **Nginx** را Reverse Proxy + SSL تنظیم کنید
4. **دیتابیس** را با volume persistent کنید
5. **Grafana + Prometheus** را برای مانیتورینگ اضافه کنید
6. **بک‌آپ خودکار** را با cron تنظیم کنید
7. **Deploy script** را بنویسید و تست کنید
8. **WireGuard** را برای دسترسی امن ادمین تنظیم کنید
9. **Fail2Ban + UFW** را فعال کنید
10. **Lynis** را اجرا کنید و سرور را harden کنید

---

## ✅ چک‌لیست پایان فاز ۵ — شما یک SysAdmin/DevOps هستید! 🎉

- [ ] Ansible Playbook بنویسید و روی چند سرور اجرا کنید
- [ ] Kubernetes/k3s را راه‌اندازی و Deployment اجرا کنید
- [ ] CI/CD Pipeline با GitHub Actions بسازید
- [ ] Prometheus + Grafana را برای مانیتورینگ راه‌اندازی کنید
- [ ] Nginx Reverse Proxy + Load Balancer پیکربندی کنید
- [ ] WireGuard VPN شخصی راه‌اندازی کنید
- [ ] سرور را با sysctl, SSH hardening, Lynis ایمن کنید
- [ ] یک پروژه کامل با Docker + Monitoring + Backup + Deploy بسازید

---

## 🗺️ نقشه راه ادامه

| مسیر | ابزارهای بیشتر |
|:---|:---|
| **Cloud** | AWS, Azure, GCP, Terraform |
| **Advanced K8s** | Helm, ArgoCD, Istio |
| **Monitoring** | ELK Stack, Datadog, New Relic |
| **Security** | Vault, Trivy, Falco |
| **IaC** | Terraform, Pulumi |
| **GitOps** | ArgoCD, Flux |

---

> 🏆 **تبریک!** شما ۴۰ جلسه (۲۰ ساعت) آموزش را تمام کردید. از یک مبتدی به یک ادمین لینوکس تبدیل شدید که می‌تواند سرور را نصب، پیکربندی، ایمن، مانیتور و اتوماتیک کند. ادامه دهید و پروژه بسازید — تجربه بهترین معلم است!

**موفق باشید! 🔴**
