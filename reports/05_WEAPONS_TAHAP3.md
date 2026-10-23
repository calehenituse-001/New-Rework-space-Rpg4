# Space RPG 4 v0.9995(44) — Mapping Senjata Tahap 3 (Godot Full 3D)

**Status:** audit statis **berdasarkan file asli `smali.zip`**, tanpa mengubah game atau proyek Godot. Dikerjakan sesudah Mapping Hardpoint Tahap 2. Angka di bawah berada dalam **satuan internal game**, bukan meter; jangan masukkan ke Godot 3D sebagai meter tanpa faktor konversi dan pengujian.

**Sumber:** `smali.zip`, SHA-256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`; terutama `h1.smali` metode `A()` (katalog barang), `B()` (registry senjata), beberapa factory `N3()` s.d. `i4()`, `r6.smali`, `ak.smali`, `q6.smali`, `ei.smali`, `q4.smali`, `x4.smali`, dan `f8.smali`. Perhitungan dilacak dari konstanta yang secara eksplisit ditulis pada bidang objek (*literal assignments*), bukan menjalankan APK.

## Ringkasan audit

- Registri senjata dasar `Lh1.H4` panjang **2000 slot**. Dari `h1.B()` terkonfirmasi **79 ID dasar yang diisi**, 0–78 (beberapa adalah placeholder/khusus dan tidak cocok digunakan sebagai senjata tempur biasa).
- Nama dari katalog `h1.A()` berhasil dikaitkan secara langsung untuk **53/79 ID**. **26 ID tidak diberi nama** jika relasi `La4` ke `Lak` belum dipastikan, supaya tidak mengarang label torpedo, varian uji, atau senjata khusus.
- Keluarga: **56** `q6` proyektil, **18** `ei` torpedo/rudal, **1** `q4` beam, **3** `x4` khusus, **1** (tipe pertahanan khusus tidak memakai pointer keluarga reguler).
- Diperoleh **74 estimasi jarak berbasis konstanta** untuk `q6/ei` dengan rumus internal `kecepatan × umur proyektil_ms / 1000`; angka `range_*_candidate` **bukan jarak optimal tempur**, belum memasukkan perubahan runtime, muatan senjata, atau kondisi khusus. Rumus ini didukung kalkulasi yang mengisi `Lq6.u` dan `Lei.K` pada source.
- `h1.B()` menggandakan senjata dasar aktif ke ID `500+n`, `1000+n`, `1500+n` (grade `Lak.c = 1,2,3`). Pemetaan **316 pasangan ID+grade** disimpan terpisah, **bukan 316 senjata unik**. Getter `Lak.a()`/`c()` mengalikan *dua kanal statistik* dengan `1 + 0.2×grade`, tetapi ini **tidak berarti semua atribut** atau semua tipe ikut dikalikan.
- Tipe slot `Lak.b` secara eksplisit terdiri dari 0,1,2,4,5. Kecocokan dengan tag `Lu7.X[6*i]` di Mapping Tahap 2 sangat kuat untuk Small/Medium/Large/Torpedo/SuperHeavy; final `equip-compatibility` masih perlu diuji dari logika pemeriksaan UI/slot.
- Tipe runtime `Lak.a=2` adalah **CIWS**, bukan otomatis menyerang badan kapal lawan; `a=4` **AD PD** belum boleh dijadikan senjata ofensif. AUTO ENGAGE harus mengecualikannya dari penghitungan DPS ke kapal.

## Contoh angka benar-benar terambil dari source

Satuan: kecepatan internal, jarak perkiraan internal. **Bukan ukuran m/s maupun meter fisik.**

| ID | Senjata | Keluarga | Slot | Speed mentah | Range kandidat |
|---:|---|---|---:|---:|---:|
| 1 | Blaster | q6 projectile | 0 | 288.0 | 432.0 |
| 2 | Striker | q6 projectile | 0 | 264.0 | 422.4 |
| 9 | CIWS | q6 projectile | 0 | 480.0 | 240.0 |
| 10 | Warden CIWS | q6 projectile | 0 | 480.0 | 336.0 |
| 22 | Med. Cannon | q6 projectile | 1 | 270.0 | 486.0 |
| 24 | Irochaser | q4 beam | 1 | — | — |
| 25 | QL Emitter | x4 special | 1 | — | — |
| 32 | Nova Core | q6 projectile | 2 | 300.0 | 540.0 |
| 40 | H. Autocannon | q6 projectile | 2 | 300.0 | 450.0 |
| 41 | H. Cannon | q6 projectile | 2 | 282.0 | 564.0 |
| 43 | Super Howitzer | q6 projectile | 2 | 240.0 | 600.0 |
| 49 | Ion Cannon | q6 projectile | 2 | 330.0 | 660.0 |
| 51 | Irohammer | q6 projectile | 5 | 300.0 | 600.0 |
| 52 | Sky Piercer | x4 special | 5 | — | — |
| 53 | Illuminator | x4 special | 5 | — | — |
| 54 | Vulcanus | q6 projectile | 5 | 294.0 | 705.6 |
| 61 | Belum dipetakan | ei torpedo | 4 | 150.0 | 1500.0 |
| 62 | Belum dipetakan | ei torpedo | 4 | 162.0 | 1620.0 |
| 78 | Belum dipetakan | ei torpedo | 4 | 330.0 | 3300.0 |

**Catatan:** Irochaser adalah kelas beam `q4`, sehingga kolom ballistic `kecepatan × lifetime` tidak relevan. Sky Piercer, Illuminator dan QL Emitter memiliki perilaku khusus `x4` dan tidak boleh dipaksa menjadi proyektil biasa. CIWS punya angka kecepatan proyektil, tetapi penggunaan targetnya berbeda.

## Struktur senjata terverifikasi

**Weapon registry:** `Lh1;->H4:[Lak;` → `Lak.a` jenis runtime, `Lak.b` kategori mount; `Lak.D=Q6`, `.E=Ei`, `.F=Q4`, `.G/.H=X4` menunjuk tipe fisika/senjata. **Jangan gunakan** `Lak.c` sebagai damage mentah: itu *upgrade tier*.

**Projectile (`Lq6`)**: `f:F` speed mentah, `k:J` umur proyektil dalam milidetik, `u:F` jangkauan turunan dari `f×k/1000` (lihat nilai ditulis di builder). `l:I` dipakai sebagai dasar interval di `Lak.d()` (dapat berubah jika ada burst: `Lak.l` dan `Lak.k`). Field `g/h/i/j` berhubungan dengan damage/dampak, tetapi jangan semua disebut sebagai satu angka *damage* tanpa pemeriksaan formula armor/shield.

**Torpedo (`Lei`)**: `f:F` speed, `k:J` lifetime, `K:F` derived range; `n:I` nilai jeda peluncuran yang direturn `Lak.d()`. `Lei` juga punya parameter steering, target rear attack, guidance, ECM, dan ammo yang memerlukan audit terpisah agar remake tidak sekadar memakai homing sempurna.

**Beam (`Lq4`)** dan **special (`Lx4`)**: jenis dan parameter berbeda (lihat `f8.h([I)V` untuk instansiasi controller). Angka ballistic normal **tidak berlaku**. `Lak.d()` untuk beam terdeteksi 4000 ms, sementara untuk `x4` bergantung pada field `d`. Ini belum menggambarkan seluruh perilaku beam, durasi tembak atau tracking.

**Prinsip runtime:** setiap kapal punya susunan mount dari `Lu7.X`, weapon yang terpasang dari `Lv7.b` atau loadout save, yang kemudian dibangun menjadi controller oleh `Lf8.h([I)V`. Jumlah mount tetap bukan jumlah weapon aktif; `-1` berarti slot kosong.

## Kompatibilitas kelas senjata ↔ slot kapal

| Slot `Lak.b` | Arti sementara | Tag `Lu7.X` | Konsekuensi Godot 3D |
|---:|---|---:|---|
| 0 | Small | 0 | CIWS dan cannon ringan; turret/senjata kecil |
| 1 | Medium | 1 | Turret menengah, termasuk beam khusus medium |
| 2 | Large/Heavy | 2 | Meriam besar, cek posisi lambung dan putar turret |
| 4 | Torpedo | 4 | Launcher, pelacakan rudal independen; bukan cannon turret |
| 5 | SuperHeavy | 5 | Senjata kelas Titan; harus cek ukuran mount dan collision |

**Perhatian:** tidak ada kategori tag 3 dalam data mount yang ditarik sebelumnya; ini tidak berarti kategori engine 3 mustahil di kode selain 92 template yang diperiksa. Status kategori membutuhkan validasi UI `equip`.

## Rancangan Auto Engage sesuai permintaan

Setiap **mount aktif** mempunyai jarak tembak sendiri, bukan satu angka range untuk semua turret:

1. TARGET LOCK memilih kapal atau objek target; boleh dilakukan di luar jangkauan firing. Syarat sensor/line-of-sight bisa diterapkan terpisah dari weapon range.
2. AUTO ENGAGE menentukan *optimal engagement distance* dari kumpulan senjata yang **boleh menargetkan kapal**, sudah memuat upgrade dan masih punya energi/amunisi. Jangan biarkan 1 torpedo jarak jauh memaksa seluruh meriam pendek menyerang dari jauh.
3. Untuk **setiap hardpoint**: verifikasi slot equip → target dalam range senjata (setelah upgrade yang relevan) → lintasan yaw/pitch diizinkan → turretnya bisa berputar → cek tabrakan arah moncong ke lambung sendiri/objek → cek cooldown, energi, dan ammo → lakukan fire.
4. Untuk `q6`, bidikan memprediksi titik intersepsi 3D berdasarkan kecepatan target dan kecepatan proyektil. Targeting AI meningkatkan estimasi (bukan 100% hit) dan **tidak mengubah range atau firing arc**.
5. Untuk `q4` beam/hitscan gunakan garis bidik dan tracking mekanis, bukan lead time balistik. Torpedo `ei` menggunakan guidance dan countermeasures, bukan raycast instan.
6. **ORBIT**, **KEEP AT RANGE** dan **APPROACH** hanya mengendalikan navigasi. Pemain boleh cancel kapan saja, dan pembatalan tidak menghapus inertia. Kamera LOCK tetap independen dari AUTO ENGAGE.
7. Untuk multiplayer: server memverifikasi amunisi, senjata, jam cooldown, line of sight, jangkauan dan hasil hit; client menampilkan VFX/animasi.

### Skenario dasar validasi 3D

- Battleship dengan turret kiri sektor 180° menghadapi musuh di kanan: turret kiri tidak mendapat izin tembak walaupun target dalam range, turret kanan bisa.
- Auto Engage memilih target 700 unit dari Ion Cannon dasar (`range ≈ 660` internal): Cannon **tidak fire**, autopilot bisa mendekat. Range Extension pada senjata yang berhak memakai modifier mengubah evaluasi jangkauan (harus cek nilai runtime khusus per jalur sebelum menganggap +30% universal).
- Interceptor terbang lurus: lebih mudah diprediksi daripada interceptor yang mengubah vektor kecepatan saat proyektil sudah meluncur.
- Beam di luar sudut pitch/terhalang sayap: tidak fire, walaupun tidak bergantung travel time.
- CIWS ditempatkan untuk pertahanan torpedo, bukan untuk menghajar hull kapal musuh.

## Batasan dan pekerjaan mapping berikutnya

**Tuntas pada tahap ini:** registri 79 entry dasar, 53 nama katalog yang dapat diverifikasi, kelompok statis, speed/lifetime mayoritas proyektil, data torpedo, ID varian grade, cross-map kelas slot.

**Belum tuntas:** beberapa nama dan kategori item khusus; nama ammo torpedo dan perilaku guidance; perhitungan damage lengkap per armor/shield; efek upgrade runtime; semua aturan equip; burst-delay, rangkaian beam/special; preset default `Lv7.b` per kapal. **Jangan klaim jarak ini adalah true effective range setiap weapon pada semua situasi**. Data dari source statis belum diuji di runtime game asli.

**Rencana Tahap 4:** (a) audit torpedo guidance/ammo/ECM + Range Extension edge cases, (b) relasi preset `Lv7.b` dengan hardpoint `X`, (c) mapping SOL + station + planet, (d) desain prototype firing 3D dua turret + target dummy menggunakan data yang sudah dikonfirmasi. Blueprint docking orbital + hangar 3D + Titan mega dock tetap dipakai; **belum** diimplementasikan.

**Status proyek Godot:** tidak disentuh, tidak ada perubahan APK. Asset game asli untuk distribusi publik memerlukan izin pemilik hak cipta.