# Pratama — Pilihan Tepat

Website jual beli berbagai produk Pratama. Frontend HTML/CSS/Vanilla JS, hosting GitHub Pages, data Google Spreadsheet melalui Apps Script, dan login admin menggunakan Firebase Authentication.

## Struktur folder

```text
zen-website/
├── index.html
├── admin.html
├── Code.gs
├── README.md
├── assets/
│   └── logo.png              # letakkan logo Pratama di sini
├── css/
│   └── styles.css
└── js/
    ├── config.js             # isi konfigurasi Firebase + Apps Script
    ├── app.js
    └── admin.js
```

## 1. Siapkan Google Spreadsheet

1. Buat Google Spreadsheet baru.
2. Salin Spreadsheet ID dari URL: `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`.
3. Buka Extensions → Apps Script.
4. Salin isi `Code.gs` ke editor Apps Script.
5. Isi `SPREADSHEET_ID` di bagian atas Code.gs.
6. Isi `FIREBASE_WEB_API_KEY` setelah membuat Firebase project (lihat bagian Firebase).
7. Jalankan fungsi `setupSpreadsheet()` satu kali dari editor Apps Script. Berikan izin yang diminta.

Script membuat tab:
- `Products`: id, name, category, price, image, description, active, updatedAt
- `Media`: id, title, type, url, active, updatedAt
- `Contacts`: id, label, type, value, isOrder, active, updatedAt
- `Settings`: key, value
- `Admins`: email, active

Masukkan email admin pada tab `Admins`, kolom A. Isi kolom B dengan `TRUE`. Email harus sama persis dengan email akun Firebase (huruf besar/kecil tidak berpengaruh).

### Deploy Apps Script sebagai API

1. Klik Deploy → New deployment.
2. Pilih type **Web app**.
3. Execute as: **Me**.
4. Who has access: **Anyone**.
5. Deploy, selesaikan otorisasi, lalu salin URL berakhiran `/exec`.
6. Setiap kali mengubah kode Apps Script, gunakan Deploy → Manage deployments → Edit → New version.

Catatan: endpoint publik hanya mengirim data katalog yang memang ditujukan untuk publik. Operasi tulis dan pembacaan dashboard admin memerlukan token Firebase valid serta email yang terdaftar di tab `Admins`.

## 2. Siapkan Firebase Authentication

1. Buka https://console.firebase.google.com/ lalu buat project.
2. Masuk Authentication → Get started → Sign-in method.
3. Aktifkan **Email/Password**.
4. Authentication → Users → Add user. Buat akun admin dari Firebase Console. Pendaftaran tidak tersedia di halaman publik.
5. Masuk Project settings → General → Your apps → Add app → Web app.
6. Salin konfigurasi Firebase.
7. Masukkan email akun tadi ke tab `Admins` di Spreadsheet dan set `active` menjadi `TRUE`.
8. Salin Firebase Web API Key ke `FIREBASE_WEB_API_KEY` di `Code.gs`.

Firebase Web API Key adalah konfigurasi aplikasi web, bukan private service-account key. Jangan pernah menaruh service account JSON atau private key di frontend.

## 3. Konfigurasi website

Buka `js/config.js` dan isi:

```js
window.ZEN_CONFIG = {
  API_URL: "https://script.google.com/macros/s/DEPLOYMENT_ID/exec",
  FIREBASE: {
    apiKey: "FIREBASE_WEB_API_KEY",
    authDomain: "PROJECT_ID.firebaseapp.com",
    projectId: "PROJECT_ID",
    appId: "FIREBASE_APP_ID"
  }
};
```

Gunakan konfigurasi Web App Firebase dari Firebase Console, bukan data rahasia server.

Letakkan logo final di `assets/logo.png`. Nama file harus persis `logo.png`.

## 4. Jalankan dan deploy GitHub Pages

1. Upload seluruh folder dan file ke repository GitHub.
2. Pastikan `index.html` berada di root repository.
3. Repository → Settings → Pages.
4. Source: Deploy from a branch.
5. Pilih branch `main`, folder `/ (root)`, lalu Save.
6. Buka URL GitHub Pages. Halaman publik ada di `/`, halaman admin di `/admin.html`.

Website tidak membutuhkan Node.js, npm, build step, atau server frontend.

