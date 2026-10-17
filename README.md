# SIP Karangnongko — Sistem Informasi Peternakan Kambing

Sistem administrasi peternakan kambing berbasis **Laravel 12** dan **Filament 3**, dikembangkan dalam konteks kegiatan **Kuliah Kerja Nyata (KKN)** di Desa Karangnongko, Klaten, Jawa Tengah.

Proyek ini berfokus pada pencatatan data kambing per kandang, absensi/pengecekan harian, dan tampilan laporan operasional untuk pengelola peternakan.

> **Status: proyek KKN / aplikasi administrasi tahap pengembangan.** Repository memiliki model, migrasi, panel Filament, halaman dashboard, serta laporan; namun keamanan akun, cakupan otorisasi, pengujian, dan kesiapan deployment produksi **belum dapat dinyatakan tervalidasi**. Jangan gunakan data atau akun asli sebelum isu pada bagian [Keamanan dan keterbatasan](#keamanan-dan-keterbatasan) ditangani.

## Ringkasan implementasi

| Modul | Implementasi dalam kode | Batasan |
| --- | --- | --- |
| Data peternakan | Model `Farm` dengan nama dan lokasi; seeder menyiapkan kandang Barat dan Timur | Belum ditemukan antarmuka CRUD khusus `FarmResource` |
| Data kambing | `GoatResource`: daftar, pencarian, filter, tambah, ubah, dan hapus; tag dibuat oleh model | Generator nomor tag perlu pengujian konkurensi; filter role tidak menggantikan policy otorisasi |
| Pengecekan harian | `DailyCheckResource`: tanggal, pengguna yang mengecek, catatan, dan operasi CRUD | Tidak merekam diagnosis kesehatan per kambing; relasi banyak-ke-banyak kambing pada model masih dikomentari |
| Dashboard | Jumlah kambing, status pengecekan hari ini, ringkasan 7 hari, dan kambing terbaru | Nilai laporan sakit bergantung pada kolom opsional; tidak membuktikan pemantauan kesehatan real-time |
| Laporan global | Halaman `LaporanGlobalPage` dengan widget tabel kambing dan absensi | Bukan bukti sistem analitik atau ekspor laporan menyeluruh |
| Pengguna | Model `User` menyimpan role dan relasi kandang; panel login Filament di `/admin` | Akses panel belum dibatasi berdasarkan role |

### Alur data

~~~text
Peternakan (Farm)
  ├── Pengguna (User: admin / barat / timur)
  └── Kambing (Goat: tag, jenis, kelamin, tanggal lahir, berat)
Pengguna
  └── Catatan pengecekan (DailyCheck: tanggal, catatan)
            ↓
      Dashboard dan laporan Filament
~~~

Pengguna non-admin difilter berdasarkan kandang pada kueri daftar kambing, dan berdasarkan pembuat catatan pada daftar pengecekan harian. Ini merupakan bagian dari implementasi filter data, **bukan jaminan kontrol akses menyeluruh** untuk semua aksi.

## Teknologi

| Lapisan | Teknologi |
| --- | --- |
| Framework | PHP 8.2+, Laravel 12 |
| Admin UI | Filament 3.3, Blade/Livewire melalui Filament |
| Database | Laravel Eloquent, migrations; `.env.example` memakai SQLite sebagai default |
| Frontend assets | Vite 6, Tailwind CSS 4 |
| Quality tools | Pest 3, Laravel Pint |

## Menjalankan secara lokal

### Prasyarat

- PHP **8.2+** dengan ekstensi yang dibutuhkan Laravel
- Composer
- Node.js dan npm
- SQLite (default) atau database lain yang dikonfigurasi sendiri

~~~bash
git clone https://github.com/alfrzhb/sip_karangnongko.git
cd sip_karangnongko

composer install
npm install

cp .env.example .env
php artisan key:generate

# Untuk DB_CONNECTION=sqlite, siapkan file database bila belum ada.
# macOS/Linux:
mkdir -p database
touch database/database.sqlite

php artisan migrate
npm run build
php artisan serve
~~~

Untuk pengembangan antarmuka dengan hot reload, jalankan `npm run dev` pada terminal terpisah. Di Windows, salin `.env.example` ke `.env` dan buat file `database/database.sqlite` secara manual atau melalui PowerShell. Pengaturan koneksi bisa disesuaikan di `.env`.

Setelah pengaturan dan migrasi berhasil, panel autentikasi Filament dikonfigurasi pada:

- `http://127.0.0.1:8000/admin` — login panel administrasi
- `http://127.0.0.1:8000/` — halaman welcome Laravel; **bukan** halaman dashboard peternakan

### Data contoh (hanya untuk lingkungan lokal)

Seeder `database/seeders/UserSeeder.php` menyiapkan kandang Barat dan Timur serta akun demonstrasi. **Perhatian:** kode seeder memuat password bawaan yang mudah ditebak. Jalankan `php artisan db:seed` hanya dalam lingkungan pengembangan terisolasi; jangan gunakan akun, email contoh, atau password tersebut untuk layanan yang dapat diakses publik.

### Perintah verifikasi

~~~bash
php artisan test
npm run build
~~~

Perintah di atas tersedia dalam proyek, tetapi keberhasilan eksekusi dan cakupan pengujian fungsional **tidak diklaim** oleh README ini.

## Struktur direktori penting

~~~text
app/
  Filament/
    Pages/                 # Dashboard dan laporan global
    Resources/             # CRUD kambing dan catatan harian
  Models/                  # Farm, Goat, DailyCheck, User
  Providers/Filament/      # Konfigurasi panel /admin
database/
  migrations/              # Skema dan perubahan skema
  seeders/                 # Data farm dan akun demo
resources/                 # Blade views / aset UI
routes/web.php             # Route halaman welcome
tests/                     # Pengujian dasar
~~~

## Keamanan dan keterbatasan

1. **Kredensial demo:** seeder menyimpan password statis. Hapus atau ganti mekanisme ini sebelum deployment dan rotasi kredensial untuk lingkungan yang pernah memakai akun contoh.
2. **Otorisasi panel:** `User::canAccessPanel()` saat ini mengembalikan `true` untuk semua pengguna yang diautentikasi. Pembatasan pada kueri/form belum setara dengan policy per-record atau pemeriksaan izin setiap aksi.
3. **Pengecekan harian bukan rekam medis ternak:** catatan masih berorientasi tanggal, pencatat, dan keterangan umum. Jangan menyebutnya pelacakan penyakit setiap kambing secara lengkap.
4. **Penomoran tag:** model membuat nomor urut berdasarkan record sebelumnya; kemungkinan tabrakan pada operasi serentak perlu diuji dan ditangani.
5. **Kualitas produksi:** belum ada bukti pengujian end-to-end, peninjauan keamanan, backup/restore, maupun verifikasi deployment produksi dalam audit README ini.

## Konteks proyek terkait

SIP Karangnongko adalah implementasi **backend/admin** berbasis Laravel/Filament. Prototype frontend dan landing page yang memiliki nama serupa disimpan secara terpisah:

- [karangnongko-goat-keeper](https://github.com/alfrzhb/karangnongko-goat-keeper) — frontend manajemen berbasis React; backend tidak termasuk di repo tersebut
- [karangnongko-farm](https://github.com/alfrzhb/karangnongko-farm) — landing page statis
- [LP-Karangnongko](https://github.com/alfrzhb/LP-Karangnongko) — eksplorasi landing page lain

Repository-repository tersebut **tidak otomatis terintegrasi** satu sama lain.

## Lisensi

Berkas [LICENSE](LICENSE) tersedia dalam repository.

---

Dikembangkan sebagai proyek pembelajaran dan digitalisasi peternakan dalam kegiatan KKN.