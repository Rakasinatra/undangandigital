# Deploy Laravel ke cPanel

Dokumen ini adalah runbook deploy aplikasi undangan digital ke cPanel. Contoh path memakai akun cPanel `clickund` dan domain `clickundangan.my.id`; sesuaikan jika akun/domain berbeda.

## Ringkasan Struktur

Struktur yang paling aman untuk cPanel biasa:

```text
/home/clickund/laravel-app  -> isi project Laravel
/home/clickund/public_html  -> isi folder public Laravel
```

`public_html/index.php` harus menunjuk ke folder Laravel:

```php
require __DIR__.'/../laravel-app/vendor/autoload.php';
$app = require_once __DIR__.'/../laravel-app/bootstrap/app.php';
```

Jika domain memakai document root sendiri, misalnya:

```text
/home/clickund/clickundangan.my.id
```

isi folder public Laravel harus berada di folder document root itu, bukan di `public_html`.

## Sebelum Upload

- Pastikan test lokal aman:
  - login admin
  - tambah/edit undangan
  - preview `/undangan/{slug}?to=Nama+Tamu`
- Jangan simpan password production di repo.
- Buat password admin production yang kuat.
- Pastikan PHP cPanel minimal `8.3`; deploy yang berhasil memakai PHP `8.4.20`.

## Upload

1. Upload ZIP deploy ke home akun cPanel, misalnya `/home/clickund`.
2. Extract ZIP di `/home/clickund`.
3. Pastikan hasil extract berisi:

```text
laravel-app
public_html
UPLOAD-README.txt
```

4. Kalau upload lewat FTP sementara, hapus akun FTP itu setelah deploy selesai.
5. Setelah extract sukses, hapus file ZIP dari server.

## Konfigurasi `.env`

Edit:

```text
/home/clickund/laravel-app/.env
```

Contoh konfigurasi production:

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://clickundangan.my.id
APP_KEY=

ADMIN_USERNAME=Rakasinatra
ADMIN_PASSWORD=password-kuat

DB_CONNECTION=sqlite
```

Jika memakai MySQL cPanel:

```env
DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=nama_database
DB_USERNAME=user_database
DB_PASSWORD=password_database
```

## Perintah Setelah Extract

Jalankan dari Terminal cPanel:

```bash
cd /home/clickund/laravel-app

find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
chmod 755 artisan
chmod -R 775 storage
chmod -R 775 bootstrap/cache

grep -q '^APP_KEY=' .env || sed -i '1iAPP_KEY=' .env
php artisan key:generate --force

php artisan package:discover
composer dump-autoload
php artisan optimize:clear
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Jika `composer dump-autoload` tidak tersedia di hosting, pastikan folder `vendor` sudah ikut ter-upload dari paket deploy.

## Verifikasi

Cek dari Terminal cPanel:

```bash
curl -I https://clickundangan.my.id
curl -I https://clickundangan.my.id/admin/login
```

Hasil sehat:

```text
HTTP/2 200
```

Lalu test di browser:

```text
https://clickundangan.my.id
https://clickundangan.my.id/admin/login
```

## Troubleshooting 500

### Lihat log

Laravel log:

```bash
tail -n 120 /home/clickund/laravel-app/storage/logs/laravel.log
```

cPanel/public log:

```bash
tail -n 120 /home/clickund/public_html/error_log
```

Untuk log baru yang bersih:

```bash
cd /home/clickund/laravel-app
> storage/logs/laravel.log
> /home/clickund/public_html/error_log
curl -s https://clickundangan.my.id > /dev/null
head -n 120 storage/logs/laravel.log
head -n 120 /home/clickund/public_html/error_log
```

### `Permission denied`

Jika log berisi permission denied pada `vendor`, `app`, `storage`, atau `bootstrap/cache`, jalankan:

```bash
cd /home/clickund/laravel-app
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
chmod 755 artisan
chmod -R 775 storage
chmod -R 775 bootstrap/cache
php artisan optimize:clear
```

### `No application encryption key has been specified`

Jika log berisi:

```text
No application encryption key has been specified.
```

pastikan `.env` punya baris `APP_KEY=` lalu generate:

```bash
cd /home/clickund/laravel-app
grep -q '^APP_KEY=' .env || sed -i '1iAPP_KEY=' .env
php artisan key:generate --force
php artisan optimize:clear
```

### Domain membaca folder yang salah

Jika `artisan about` normal tapi website tetap 500/404, cek document root domain. Pada cPanel addon domain, document root bisa berupa:

```text
/home/clickund/clickundangan.my.id
```

bukan:

```text
/home/clickund/public_html
```

Cek isi folder:

```bash
ls -la /home/clickund/clickundangan.my.id
ls -la /home/clickund/public_html
```

Folder document root harus berisi `index.php`, `.htaccess`, `build`, dan `uploads`.

## Folder Upload Gambar

Admin menyimpan foto upload ke:

```text
public_html/uploads/invitations/{slug}
```

Pastikan folder writable:

```bash
mkdir -p /home/clickund/public_html/uploads
chmod -R 775 /home/clickund/public_html/uploads
```

Jika document root domain bukan `public_html`, sesuaikan path `uploads` ke folder document root tersebut.

## Checklist Akhir

- `APP_ENV=production`
- `APP_DEBUG=false`
- `APP_URL` memakai `https://domain`
- `APP_KEY` sudah terisi
- `php artisan migrate --force` sukses
- halaman utama `200 OK`
- `/admin/login` `200 OK`
- login admin berhasil
- file ZIP deploy sudah dihapus dari server
- akun FTP sementara sudah dihapus/disable