## 5. Cara mengelola konten

### Produk
Isi nama, kategori, harga, link foto publik, deskripsi, dan status. Produk aktif akan muncul di halaman Produk. Tombol ORDER menggunakan kontak yang ditandai sebagai tujuan order. Jika tidak ada yang ditandai, sistem memilih WhatsApp aktif terlebih dahulu, lalu kontak aktif pertama.

### Kontak
Pilih tipe WhatsApp, Instagram, Facebook, Telepon, Email, Website, atau Lainnya. Untuk WhatsApp, masukkan nomor dengan kode negara, misalnya `6281234567890` (tanpa tanda `+`, spasi, atau strip). Untuk Instagram/Facebook/Website, masukkan URL lengkap. Tandai satu kontak sebagai tujuan order.

### Foto dan video
Semua media ditambahkan menggunakan URL, bukan upload file ke website.
- Foto: gunakan URL langsung menuju file gambar yang dapat diakses publik.
- YouTube: URL video biasa, Shorts, atau youtu.be.
- Instagram: URL post/reel publik.
- Facebook: URL video publik.
- Video file: URL langsung `.mp4`, `.webm`, atau `.ogg`.

Untuk media yang memerlukan login, dibatasi pemilik, atau tidak mengizinkan embed, browser mungkin tidak dapat menampilkan atau memutarnya. Jadikan file dan postingan publik serta izinkan embedding jika platform menyediakan pengaturan tersebut.

## 6. Performa dan cache

- Halaman publik membaca data melalui JSONP agar dapat mengambil data dari Apps Script lintas domain.
- Data terakhir disimpan di LocalStorage sehingga dapat langsung ditampilkan pada kunjungan berikutnya, kemudian disegarkan dari Spreadsheet.
- Apps Script menggunakan CacheService selama 45 detik untuk data publik.
- Perubahan admin menghapus cache server dan memperbarui cache browser setelah data dibaca ulang.

Catatan: Google Spreadsheet dan Apps Script tetap memiliki waktu respons jaringan. Karena itu tidak mungkin menjamin data selalu tampil tanpa loading sama sekali pada kunjungan pertama atau koneksi lambat. Cache membantu membuat kunjungan berikutnya terasa instan.

## 7. Batas dukungan format media

Tidak ada website browser yang dapat menjamin semua jenis file bisa dipreview atau diputar. Browser hanya dapat menampilkan format yang didukung dan URL yang dapat diakses tanpa autentikasi. Untuk hasil paling konsisten, gunakan JPG, PNG, WebP untuk gambar serta MP4 (H.264/AAC) untuk video langsung. Link platform sosial ditampilkan melalui embed jika platform mengizinkan.

## 8. Keamanan penting

- Jangan pernah menaruh service-account key, password, atau private key di repository GitHub.
- Aktifkan Firebase Email/Password dan buat akun melalui Firebase Console.
- Jangan berikan akses edit Spreadsheet kepada pengunjung website.
- Lindungi tab `Admins`; hanya pemilik yang boleh mengubah daftar admin.
- Apps Script Web App dapat diakses publik untuk membaca katalog, tetapi operasi admin tetap memverifikasi Firebase ID token dan daftar email admin.
- Web API Key Firebase dapat terlihat di frontend; batasi API key di Google Cloud sesuai kebutuhan dan jangan menganggap API key sebagai pengganti autentikasi.
- Sebelum dipakai produksi, uji login, tambah/edit/hapus, dan tampilan media dengan akun admin serta akun publik.

## 9. Troubleshooting

- **Data tidak muncul:** periksa URL `/exec`, izin deploy Apps Script, nama tab, dan jalankan `setupSpreadsheet()`.
- **Login berhasil tetapi akses ditolak:** pastikan email terdaftar di tab `Admins` dan `active` bernilai `TRUE`.
- **Perubahan tidak masuk:** pastikan Apps Script deployment memakai versi terbaru dan cek Executions di Apps Script.
- **Gambar tidak tampil:** gunakan direct image URL yang dapat dibuka di tab baru tanpa login.
- **Instagram/Facebook tidak tampil:** akun/post harus publik dan platform harus mengizinkan embed. Jika tidak, gunakan link langsung sebagai alternatif.
