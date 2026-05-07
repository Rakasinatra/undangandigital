# Roadmap Platform Undangan Digital

Dokumen ini memetakan upgrade dari MVP undangan tunggal menjadi platform undangan digital dengan katalog tema, dashboard admin, customer area, dan editor undangan.

## Tujuan Produk

Target akhir bukan hanya halaman undangan, tetapi panel mirip WordPress khusus undangan:

- pengunjung bisa melihat katalog tema
- pelanggan bisa login dan membuat/mengelola undangan sendiri
- admin bisa mengelola tema, media, taxonomy, placeholder, pelanggan, pesanan, dan user login
- aset tema dan aset undangan bisa diupload, dipakai ulang, dicari, dan dirapikan

## Portal Utama

### Public Website

URL:

```text
/
/themes
/themes/{slug}
/pricing
/preview/{theme}
```

Fungsi:

- menampilkan katalog tema
- menampilkan detail dan preview tema
- CTA buat undangan
- halaman utama tidak langsung membuka undangan contoh

### Customer Area

URL:

```text
/login
/register
/dashboard
/orders
/invitations/create
/invitations/{invitation}/edit
/invitations/{invitation}/media
```

Fungsi:

- wajib login untuk membuat undangan
- pelanggan mengisi form data undangan
- pelanggan memilih tema
- pelanggan upload foto dan media sendiri
- pelanggan melihat status undangan/order
- pelanggan mendapatkan link undangan

### Admin Dashboard

URL:

```text
/admin
/admin/login
/admin/themes
/admin/theme-assets
/admin/media
/admin/taxonomies
/admin/placeholders
/admin/customers
/admin/orders
/admin/invitations
/admin/users
/admin/settings
```

Fungsi:

- panel pusat untuk operasional
- bukan hanya edit satu undangan
- admin bisa mengelola konten, user, pelanggan, tema, media, dan data master

## Modul Admin

### Dashboard

Isi:

- jumlah undangan aktif
- jumlah pelanggan
- order terbaru
- tema paling sering dipakai
- status draft/published/archived
- shortcut tambah tema, tambah undangan, upload media

### Theme Manager

Data yang dikelola:

- nama tema
- slug tema
- status: draft/published/archived
- kategori tema
- style tokens: warna, font, layout, spacing
- preview image
- demo invitation
- required assets
- schema form khusus tema

Contoh aset wajib tema:

```text
cover_photo
desktop_cover_photo
quote_photo
moment_photo
thanks_photo
bride_photo
groom_photo
gallery
background_music
```

### Placeholder Manager

Fungsi:

- mengatur field dinamis yang dipakai template
- menentukan label, help text, tipe input, validasi, dan default value
- membuat tema bisa punya form custom

Contoh placeholder:

```text
wedding.title
bride.full_name
groom.full_name
events.akad.venue
gift.account_number
gallery[]
```

### Taxonomy Manager

Fungsi:

- kategori tema
- tag tema
- style tema
- jenis event
- paket harga

Contoh:

```text
theme_category: elegant, classic, modern, islamic, minimalist
theme_style: dark, floral, editorial, luxury
event_type: wedding, engagement, birthday, corporate
```

### Media Library

Fungsi:

- upload media umum
- upload media per undangan
- filter berdasarkan owner, tema, tipe, tanggal upload
- rename, delete, replace
- copy URL
- assign media ke placeholder undangan

Jenis file:

```text
image
audio
video
document
```

### Invitation Manager

Fungsi:

- daftar semua undangan
- filter status/customer/theme
- edit data undangan
- preview sebagai tamu
- publish/unpublish
- duplicate undangan
- arsip/hapus

Status:

```text
draft
in_review
published
archived
```

### Customer Manager

Fungsi:

- daftar pelanggan
- profil pelanggan
- undangan milik pelanggan
- order pelanggan
- reset password
- impersonate customer untuk support

### Order Manager

Fungsi:

- paket yang dibeli
- status pembayaran manual
- status pengerjaan
- link invoice/manual note
- assign undangan ke customer

Status:

```text
pending
paid
processing
completed
cancelled
```

### User & Role Manager

Role awal:

```text
super_admin
admin
editor
support
customer
```

Fungsi:

- user admin tidak lagi pakai `ADMIN_USERNAME`/`ADMIN_PASSWORD` dari `.env`
- login admin/customer pakai tabel `users`
- role menentukan akses menu

## Database Inti

Tabel baru yang dibutuhkan:

```text
users
customers
themes
theme_assets
theme_placeholders
taxonomies
taxonomy_terms
media_assets
orders
invitations
invitation_media
settings
```

Relasi utama:

```text
users 1-1 customers
customers 1-n orders
customers 1-n invitations
themes 1-n invitations
themes 1-n theme_assets
themes 1-n theme_placeholders
taxonomies 1-n taxonomy_terms
media_assets n-1 users
media_assets n-1 invitations
```

## Urutan Implementasi

### Fase 1: Fondasi Platform

- ubah `/` menjadi landing katalog tema
- pindahkan undangan contoh ke `/undangan/elyana-syahril`
- buat layout admin dashboard dengan sidebar
- buat tabel `themes`, `media_assets`, `customers`, `orders`
- buat halaman admin dashboard
- buat seed awal tema dari config yang sudah ada

### Fase 2: Auth & Role

- ubah admin login dari `.env` ke database user
- tambah login/register customer
- tambah role sederhana
- customer wajib login untuk membuat undangan

### Fase 3: Theme & Taxonomy CMS

- CRUD tema
- CRUD taxonomy dan term
- CRUD placeholder tema
- upload preview/asset tema
- publish/archive tema

### Fase 4: Media Library

- upload media global
- upload media per undangan
- browser media di form undangan
- delete/replace/copy URL media

### Fase 5: Customer Invitation Builder

- customer pilih tema
- customer isi form undangan
- customer upload aset wajib
- preview sebelum publish
- link undangan aktif setelah publish/admin approval

### Fase 6: Order & Paket

- paket harga
- order manual
- status pembayaran
- assign undangan ke order
- dashboard status untuk pelanggan

## Catatan Teknis

- Jangan simpan password asli di repo.
- `.env` tetap untuk konfigurasi environment, bukan data user admin.
- File upload production tetap di `public/uploads` untuk kompatibilitas cPanel.
- Setelah platform stabil, pertimbangkan pindah storage ke S3-compatible object storage.
- Gunakan migration untuk semua data master baru, jangan config PHP untuk data yang harus diedit admin.
