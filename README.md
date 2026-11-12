# Tertiup

Dua halaman, dua adegan, dirender dengan three.js, tanpa build step:

| Berkas | Adegan |
| --- | --- |
| `index.html` | kanopi dahan berdaun tertiup angin sore, latar dirender penuh |
| `daun-terbang.html` | foto `background1.jpg` sebagai latar diam, di depannya beberapa daun kecil melayang tertiup angin |

## Menjalankan

Buka `index.html` langsung di browser (klik dua kali). Yang dibutuhkan hanya koneksi
internet saat pertama kali, karena three.js diambil dari CDN.

`daun-terbang.html` juga begitu — termasuk gambarnya. JPEG-nya ditanam di dalam berkas
sebagai base64 (`.jpg` 546 KB jadi ±727 KB teks), bukan dirujuk lewat `<img src>`. Itu
sengaja: kalau dirujuk, Chrome memblokir tekstur dari `file://` karena CORS dan halamannya
gagal diam-diam. Harga yang dibayar adalah satu baris raksasa di baris ke-71 — jangan
disunting baris itu; sunting kode di atas dan di bawahnya saja.

Kalau lebih suka memakai server, jalankan `python -m http.server 8000` di folder ini lalu
buka `http://localhost:8000/daun-terbang.html`; versi itu boleh memakai `background1.jpg`
sebagai tekstur biasa.

Kalau ingin tanpa internet: unduh `three.min.js` r155, taruh di sebelah `index.html`,
lalu ganti tag `<script src="https://...">` menjadi `<script src="three.min.js">`.

## Yang ada di dalamnya

- **Angin** — gerak tidak diatur per daun, tapi lewat medan gelombang yang menjalar
  melewati ruang. Daun yang berdekatan bergerak serempak, jadi kanopi berayun sebagai
  satu massa, bukan serakan objek. Tiap daun menambah kibasan sendiri di ujungnya.
- **Daun** — bentuk, urat, dan cupingnya digambar di shader (bukan tekstur), jadi tetap
  tajam di segala ukuran. Pencahayaan dua sisi: sisi yang menghadap matahari terang,
  sisi sebaliknya meneruskan cahaya.
- **Oklusi** — saat dibuat, tiap daun dihitung kerapatan tetangganya. Daun di dalam
  massa jadi lebih gelap, yang di permukaan tetap terang. Ini yang memberi kanopi bentuk.
- **Aksen sinematik** — letterbox 2.39:1, bloom, aberasi kromatik di tepi bingkai,
  vignette, grain, grading teal-orange, berkas cahaya, serbuk terbang, dan kamera yang
  bergerak pelan dengan goyangan tangan operator.
- **Interaksi** — gerakkan kursor untuk parallax; tahan klik untuk satu hembusan.
  Pengguna dengan `prefers-reduced-motion` otomatis mendapat gerak jauh lebih tenang.

## Kalau ingin disetel

Semua di dalam `index.html`:

| Cari | Artinya |
| --- | --- |
| `sunDir` | arah matahari (juga arah berkas cahaya) |
| `fogDensity` / `fogColor` | kedalaman kabut |
| `uDeep`, `uMidC`, `uLightC`, `uBackC` | warna daun: bayangan, tubuh, sisi terang, cahaya tembus |
| `addCluster({ ... })` | jumlah daun, ukuran, dan sebaran tiap gugus |
| `windNow` di `animate()` | kekuatan angin dasar dan hembusannya |
| `uExposure`, `uBloomStrength`, `uGrain` | rasa akhir di pass komposit |

Jumlah daun saat ini ±1.400. Menaikkannya membuat kanopi lebih padat, tapi di atas
±4.000 daun teksturnya mulai menyatu jadi mosaik keruh — saat itu terjadi, kecilkan
`size` atau besarkan `radii` gugusnya, jangan menambah jumlah.

## Daun Terbang: latar foto, tiga hal yang bisa disetel

Adegan ini lebih sederhana dari `index.html`: satu kamera diam, satu lapisan daun, satu
pass komposit. Fotonya tidak ikut bergerak — ia hanya ditempel ke bidang penuh dengan
*cover fit* (diperbesar sampai menutup layar, sisi lebih dipangkas rata tengah), jadi
tidak pernah ada garis kosong di tepi berapa pun rasio jendelanya.

- **Ganti fotonya** — timpa `background1.jpg`, lalu perbarui dua hal: blok base64 di
  baris 71, dan `IMG_W`/`IMG_H` (lebar dan tinggi foto, dalam piksel) di baris 112. Kalau lupa angkanya, gambarnya terpotong dengan hitungan rasio yang salah.
- **Jumlah daun** — `COUNT` (baris 325): 68 di layar sempit, 104 di layar lebar. Jumlah
  segitu sengaja sedikit; tujuannya beberapa lembar yang terbaca satu per satu, bukan
  hujan daun. Ukurannya diatur lewat `frac` (baris 345), dan hanya sekitar 7,5% daun yang
  diberi warna hangat (`Math.random() < 0.075`).
- **Rasa akhir** — `uExposure` (0.74), `uContrast` (0.07), `uVig` (0.26), `uGlow` (0.24),
  semuanya di sekitar baris 438.
  Angkanya sudah diturunkan dari nilai awal yang lebih agresif karena kurva ACES
  menaikkan midtone cukup jauh: pada 1.02 langitnya jadi ±+25 lebih terang daripada
  fotonya. Dengan 0.74, sampel langit terang kembali dalam ±5 dari aslinya, sementara
  bayangan bawah tetap pekat.

Foto aslinya rata-rata luma 95; hasil akhirnya 88. Selisih itu bukan kesalahan —
itu toe kurva film yang dipakai bersama dengan `index.html`.



Post-processing ditulis sendiri (bright pass → blur terpisah → komposit) supaya tidak
perlu addon. Scene dirender ke render target half-float dengan MSAA 4x, lalu di-tonemap
dan di-encode sRGB secara manual di pass terakhir. Kalau frame time melebihi ~26 ms,
pixel ratio diturunkan sekali secara otomatis.
