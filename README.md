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
- **Ekspor CSV** — unduh laporan beserta rekap seluruh durasi pelayanan.

---

## Cara Pakai

1. Buka `index.html` di browser (Chrome/Edge/Firefox, desktop atau HP).
2. Tab **Tracker** → isi **Object Location**, atur **Duration**, tentukan jumlah **Servers (s)**.
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

---

## Lisensi

[MIT](LICENSE) © Rafi Sanjaya