# Space RPG 4 v0.9995 — Mapping Tahap 2: Hardpoint, sektor tembak & rancangan 3D

**Tanggal:** 23 Oktober 2026  
**Status:** pemeriksaan statis source asli (`smali.zip`), BELUM diimplementasikan ke Godot. Prioritas: hardpoint dengan laras terhalang lambung dan sektor tembak terbatas, sebagai dasar sistem AUTO ENGAGE.

## Ringkasan hasil terverifikasi

- Membaca `h1.smali` secara langsung dan melacak array **`Lu7;->X:[F`** pada **98 factory** yang memberi geometri mount. **92** mempunyai ID kapal statis; **6** merupakan factory berparameter (ID final harus ditentukan sesuai argumen runtime).
- Terekstraksi **606 kelompok enam angka**, mencakup slot senjata maupun kemungkinan slot khusus. **Jangan menyebut semuanya turret aktif.** Distribusi tag mentah pertama: 0: 234, 1: 188, 2: 107, 4: 70, 5: 7. Terdapat **7** factory dengan 0 kelompok.
- Dataset ukuran kapal 92 entri sebelumnya dicocokkan silang: **92/92 nama, ID, panjang, dan lebar konsisten** dengan hasil ekstraksi ini.
- **Penting: ukuran/arc di sini adalah geometri 2D** dan belum memberi tinggi/deck, halangan lambung, elevasi senjata, maupun izin 3D. Kolom 3D harus dirancang per model GLB.

## Susunan satu hardpoint original

Di `h1.smali` setiap kapal membuat `Lu7.X` berupa array float. `f8.smali` dalam `h([I)V` mengaksesnya **dengan stride 6** melalui `v16 += 6`, menggunakan `X[base+0]` untuk tag/jenis, `X[base+1]` dan `[+2]` untuk koordinat mount, serta `[+3..5]` sebagai parameter sudut saat membangun `Lzj`/controller turunannya. `Lv7.b` mendefinisikan pasangan konfigurasi senjata bawaan, sementara registry senjata berada di `Lh1.H4`.

| Offset | Interpretasi untuk mapping | Dasar/kepastian |
|---|---|---|
| 0 | Tag jenis slot mentah (0,1,2,4,5 di template yang terbaca) | Langsung dari pola `f8` dan `h1`; penamaan kategori 4/5 masih perlu konfirmasi |
| 1 | Offset mount x dalam ruang 2D kapal | Diteruskan ke koordinat controller `Lzj.e` |
| 2 | Offset mount y dalam ruang 2D kapal | Diteruskan ke koordinat controller `Lzj.f` |
| 3 | Orientasi awal/arah visual mounting 2D | Diteruskan ke `Lzj.q` |
| 4 | Arah tengah sektor bidik relatif kapal | Diteruskan ke `Lzj.o`, dipakai pada batas putaran di `zj` |
| 5 | Lebar sektor bidik dalam derajat | Diteruskan ke `Lzj.p`, dibagi dua untuk batas kiri/kanan |

**Contoh nyata dari original:** O.S.C. Cruiser (`Lh1;->A1()`), dua mount pertamanya adalah `[0, 33, -50, -180, -120, 180]` dan `[0, -33, -50, 180, 120, 180]`. Masing-masing lebar sektor **180°**, bukan omnidirectional; sisi kiri/kanan saling bercermin. Sektor lain ada yang 60°, 90°, 120°, 150°, 210°, 240°, 330°, dan 360°. Tidak semua hardpoint setara.

> Kesimpulan desain: usulanmu mengenai turret samping dengan jangkauan putar hanya sekitar 180° ternyata *sejalan dengan data hardpoint asli*. Akan tetapi menyalin batas sudut 2D langsung sebagai `yaw` 3D saja tidak cukup; masih ada batas `pitch` dan fisik lambung.

## Contoh daftar kapal dari ekstraksi

