# Daun Terbang sebagai hero

Catatan pemasangan `daun-terbang.html` sebagai latar belakang hero di situs lain.
Isinya bukan teori: semua angka di sini sudah terukur dari berkas di folder ini.

## Kenapa cocok

Adegannya sudah berbentuk seperti hero yang baik — kameranya diam, latarnya foto statis,
tidak ada objek yang masuk-keluar bingkai, dan *cover fit* di shader menangani rasio apa pun
tanpa pernah menyisakan tepi kosong. Tidak ada hal yang perlu ditambal di sisi WebGL.

## Yang harus diubah

### 1. Canvas jadi anak dari section, bukan `fixed`

Sekarang `#stage` menempel ke seluruh jendela. Di hero ia harus dibatasi oleh section
bertinggi tetap, dengan konten di atasnya.

```html
<section class="hero">
  <canvas id="stage" aria-hidden="true"></canvas>
  <div class="hero__scrim"></div>
  <div class="hero__copy">
    <h1>Judul</h1>
    <p>Subjudul</p>
  </div>
</section>
```

```css
.hero {
  position: relative;
  height: min(78svh, 760px);
  overflow: hidden;
  background: #05080a;
}
.hero #stage { position: absolute; inset: 0; width: 100%; height: 100%; display: block; }
.hero__scrim {
  position: absolute; inset: 0; pointer-events: none;
  background: linear-gradient(90deg, rgba(5, 8, 10, .62) 0%, rgba(5, 8, 10, 0) 62%);
}
.hero__copy { position: relative; z-index: 2; }
```

Pakai `svh`/`dvh`, jangan `vh` — di iOS Safari `100vh` ikut menghitung bilah alamat
yang bisa menyusut saat digulir, jadi hero melompat.

Canvas diberi `aria-hidden="true"` karena murni dekoratif, dan teks hero tetap ditulis
sebagai DOM asli — bukan dibakar ke gambar — supaya terbaca mesin pencari dan pembaca layar.

### 2. Jangan inline base64

Blob 727 KB di baris 71 hanya perlu supaya berkas bisa dibuka lewat klik dua kali
(`file://` memblokir tekstur dari URL lain karena CORS). Di situs asli, rujuk
`background1.jpg` sebagai URL biasa:

```js
new THREE.TextureLoader().load('/img/background1.jpg', onReady, undefined, onFail);
```

546 KB, dan browser bisa meng-cache-nya. Kalau diubah ke WebP, turun ke sekitar 150–200 KB.
Ini penghematan terbesar yang tersedia, dan tidak mengubah satu baris pun shader.

Kalau fotonya diganti, ikut ubah `IMG_W`/`IMG_H` di baris 112 (lebar dan tinggi foto dalam
piksel) — kalau tidak, *cover fit*-nya memotong dengan hitungan rasio yang salah.

### 3. Hentikan render saat tidak terlihat

Ini yang membedakan hero sopan dari hero yang menguras baterai. Tanpa ini, adegan 60 fps
terus jalan di halaman yang sudah digulir jauh ke bawah.

```js
var running = true;

new IntersectionObserver(function (entries) {
  running = entries[0].isIntersecting && !document.hidden;
}, { threshold: 0 }).observe(document.querySelector('.hero'));

document.addEventListener('visibilitychange', function () {
  running = !document.hidden;
});

function loop(now) {
  requestAnimationFrame(loop);
  if (!running) return;      // jam berhenti, tidak ada dt yang menumpuk
  // ...
}
```

Penting: kalau render dilewati, **jangan** menambah `uTime`. Kalau jam tetap jalan,
daun akan melompat jauh begitu hero terlihat lagi.

### 4. Batasi DPR dan matikan MSAA di layar padat

```js
var DPR = Math.min(window.devicePixelRatio || 1, 1.75);
var samples = DPR > 1 ? 0 : 4;
```

Lapisan daunnya tipis dan ber-alpha; di layar DPR 2 tanpa MSAA pun tidak terlihat bergerigi,
sementara 4 sampel di resolusi 2× menggandakan biaya piksel empat kali lipat.

### 5. Hormati `prefers-reduced-motion`

Bekukan daun di satu frame — jangan hapus gambarnya. Fotonya tetap indah; yang dihilangkan
hanya geraknya.

```js
var REDUCED = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
// di dalam loop: kalau REDUCED, jangan tambah uTime dan uTravel
```

### 6. Scrim di area teks

Fotonya terang di langit kiri-atas, jadi judul di kiri akan tenggelam. Gradien CSS
(lihat `.hero__scrim` di atas) lebih murah dan lebih mudah disetel per breakpoint
daripada mengubah vignette di shader. Vignette yang ada sekarang (`uVig` 0.26) sudah
dipakai untuk membingkai sudut; jangan ditumpuk dengan scrim penuh.

## Yang perlu ditimbang jujur

three.js UMD ±600 KB plus JPEG ±550 KB, untuk sesuatu yang dekoratif. Kalau situsnya
sudah memuat three.js untuk hal lain, ini gratis — modulnya sudah ada. Kalau tidak,
1,1 MB demi latar belakang adalah harga mahal.

46 daun ber-alpha sebenarnya juga bisa dikerjakan dengan Canvas2D dan skrip jauh lebih
kecil; yang hilang hanya tumble 3D-nya. Jadi keputusannya bukan soal "bisa atau tidak",
tapi soal apakah situs ini memang sudah punya three.js.

## Daftar periksa sebelum tayang

- [ ] Canvas dibatasi section, bukan `fixed`
- [ ] Foto dirujuk lewat URL, base64 dibuang
- [ ] `IMG_W`/`IMG_H` cocok dengan foto yang dipakai
- [ ] Render berhenti saat hero di luar viewport dan saat tab tidak aktif
- [ ] DPR dibatasi, MSAA mati kalau DPR > 1
- [ ] `prefers-reduced-motion` membekukan gerak
- [ ] Teks hero DOM asli, canvas `aria-hidden`
- [ ] Scrim cukup untuk keterbacaan di atas langit terang
- [ ] Diuji di 430×932 (potret) dan 1600×900 (lebar)
