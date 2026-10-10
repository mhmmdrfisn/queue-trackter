# Queue Tracker

Aplikasi web untuk **mendigitalisasi** observasi antrean **(Model M/M/s)** pada praktikum Teori Antrean.

Berfungsi ganda sebagai **tally counter lapangan** sekaligus **kalkulator analitis** yang
langsung menerapkan rumus M/M/s dari data yang Anda catat sendiri.

> Entirely client-side. Tidak ada server, tidak ada akun, tidak ada data yang dikirim ke mana pun.

---

## Fitur

| Modul | Fungsi |
|---|---|
| **Tracker** | Pencatatan kedatangan dan durasi pelayanan per server (kasir) |
| **Analysis** | Perhitungan otomatis parameter M/M/s + kesimpulan operasional |
| **Guide** | Petunjuk penggunaan lapangan |
| **Author** | Profil pengembang dan portofolio proyek |

Fitur pendukung:

- **Smart Queue Lock** — server tidak bisa memanggil pelanggan bila antrean kosong.
- **Auto-stop** — observasi berhenti otomatis saat durasi target habis.
- **Wake Lock** — layar tidak mati selama observasi berjalan.
- **Auto-save** — state tersimpan di `localStorage`, aman saat browser ditutup.
- **Presisi milidetik** — durasi pelayanan diukur dengan `Date.now()`, bukan tick timer
  1 detik, sehingga layanan singkat tidak ikut dibulatkan.
- **Riwayat kejadian** — setiap kedatangan dan setiap layanan tercatat lengkap
  (server mana, mulai, selesai, durasi), sehingga laporan bisa diaudit.

---

## Ekspor Laporan

Tombol di tab **Analysis** menghasilkan laporan yang siap dikumpulkan.
Seluruh angka diturunkan dari satu fungsi `buildReport()`, sehingga yang tampil
di layar dan yang ada di file Excel **selalu identik**.

| Tombol | Keluaran |
|---|---|
| **Excel** | `.xls` (SpreadsheetML) berisi 5 sheet: Ringkasan, Hasil M-M-s, Per Server, Log Layanan, Log Kedatangan |

### Isi laporan

```
A. IDENTITAS OBSERVASI   lokasi, pengamat, tanggal, waktu mulai/selesai,
                         durasi target & aktual, jumlah server, cara berhenti
B. DATA OBSERVASI        total kedatangan, total dilayani, belum dilayani,
                         rasio pelayanan, rata-rata durasi, rata-rata jarak kedatangan
C. RINCIAN PER SERVER    jumlah dilayani, total/rata-rata/min/max durasi, utilisitas
D. PARAMETER SISTEM      λ, μ, ρ, kapasitas total, selisih kapasitas
E. HASIL M/M/s           P₀, Lq, Ls, Wq, Ws (dalam jam & menit)
F. RUMUS YANG DIGUNAKAN   daftar rumus sehingga angka bisa ditelusuri
G. LOG KEJADIAN          rekap tiap layanan: server, mulai, selesai, durasi
```

Kenapa ini berguna: sheet **Log Layanan** berisi data mentah yang bisa dihitung
ulang dengan kalkulator biasa, sehingga hasil aplikasi bisa **dipertanggungjawabkan**
saat ditanya dosen.

---

## Cara Pakai

1. Buka `index.html` di browser (Chrome/Edge/Firefox, desktop atau HP).
2. Tab **Tracker** → isi **Object Location** dan **Observer Name**, atur **Date**,
   atur **Duration**, tentukan jumlah **Servers (s)**.
3. Tekan **START RECORDING**.
4. Saat pengamatan:
   - Pelanggan datang → tekan **+1 TO QUEUE**.
   - Kasir kosong memanggil pelanggan → tekan tombol **FREE** pada kasir tersebut
     (tombol berubah jadi **BUSY**, angka antrean berkurang 1).
   - Pelanggan selesai dilayani → tekan lagi tombol **BUSY** (kembali jadi **FREE**,
     durasi pelayanan tercatat).
5. Observasi berhenti otomatis saat waktu habis, atau tekan **STOP RECORDING**.
6. Buka tab **Analysis** untuk melihat hasil perhitungan.

---

## Rumus yang Digunakan

Besaran yang dihitung:

- **λ (arrival rate)** = jumlah kedatangan ÷ waktu observasi (jam)
- **μ (service rate)** = 3600 ÷ rata-rata durasi pelayanan (detik per pelanggan per server)

Dengan `ρ = λ / (s·μ)` sebagai tingkat utilisasi:

```
P₀ = 1 / [ Σ(n=0…s−1) (λ/μ)ⁿ/n!  +  (λ/μ)ˢ / (s!·(1−ρ)) ]

Lq = P₀ · (λ/μ)ˢ · ρ  /  [ s! · (1−ρ)² ]

Ls = Lq + λ/μ
Wq = Lq / λ
Ws = Wq + 1/μ
```

PARAMetrik dari kode di `calculateMMS()`.

### ❗ Catatan penting tentang akurasi

Model **M/M/s** adalah model **steady-state** dengan asumsi:

- Kedatangan bersifat **Poisson** (acak),
- durasi pelayanan **eksponensial**,
- jumlah server **tetap**.

Observasi lapangan manual **tidak selalu** memenuhi asumsi tersebut, sehingga angka M/M/s
bersifat **estimasi**, bukan kebenaran absolut. Selain itu:

- Jika **ρ ≥ 1**, sistem **tidak stabil** (antrean tumbuh tanpa batas) — aplikasi
  menampilkan peringatan **System Overload** dan sengaja tidak menampilkan P₀, Lq, Ls, Wq, Ws
  karena nilainya tidak bermakna.
- Layanan yang masih berjalan saat observasi berakhir otomatis **tidak dihitung**
  (durasi terpotong akan membuat μ terlalu besar).

Durasi pelayanan diukur memakai **timestamp milidetik** (`Date.now()`), bukan
tick timer 1 detik, sehingga layanan singkat tidak ikut dibulatkan ke detik
penuh. Hal ini berdampak langsung pada presisi μ.

---

## Struktur

```
Queue Tracker/
├── index.html      # seluruh aplikasi (HTML + CSS + JS, tanpa build step)
├── README.md
├── LICENSE
└── .gitignore
```

Aplikasi ini **sengaja dibuat dalam satu file** agar bisa langsung dibuka/dibagikan tanpa
build, tanpa server, dan tanpa dependency yang perlu di-install.

### Teknologi

- HTML5
- Tailwind CSS (CDN)
- Lucide Icons (CDN)
- Vanilla JavaScript

> Karena memakai CDN, aplikasi memerlukan koneksi internet saat pertama kali dibuka
> untuk memuat Tailwind dan Lucide.

---

## Pengujian

Perhitungan M/M/s diverifikasi terhadap implementasi referensi independen
(metode *log-space* untuk menghindari overflow pada `s` besar) dan terhadap
nilai baku buku teks (λ=3, μ=2, s=2 → Lq=1.9286, Ls=3.4286, Wq=0.6429, Ws=1.1429).

Hasil: **seluruh skenario cocok** (selisih < 1e−4 pada nilai yang ditampilkan, dan
< 1e−13 pada presisi penuh). Deteksi kondisi overload (ρ ≥ 1) juga terverifikasi.

Pengujian fungsional juga menutup jalur di mana data bisa rusak: filter durasi 0
dan tidak-finite, migrasi state versi lama, proteksi antrean kosong, dan konsistensi
antar-format (angka pada tampilan = angka pada file Excel).

---

## Lisensi

[MIT](LICENSE)

Copyright (c) 2026 Rafi Sanjaya