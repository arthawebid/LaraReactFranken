Bagus! Kamu pakai **Laravel 12 + React (frontend)** dan ingin jalan di **FrankenPHP** di VPS Debian — ini stack yang modern dan performa tinggi (native PHP server + asynchronous support).

Berikut **langkah-langkah lengkap untuk deploy Laravel + React dengan FrankenPHP di Debian VPS**.

---

## ⚙️ **Apa Itu FrankenPHP?**

FrankenPHP adalah server PHP modern (dibangun di atas Caddy) yang:

* Bisa jalanin PHP langsung (tanpa Nginx/Apache/FPM)
* Mendukung Laravel Octane, HTTP/2, SSE, dan worker
* Ideal untuk Laravel + React SPA

---

## ✅ Langkah-Langkah Deploy Laravel + React dengan FrankenPHP

---

### 1. 🔐 **Login VPS & Update Debian**

```bash
ssh root@ip-vps-kamu
apt update && apt upgrade -y
```

---

### 2. 🐘 **Install PHP & Composer**

Laravel 12 butuh PHP 8.2 atau lebih:

```bash
apt install php8.2 php8.2-cli php8.2-mbstring php8.2-xml php8.2-bcmath php8.2-curl php8.2-mysql unzip git curl composer -y
```

---

### 3. 🚀 **Install FrankenPHP**

Install binary-nya langsung:

```bash
curl -fsSL https://frankenphp.dev/install.sh | sh
```

Cek versi:

```bash
frankenphp --version
```

---

### 4. 🌐 **Clone Laravel Project**

```bash
cd /var/www
git clone https://github.com/kamu/laravel-react-app.git laravelapp
cd laravelapp
composer install
npm install && npm run build # Untuk React
cp .env.example .env
php artisan key:generate
```

---

### 5. 🗂️ **Atur File `.env` Laravel**

```dotenv
APP_ENV=production
APP_URL=https://domainkamu.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laraveluser
DB_PASSWORD=yourpassword
```

---

### 6. 🎛️ **Atur Permissions**

```bash
chown -R www-data:www-data /var/www/laravelapp
chmod -R 775 storage bootstrap/cache
```

---

### 7. 🧠 **Buat Config FrankenPHP**

Buat file config:
`nano /var/www/laravelapp/Caddyfile`

Isi Caddyfile:

```caddyfile
https://domainkamu.com {
    root * /var/www/laravelapp/public
    php {
        router /index.php
        worker /index.php
    }

    encode gzip

    file_server
}
```

> Ganti `https://domainkamu.com` dengan domain kamu (bisa `http://` untuk testing lokal).

---

### 8. 🛠️ **Jalankan FrankenPHP**

```bash
cd /var/www/laravelapp
frankenphp run --config Caddyfile
```

Kalau kamu mau **jalankan sebagai service (systemd)**:

```bash
nano /etc/systemd/system/frankenphp.service
```

Isi:

```ini
[Unit]
Description=FrankenPHP
After=network.target

[Service]
ExecStart=/usr/local/bin/frankenphp run --config /var/www/laravelapp/Caddyfile
WorkingDirectory=/var/www/laravelapp
Restart=always
User=www-data
Group=www-data

[Install]
WantedBy=multi-user.target
```

Lalu:

```bash
systemctl daemon-reexec
systemctl daemon-reload
systemctl enable --now frankenphp
```

---

### 9. 🧪 **Testing**

* Buka `https://domainkamu.com`
* Cek Laravel + React SPA berhasil jalan.
* Pastikan file `public/index.php` dan React `build` tersedia (di `public/`).

---

## 🔐 Tambahan: HTTPS Otomatis (Let's Encrypt)

FrankenPHP (pakai Caddy) otomatis ambil SSL dari Let's Encrypt kalau kamu pakai domain valid dan port 443 terbuka.

---

## 🔁 Tambahan: Laravel Octane (opsional)

Kalau kamu ingin performa maksimal:

```bash
composer require laravel/octane
php artisan octane:install
php artisan octane:start --server=frankenphp
```

Tapi kalau sudah pakai `php { worker /index.php }` di Caddyfile, FrankenPHP bisa langsung handle persistent worker **tanpa Octane**.

---

Kalau kamu butuh:

* Contoh Caddyfile untuk subdomain
* Docker setup untuk Laravel + React + FrankenPHP
* CI/CD deploy pipeline (GitHub Actions)

Aku bisa bantu. Mau lanjut ke mana sekarang?