| ID | Nama | Ukuran 2D panjang×lebar | Kelompok X/6 | Jumlah tag mentah | Factory |
|---:|---|---:|---:|---|---|
| 71 | Kynigos | 64×32 | 2 | 0:1,1:1 | `Q1` |
| 72 | Boreas | 150×60 | 9 | 0:3,1:2,2:2,4:2 | `N1` |
| 73 | Kharon | 250×100 | 15 | 0:4,1:5,2:2,4:4 | `J1` |
| 74 | Ares | 350×210 | 26 | 0:7,1:9,2:7,4:2,5:1 | `R1` |
| 94 | Hermes | 410×178 | 21 | 0:6,1:7,2:5,4:2,5:1 | `S1` |
| 33 | F.E.R. Battleship | 240×100 | 15 | 0:4,1:4,2:5,4:2 | `h0` |
| 34 | F.E.R. Cruiser | 150×60 | 12 | 0:3,1:5,2:2,4:2 | `n0` |
| 35 | F.E.R. Destroyer | 110×40 | 7 | 0:4,1:2,4:1 | `o0` |
| 36 | F.E.R. Frigate | 80×30 | 3 | 0:2,1:1 | `s0` |
| 40 | O.S.C. Cruiser | 200×100 | 10 | 0:3,1:3,2:3,4:1 | `A1` |
| 55 | Iro Titan | 380×200 | 22 | 0:8,1:8,2:5,5:1 | `s1` |
| 64 | Void Titan | 380×180 | 25 | 0:6,1:12,2:5,4:2 | `Q2` |

**Catatan khusus:** ID **71 Kynigos** memiliki 2 kelompok, **72 Boreas** 9, **73 Kharon** 15, **74 Ares** 26, **94 Hermes** 21, **33 F.E.R. Battleship** 15, dan **55 Iro Titan** 22. Kategori tag 4/5 **belum boleh diasumsikan sebagai turret**, karena bisa mencakup mount khusus/sistem lain. Ada pula definisi khusus semacam Homebase, Runner, dan tanker yang mempunyai `X` kosong; itu tidak berarti otomatis dapat dipasang senjata.

## Temuan terhadap mekanisme menembak asli

1. `f8.smali` mengambil daftar ID senjata yang terpasang dari `Lv7.b` (untuk varian template saat `p1 == null`) dan melewati ID `-1` sebagai slot kosong. Jadi **jumlah slot terdefinisi ≠ jumlah senjata aktif**.
2. Jenis weapon yang terpasang dicari di `Lh1.H4:[Lak;`, lalu controller `zj`/subclass dibuat dengan posisi dan parameter sektor dari `X`.
3. `zj.smali` membatasi sudut turret dengan normalisasi sudut dan setengah lebar sektor (lihat penggunaan `Lzj.o`, `Lzj.p`). Ini memberi landasan limit *yaw*.
4. `zj.smali` juga memakai status Targeting AI di `Lw7.P:Z`; pada satu cabang waktu antisipasi gerakan dikurangi menjadi seperempat saat AI tidak ada. Itu **perbaikan prediksi**, bukan garantasi kena.
5. Runtime jangkauan berasal dari definisi keluarga senjata, bukan dari besar sudut mount: lihat `q6.u`, `ei.K`, `q4.l`, dan pemakaian di `f8`/controller senjata. Satuan angka final per ID dan perilaku kloning upgrade masih **belum diekstrak secara lengkap**.
6. `Lu7.X` juga ditulis di **`s8.smali`** untuk beberapa template/varian khusus; maka tabel di sini memuat **template `h1` mentah**, belum selalu *state final* setelah tahap bootstrap, upgrade, atau varian dinamis.

## Aturan desain 3D yang direkomendasikan

### A. Mount menyimpan posisi dan sektor bidik fisik

Kita tambahkan `Marker3D` untuk *base turret* dan `Marker3D` di ujung laras (*muzzle*), masing-masing terikat pada model kapal. Untuk 3D penuh, mount perlu dua limit mandiri:
- **Yaw** (kiri-kanan): contoh side turret kiri: tengah –90° dan setengah sektor 90° → rentang 180°. Bukan selalu 180°: mengikuti konfigurasi mount yang dipilih.
- **Pitch** (atas-bawah): misalnya –15° sampai +65° untuk turret yang dipasang di atas lambung (angka **usulan**, bukan hasil ekstraksi original).

Tidak semua senjata otomatis berputar: fixed cannon harus mengikuti arah kapal; missile launcher memiliki aturan peluncuran sendiri; beam harus punya line-of-sight.

### B. Lambung kapal **bisa menghalangi peluru sendiri**

Saat target di sektor yaw/pitch yang diizinkan, belum tentu laras memiliki garis tembak bersih.

