# Mapping Tahap 11 — Grav Sensors, Analytic Sensors, Turret Variants & Damage Modifier

**Tanggal:** 23 Oktober 2026 · **Referensi:** Space RPG 4 v0.9995(44) · **Audit:** pemeriksaan statis ZIP `smali.zip` (SHA-256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`). **Tidak ada perubahan APK/Godot dan tidak ada tes runtime.**

Tahap 11 memperdalam [Tahap 10](12_TURRET_DAMAGE_SENSOR_TAHAP10.md) dan memberi satu koreksi penting: **upgrade Grav/Analytic Sensors pada kode asli secara langsung mengontrol tampilan radar/HUD, bukan bukti bonus jangkauan senjata atau radar agresi NPC.**

## 1. Pemetaan sensor original: upgrade → status → HUD

Penelusuran `h1.smali`, `w7.smali`, `q7.smali`, `p7.smali`, dan `u2.smali` memberi crosswalk berikut:

| Upgrade | ID | Registry | Efek pada kapal `w7` | Konsumen runtime |
|---|---:|---|---|---|
| Grav Sensors | **210** (`0xD2`) | `h1.H` | `w7.R = 1` | `q7.a(J)` |
| Analytic Sensors | **211** (`0xD3`) | `h1.I` | `w7.R = 2` | `q7.a(J)`, `q7.b(p7,w7)` |

- **Bukti:** `w7.smali` sekitar baris fisik **9632–9653** menguji `h1.H/h1.I` lalu menulis `R`. `w7` menginisialisasi dan mengosongkan `R=0` dalam konstruktor dan saat membangun ulang status upgrade.
- `q7.a(J)` baris fisik **~60–105** membaca `player.w7.R`; `R != 0` mengaktifkan indikator sensor; `R == 2` mengaktifkan mode analitik. `q7.b(Lp7;Lw7;)I` baris **4844–5145** menentukan indeks sprite/warna dan variasi ukuran indikator.
- Mode analitik memakai hubungan faksi **dua arah** `k2.c` dan informasi threat `y7.c`, selain status spesial kapal. Dalam cabang klasifikasi sprite ada nilai **11/27/43/59** sebagai indeks sprite/palette dan kelas ukuran turunan `Lu7.f`. **Angka tersebut bukan jangkauan radar atau damage**.
- `q7.d(Lf3;)V` sekitar baris **5553–5880** mendaftarkan objek dunia ke map `q7.i:HashMap` sebagai kontak `p7`. `i3` dan `u2` memanggil `q7.d` ketika objek dipasang. `q7.a(J)` kemudian mengolah/menampilkan penanda; sebagian keputusan memakai hasil pengecekan viewport `z0.a(FF)`, posisi relatif objek dan transisi indikator (termasuk offscreen).
- **`q7` adalah subsistem sensor HUD untuk kapal pemain**. Tidak ada bukti dari jalur ini bahwa `w7.R` menambah jarak kunci senjata, jarak tembak turret, atau sensor aggro AI NPC.

### Yang disebut deskripsi original
`strings.xml` menyatakan **Grav Sensors** menampilkan keberadaan, arah, dan ukuran kasar kapal yang di luar tampilan **tanpa pewarnaan relasi**. **Analytic Sensors** menambahkan identifikasi/pewarnaan berdasarkan disposition/signature. Ini **sesuai** dengan jalur HUD yang ditemukan. Tetapi string bukan kontrak seluruh aturan deteksi: belum ditemukan konstanta *sensor max distance* universal dalam `q7`, dan jangan menjadikan parameter visual `3760` sebagai radar_range.

### Perbedaan empat jarak dalam 3D
| Sistem | Definisi remake | Apakah sudah ada angka asli tervalidasi? |
|---|---|---|
| Contact detection | Objek dapat menjadi kontak pada UI sensor | **Belum** ada radius universal yang terbukti |
| Combat target lock | Pemain dapat mengunci objek sebagai target tempur | **Belum** ada rumus/limit final |
| Weapon firing range | Syarat tembak per-mount `zj.m` dan modifier | Sebagian **sudah**, lihat Tahap 10 |
| HUD render/offscreen indicator | Bagaimana kontak diproyeksikan ke layar/kompas | Alur `q7/p7/z0` **terpetakan** |

**Rekomendasi:** radar 3D menggunakan arah azimuth + elevasi, tapi indikator kehadiran kontak **tidak otomatis berarti serangan diperbolehkan**. Combat tetap memerlukan target sah, jarak senjata, arc, LOS, power, ammo, cooldown. Jika hendak memberi kemampuan bonus deteksi/lock untuk Analytic Sensors di remake, tandai sebagai **fitur desain baru**, bukan port tepat original.

## 2. Tipe controller senjata: jangan satu script laser untuk semua

Kelas `Lzj` mengatur mount dasar; `Lf8` membentuk mount sesuai slot dan senjata yang di-equip. Keluarga subclass memiliki **method berbeda**:

| Controller | Base | `a()Z` | `b(Lw7;)I` fire | `f(JLw7;)V` tracking |
|---|---|---|---|---|
| `a5` | `zj` | ~250 | ~486 | ~1331 |
| `f5` | `zj` | ~230 | ~466 | ~880 |
| `z4` | `zj` | ~234 | ~457 | ~2345 |
| `gi` | `zj` | ~21 | ~272 | ~1599 |

- `a5/f5` terkait katalog `q6` projectile, tetapi punya urutan dan cara fire/tracking sendiri. **Jangan menyamakan** `f5` dengan `a5` tanpa memeriksa cabang per senjata.
- `z4` menggunakan keluarga `q4` beam; jalur fire/power dan efek sudut berbeda dari projectile.
- `gi` menggunakan `ei` torpedo dan mewajibkan target di cabang firing yang sebelumnya ditelusuri, dengan guidance di `fi/k7`.
- `f8.d()` memisahkan defense weapon bertipe `zj.a == 2` (CIWS/jalur defense) dari salvo ofensif.
- `f8.a(J)` meng-update tracking dan cooldown. `f8.g()` memilih mount siap tembak dan memanggil `zj.b(target)`, mengurutkan readiness dengan `TreeMap`. Ini penting agar laser prototipe sementara yang berasal dari ujung kapal **tidak** menjadi desain final sistem.

Untuk Godot disarankan `WeaponMount3D` (yaw/pitch/occlusion/cooldown) + `WeaponBehavior` (`PROJECTILE`, `BEAM`, `MISSILE`, `DEFENSE`, `SPECIAL`) dengan data original yang dapat diimpor. Model kapal 3D harus memiliki muzzle/hardpoint yang benar.

## 3. Damage: bukti baru untuk Adaptive Shielding

Pengecekan silang:

1. `h1.K` adalah **Adaptive Shielding (ID 213)** menurut registry dan katalog.
2. Ketika upgrade ini terpasang, `w7.smali` baris **~9656–9668** menulis **`e8.m = true`**.
3. `e8.b(FFFFFF)I` baris **~3490–3550** melakukan `if e8.m: p1 *= 0.85` sebelum tahap pengurangan shield.
4. Jadi efek **mengurangi 15% kanal damage yang masuk ke shield** memiliki jalur kode yang dapat ditelusuri; berbeda dari sekadar string katalog.

**Penting:** Ada pengali lain **0.85, 0.75, 0.7** dalam `e8.b` yang memakai flag/status kapal lain (`w7.H/I/T`, `u2.M`, kondisi misi/player/fighter). Angka pengali yang sama **tidak otomatis berarti** berasal dari upgrade Adaptive Shielding. Jangan menggandakan pengali tanpa mengikuti cabang aslinya.

### Defense stages yang telah dikonfirmasi pada Tahap 10
`e8.b/f` shield current/max; `e8.d/h` armor 4 sektor; `e8.c/g` hull current/max; `b8.b/c/d` energy current/regen/max. `e8.b(FFFFFF)` memakai impact point lokal dan beberapa kanal projectile/torpedo. Penamaan fisik tepat seluruh kanal `p1..p4` dan rumus subsystem damage lengkap **masih perlu audit**.

## 4. Dampak untuk desain sensor, NPC dan AUTO ENGAGE 3D

**Jangan mencampur sensor HUD dengan otoritas tempur:** AI NPC menggunakan `y7/m` untuk ancaman dan keputusan; `f8.r` (Tahap 9) merupakan **ringkasan jarak senjata**, bukan radius sensor.

Proposal alur:
```text
World contacts -> Sensor HUD (GRAV / ANALYTIC)
   └─ select ship -> VALIDATE TARGET LOCK
                       └─ AUTO ENGAGE explicitly ON
                          ├─ navigation range plan: APPROACH / KEEP RANGE / ORBIT
                          └─ for EACH mount: max weapon range, yaw/pitch, hull LOS,
                             aim prediction, cooldown, power, ammo, type behavior
                             └─ projectile collision / beam / missile guidance
```
Semua navigasi otomatis dapat dibatalkan kapan pun dan tidak menghapus inertia. Untuk multiplayer nanti, server memvalidasi target legal, hit, damage dan transaksi docking; HUD boleh dihitung client-side.

**Aspek khusus 3D yang belum terdapat di source 2D:** elevasi turret, raycast occlusion dengan hull sendiri, projection blip azimuth/elevation, physics projectile 3D, 3D armor hit sectors. Pemain dapat mengaktifkan CAMERA LOCK tanpa mempengaruhi TARGET LOCK.

## 5. Test matrix Tahap 11 — BELUM DIJALANKAN

| Test | Skenario | Kriteria |
|---|---|---|
| T11-A | Tanpa Grav/Analytic | Sensor ring offline; HUD lain tetap berfungsi |
| T11-B | Grav dipasang | Contact ring aktif, ada arah+kelas ukuran tanpa klasifikasi relasi |
| T11-C | Analytic dipasang | Contact ring memiliki warna klasifikasi faksi/threat dinamis |
| T11-D | Hostile offscreen tetapi di luar weapon range | Kontak terlihat, **tidak** dapat ditembak |
| T11-E | NPC tanpa upgrade sensor | NPC combat tidak otomatis kehilangan target karena HUD player mati |
| T11-F | Beam vs cannon vs torpedo vs CIWS | Controller berbeda; per-weapon gating benar |
| T11-G | Adaptive Shielding | Kanal damage ke shield memakai 0.85 saat flag `e8.m` aktif |
| T11-H | Turret kiri, target kanan, pitch tidak valid | Tidak fire, tidak clipping hull |
| T11-I | Auto Engage dimatikan saat docking/joystick | Tidak snap kecepatan; berhenti auto navigation |
| T11-J | Banyak kontak di kamera 3D/free look | HUD label tidak melompat, icon offscreen benar |

## 6. Trace matrix dan batas mapping
Bukti dari ZIP disajikan di [Crosswalk Tahap 11](../data/sensor_weapon_damage_crosswalk_stage11.csv) dan format JSON [Rules Tahap 11](../data/sensor_modes_and_rules_stage11.json).

**Belum selesai:** rumus contact detection jika ada cut-off, combat lock range asli, prioritas sensor saat cloak/ECM scan, semua beda spesifik `f5`, formula lengkap armor/penetrasi/subsystem, game runtime test. Setelah itu lakukan **Tahap 12: desain harness tes dua turret vs satu Interceptor** terlebih dahulu, lalu implementasi Godot hanya jika pengguna menyuruh. Mapping statis tidak berarti fitur sudah dapat dimainkan.

**Source audit:** `smali.zip`, bukan APK yang diubah. Jangan unggah Smali/asset game original ke repository laporan publik.
