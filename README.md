# 🚀 Panduan Lengkap Instalasi & Deployment Wacentra di aaPanel (Linux VPS)

Dokumen ini memandu Anda melakukan instalasi platform **Wacentra** dari awal (fresh VPS) hingga siap beroperasi di **aaPanel** lengkap dengan database MySQL, service background 24/7 (Supervisor), Reverse Proxy Nginx, dan SSL HTTPS (Let's Encrypt).

---

## 📋 Daftar Isi
1. [Spesifikasi Server yang Direkomendasikan](#1-spesifikasi-server-yang-direkomendasikan)
2. [Langkah 1: Instalasi aaPanel di Fresh VPS (Terminal SSH)](#langkah-1-instalasi-aapanel-di-fresh-vps-terminal-ssh)
3. [Langkah 2: Instalasi Komponen di App Store aaPanel](#langkah-2-instalasi-komponen-di-app-store-aapanel)
4. [Langkah 3: Pembuatan Database MySQL di aaPanel](#langkah-3-pembuatan-database-mysql-di-aapanel)
5. [Langkah 4: Instalasi Golang & Node.js via Terminal aaPanel](#langkah-4-instalasi-golang--nodejs-via-terminal-aapanel)
6. [Langkah 5: Upload Source Code ke Server](#langkah-5-upload-source-code-ke-server)
7. [Langkah 6: Konfigurasi Environment (`.env`)](#langkah-6-konfigurasi-environment-env)
8. [Langkah 7: Build Frontend & Backend](#langkah-7-build-frontend--backend)
9. [Langkah 8: Menjalankan Backend 24/7 via Supervisor](#langkah-8-menjalankan-backend-247-via-supervisor)
10. [Langkah 9: Konfigurasi Domain, SSL, & Reverse Proxy Nginx](#langkah-9-konfigurasi-domain-ssl--reverse-proxy-nginx)
11. [Langkah 10: Uji Coba & Kredensial Awal](#langkah-10-uji-coba--kredensial-awal)
12. [Pemeliharaan & Troubleshooting](#pemeliharaan--troubleshooting)

---

## 1. Spesifikasi Server yang Direkomendasikan
* **Sistem Operasi**: Ubuntu 22.04 LTS / 24.04 LTS atau Debian 11 / 12 (64-bit).
* **CPU**: Minimal 1 vCPU (Disarankan 2 vCPU).
* **RAM**: Minimal 2 GB (Disarankan 4 GB untuk beban broadcast WhatsApp).
* **Penyimpanan**: Minimal 20 GB SSD.
* **Port Terbuka (Firewall)**: 80, 443, 3306 (lokal), dan port login aaPanel (biasanya acak saat instalasi, misal 7800 atau 8888).

---

## Langkah 1: Instalasi aaPanel di Fresh VPS (Terminal SSH)
Buka terminal SSH Anda (menggunakan PuTTY, Termius, atau PowerShell) dan masuk sebagai user `root`:

```bash
# Untuk Ubuntu / Debian:
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh && sudo bash install.sh aapanel
```

> **Catatan**: Setelah instalasi selesai, terminal akan menampilkan **Login URL**, **Username**, dan **Password** aaPanel Anda. Simpan data tersebut.

Buka Login URL aaPanel di browser Anda, lalu login menggunakan kredensial yang diberikan.

---

## Langkah 2: Instalasi Komponen di App Store aaPanel
Saat pertama kali login, aaPanel akan menawarkan paket instalasi LNMP / LAMP:
1. Pilih opsi **LNMP**.
2. Pastikan paket yang dipilih:
   * **Nginx** (Pilih versi `1.22` atau `1.24`)
   * **MySQL** (Pilih versi `5.7` atau `8.0`)
   * Metode instalasi: Pilih **Fast** (rekomendasi).
3. Setelah proses selesai, masuk ke menu **App Store** di sidebar aaPanel:
   * Cari: **Supervisor**
   * Klik **Install** (Tool ini digunakan agar backend Golang tetap menyala 24/7 dan auto-restart jika VPS reboot).

---

## Langkah 3: Pembuatan Database MySQL di aaPanel
1. Buka menu **Databases** di sidebar aaPanel.
2. Klik tombol **Add Database**.
3. Isi kolom sebagai berikut:
   * **DBName**: `wacentra`
   * **DB Character**: `utf8mb4`
   * **Username**: `wacentra_user`
   * **Password**: *(Buat password yang kuat atau klik generate, lalu catat)*
   * **Access Permission**: `Localhost` (Demi keamanan).
4. Klik **Submit**.

---

## Langkah 4: Instalasi Golang & Node.js via Terminal aaPanel
Buka menu **Terminal** di aaPanel (atau langsung via SSH terminal Anda sebagai `root`), lalu jalankan baris perintah berikut:

```bash
# 1. Update repository dan pasang tools pembantu
apt update -y && apt install -y git curl build-essential

# 2. Pasang Node.js 20 LTS & npm
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# 3. Pasang Golang 1.22+
rm -rf /usr/local/go
curl -LO https://go.dev/dl/go1.22.6.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.22.6.linux-amd64.tar.gz
rm go1.22.6.linux-amd64.tar.gz

# 4. Tambahkan Go ke Environment PATH sistem
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# 5. Verifikasi instalasi (Pastikan semua perintah mengeluarkan versi)
go version
node -v
npm -v
```

---

## Langkah 5: Upload Source Code ke Server
Standar direktori website pada aaPanel berada di `/www/wwwroot/`.

```bash
# Masuk ke direktori web root
cd /www/wwwroot/

# Buat folder proyek
mkdir -p /www/wwwroot/wacentra
cd /www/wwwroot/wacentra
```

**Pilihan Upload Code:**
* **Opsi A (Git Clone)**:
  ```bash
  git clone <URL_REPOSITORY_ANDA> /www/wwwroot/wacentra
  ```
* **Opsi B (Upload File ZIP via GUI aaPanel)**:
  1. Masuk ke menu **Files** di sidebar aaPanel.
  2. Buka folder `/www/wwwroot/wacentra`.
  3. Klik tombol **Upload**, pilih file ZIP proyek Anda.
  4. Klik kanan file ZIP tersebut -> pilih **Uncompress**.

Pastikan permission folder dapat diakses dengan baik:
```bash
chmod -R 755 /www/wwwroot/wacentra
```

---

## Langkah 6: Konfigurasi Environment (`.env`)
Salin file template `.env` dan sesuaikan dengan data produksi server Anda:

```bash
cd /www/wwwroot/wacentra
cp .env.example .env
nano .env
```

Sesuaikan baris-baris kunci berikut:
```env
# Mode Produksi
APP_ENV=production
PORT=3030

# Ganti dengan domain asli Anda (lengkap dengan https://)
APP_URL=https://domainanda.com
FRONTEND_URL=https://domainanda.com

# Kunci Keamanan JWT (Ganti dengan teks acak panjang)
JWT_SECRET=wacentra-secret-super-panjang-dan-aman-2026-production

# Koneksi MySQL aaPanel (Sesuaikan dengan Langkah 3)
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=wacentra_user
DB_PASS=PasswordDatabaseAnda
DB_NAME=wacentra

# Pool Koneksi Database
DB_MAX_OPEN_CONNS=25
DB_MAX_IDLE_CONNS=5
DB_CONN_MAX_LIFETIME_MIN=30

# Akun Awal Superadmin
SUPERADMIN_USERNAME=admin
SUPERADMIN_PASSWORD=PasswordAdminKuat2026!

# Payment Gateway Duitku (Opsional, isi jika sudah aktif)
DUITKU_MERCHANT_CODE=
DUITKU_API_KEY=
DUITKU_SANDBOX=false
DUITKU_CALLBACK_URL=https://domainanda.com/api/payment/duitku/callback
DUITKU_RETURN_URL=https://domainanda.com/billing
```
*(Tekan `Ctrl + O`, lalu `Enter` untuk simpan, dan `Ctrl + X` untuk keluar dari nano).*

---

## Langkah 7: Build Frontend & Backend
Jalankan kompilasi aset frontend dan binary backend Golang di terminal:

```bash
# 1. Kompilasi Frontend Vite (React)
cd /www/wwwroot/wacentra/frontend
npm install
npm run build

# 2. Kompilasi Backend Golang
cd /www/wwwroot/wacentra/backend
go build -o server main.go
chmod +x server
```

> **Info**: Hasil build backend akan menghasilkan file binary `/www/wwwroot/wacentra/backend/server`. File ini siap dieksekusi tanpa membutuhkan Node.js runtime lagi untuk backend-nya.

---

## Langkah 8: Menjalankan Backend 24/7 via Supervisor
Agar backend Go menyala otomatis, tidak mati saat terminal ditutup, dan otomatis restart saat VPS reboot:

1. Buka dashboard aaPanel -> menu **App Store**.
2. Cari **Supervisor**, lalu klik **Setting**.
3. Masuk ke tab **Add Daemon**.
4. Isi data berikut:
   * **Name**: `wacentra`
   * **Run User**: `root`
   * **Run Dir**: `/www/wwwroot/wacentra`
   * **Command**: `/www/wwwroot/wacentra/backend/server`
   * **Number of processes**: `1`
5. Klik **Confirm**.
6. Periksa daftar daemon Supervisor, pastikan status `wacentra` bertuliskan hijau **Running**.

*(Jika statusnya STOPPED, klik icon mata/log untuk melihat pesan kendalanya).*

---

## Langkah 9: Konfigurasi Domain, SSL, & Reverse Proxy Nginx

### 9.1 Menambahkan Domain di aaPanel
1. Masuk ke menu **Website** di sidebar aaPanel.
2. Klik tombol **Add site**.
   * **Domain**: Masukkan domain Anda (misal: `domainanda.com` atau `app.domainanda.com`).
   * **Database**: `Don't create` (karena sudah dibuat di Langkah 3).
   * **PHP Version**: `Static` (tidak membutuhkan PHP).
3. Klik **Submit**.

### 9.2 Mengaktifkan SSL HTTPS (Let's Encrypt)
*(Pastikan Domain / DNS A Record sudah diarahkan ke IP VPS Anda)*
1. Pada daftar website, klik nama domain Anda.
2. Buka tab **SSL** di sidebar kiri popup.
3. Pilih tab **Let's Encrypt**.
4. Centang nama domain Anda, lalu klik tombol **Apply**.
5. Setelah SSL aktif, centang toggle **Force HTTPS**.

### 9.3 Konfigurasi Reverse Proxy Nginx
1. Masih di jendela popup pengaturan domain, buka tab **Reverse Proxy**.
2. Klik tombol **Add Reverse Proxy**.
   * **Proxy Name**: `wacentra-proxy`
   * **Target URL**: `http://127.0.0.1:3030`
   * **Sent Domain**: `$host`
3. Klik **Submit**.
4. Setelah dibuat, klik menu **Config** (tombol edit) pada proxy tersebut.
5. Ganti seluruh isi konfigurasinya dengan konfigurasi optimal berikut (mendukung WebSocket live chat & upload media besar):

```nginx
location / {
    proxy_pass http://127.0.0.1:3030;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Pengaturan WebSocket (Wajib untuk Realtime WhatsApp Live Chat)
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    # Batas upload media WhatsApp (gambar/dokumen hingga 50MB)
    client_max_body_size 50m;

    # Timeout koneksi real-time
    proxy_read_timeout 86400s;
    proxy_send_timeout 86400s;
}
```
6. Klik **Save**.

---

## Langkah 10: Uji Coba & Kredensial Awal
Buka browser Anda dan akses domain yang telah dikonfigurasi:

* **Landing Page**: `https://domainanda.com`
* **Login Tenant**: `https://domainanda.com/login`
* **Superadmin Panel**: `https://domainanda.com/admin`
  * Gunakan username dan password yang disetel pada `.env`:
  * **Username**: `admin`
  * **Password**: `PasswordAdminKuat2026!`

---

## Pemeliharaan & Troubleshooting

### Cara Update Kode Terbaru (Deploy Ulang):
Jika ada pembaruan kode di kemudian hari, jalankan perintah ini di terminal:
```bash
cd /www/wwwroot/wacentra

# Tarik update kode (jika via Git)
git pull

# Rebuild frontend jika ada perubahan UI
cd /www/wwwroot/wacentra/frontend
npm install
npm run build

# Rebuild binary backend jika ada perubahan Go
cd /www/wwwroot/wacentra/backend
go build -o server main.go
```
Lalu restart service di aaPanel:
* Buka **App Store** -> **Supervisor** -> **Setting** -> klik tombol **Restart** pada daemon `wacentra`.

### Memeriksa Log jika Terjadi Kendala:
* **Log Backend**: Buka **Supervisor** -> klik icon **Log** pada baris `wacentra`.
* **Cek Port 3030 Aktif**:
  ```bash
  netstat -tlpn | grep 3030
  ```
* **Izin Folder Media / Upload**:
  Pastikan folder data dan uploads dapat ditulis oleh aplikasi:
  ```bash
  mkdir -p /www/wwwroot/wacentra/backend/data/uploads
  chmod -R 775 /www/wwwroot/wacentra/backend/data
  ```