**Urutan periksa:** target valid → senjata terpasang → target dalam **jangkauan efektif/maksimum setelah upgrade** → sudut turret valid → posisi laras aman → **raycast collision ke lambung sendiri dan obstacle** → cooldown, amunisi/energi → tembak.

Untuk merancang raycast Godot: jalankan dari muzzle menuju titik intersepsi, **sertakan collision layer bagian lambung sendiri** ketika menguji occlusion. Jangan mengecualikan seluruh `ship` dari raycast, karena itu justru akan membuat turret menembus lambung. Collision turret/muzzle sendiri boleh dikecualikan. Tempatkan marker laras di luar volume armor agar tidak selalu menabrak collision milik sendiri.

Jika tembakan tertutup oleh superstructure atau sayap: **tahan tembakan** dan pilih turret lain; **jangan** menembus badan kapal dan jangan membuat proyektil muncul dari hidung kapal tanpa hardpoint. Beban fisika dapat dikelola dengan raycast per peluang tembak, bukan memeriksa semua ribuan senjata setiap frame.

### C. AUTO ENGAGE berlaku **per mount**, bukan per kapal saja

- Musuh terkunci tetapi di luar range semua senjata → `APPROACH` dapat mendekat, **tidak menembak**.
- Masuk range missile namun belum range cannon → hanya missile yang layak menembak.
- Target masuk range cannon tapi tidak berada dalam sektor turret samping / tertutup badan kapal → turret tersebut **tidak menembak**.
- AUTO ENGAGE mempertimbangkan senjata yang sanggup mengenai target untuk memilih jarak dan arah hadap; perintah ORBIT/KEEP RANGE tetap bisa di-cancel.
- Targeting AI meningkatkan kualitas prediksi intersepsi, bukan sektor putaran, kemampuan pitch, kecepatan putar mekanis, atau range maksimum.
- Beam/hitscan berbeda dari proyektil ballistic; hit tetap bergantung orientasi turret dan collision. Server multiplayer nantinya tetap menjadi penentu hasil serangan.

### D. Skala model 3D dan titik mount

Gunakan **satu world scale** untuk panjang/lebar dari source (`Lu7.n/o`). Konversi contoh: 1 unit 2D → 0,25 unit Godot, **belum angka final**. Proyeksikan `(x_2d, y_2d)` menjadi `(X, Z)` Godot sesuai orientasi model kapal, lalu **tetapkan ketinggian mount Y secara manual sesuai permukaan model**, jangan sembarang `Y=0` karena laras bisa terbenam di lambung.

Empat model Astra yang sudah kita gunakan **tidak otomatis mempunyai hardpoint yang cocok** dengan semua kapal original. Kita perlu daftar mount per desain, termasuk sayap, deck atas/bawah, barbettes, dan posisi missile bay. Keterangan/kategori mount akan muncul di Shipyard UI tanpa perlu membuat bulatan logam menonjol di seluruh lambung.

## Cek kualitas, batas pemeriksaan, dan tugas berikutnya

**Pemeriksaan internal:** 92 definisi statis cocok silang dengan **92/92** data ukuran kapal dari audit sebelumnya; setiap `X` mempunyai panjang kelipatan enam; tidak ada nilai mount non-numerik di hasil ekstraksi; contoh kiri/kanan O.S.C. Cruiser ditemukan langsung di source.

**Batas bukti:** proses ini adalah *static extraction* dan belum memvalidasi setiap mount di runtime game asli. Khusus tag slot 4/5, mount yang dihasilkan oleh `s8`, factory berparameter, dan urutan weapon variant perlu pengujian lebih lanjut. Belum memindahkan efek damage, kolisi, spawn, tekstur planet, atau file 3D ke Godot. File sumber Smali tetap tidak diubah.

**Mapping tahap selanjutnya:** (1) ekstraksi ID senjata yang kompatibel dengan tiap tag hardpoint + preset `Lv7.b`; (2) pemetaan durasi/range dan keluarga projectile/beam/missile dari `H4/Lak/Q6/Ei/Q4`; (3) integrasi katalog Sol/Earth/Luna dan definisi docking; (4) uji prototipe 1 kapal 3D + 2 turret kiri/kanan + target dummy untuk membuktikan laras benar-benar dapat tertutup oleh hull.