# Space RPG 4 v0.9995 (44) — Pemetaan Source untuk Remake Godot Full 3D

**Tanggal:** 23 Oktober 2026  
**Status:** pemetaan awal diverifikasi langsung dari ZIP, *belum* mencakup ekstraksi semua angka senjata atau implementasi ulang gameplay. Tidak ada berkas Godot yang diubah.

## Berkas yang ditinjau

| Berkas | Isi | SHA-256 |
|---|---|---|
| `smali.zip` | 635 entri ZIP, 599 file `.smali` | `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5` |
| `Space RPG 4_v0.9995(44)_src (3).zip` | 789 entri, termasuk 602 PNG, 78 OGG, dan `classes.dex` | `52c4aaed622a8f6b9f6e8ec026b36b6d1566ebb6b296f5f48dfa45310ef` |
| `SRPG4_Sol3D_Flight_v0_2_3.zip` | Proyek Godot prototipe terkini; 4 model GLB dan skrip penerbangan/laser | `cf0a820084ba7e0d8a3e323dd9673bdda00b35ef15b9f6b63f36385e7c22a991` |

## Peta subsistem prioritas

| Subsistem | Source di Smali | Peran yang telah diperiksa | Padanan yang perlu dirancang di Godot 3D |
|---|---|---|---|
| Database kapal | `h1.smali` (factory kapal), `s8.smali` (bootstrap), `u7.smali` (template) | Definisi dan registrasi kapal; statistik, dimensi, slot | `ShipDefinition` sebagai data tanpa logika rendering |
| Susunan senjata/hardpoint | `u7.smali` (`V`, `W`, `X`, `k` sebagai array terkait mount/konfigurasi), `v7.smali`, `f8.smali` metode `h([I)V` | Kombinasi template kapal dan ID weapon equip ke controller senjata aktif | `WeaponMount3D`, transform lokal per laras, batas sudut turret; **tidak** menggambar satu laser dari kepala kapal |
| Katalog dan konfigurasi senjata | `h1.smali` metode `B()V`, array `H4:[Lak;`; `ak.smali` | Registry weapon, tipe weapon, parameter tiap definisi | `WeaponDefinition`: tipe, damage, interval, energi, projectile/beam/missile, jangkauan |
| Projectile dan missile | `q6.smali`, `ei.smali`, `q4.smali`, `x4.smali`, `w6.smali`, runtime `u6.smali` | Keluarga projectile, torpedo/missile, beam/special, template dan state projectile | Sistem berbeda untuk projectile fisik, beam/hitscan, missile homing |
| Perilaku bidikan/turret | `zj.smali`, `f5.smali`, `a5.smali`, `z4.smali` | Tracking, perhitungan arah target, umur/jarak senjata | Rotasi turret + prediksi intersepsi 3D, tanpa jaminan hit |
| Upgrade | `h1.smali` catalog, `w7.smali` runtime, `kh.smali` overhaul | Targeting AI, Range Extension, energi, engine, 7 kelas subsystem | Efek upgrade sebagai pengali/aturan ke parameter dasar yang relevan |
| Galaxy & sistem bintang | `h1.smali` metode `b([Lk2;)V`, `k8.smali` | Identitas sistem, pemilik faksi, nilai galaxy map, tabel encounter | `SystemDefinition`; mode galaxy terpisah dari koordinat dalam sistem 3D |
| Planet & fasilitas | `z6.smali`, `l6.smali`, `k1.smali` | Penciptaan planet, nama, ID, koordinat internal, fasilitas, persistence | `CelestialBodyDefinition`, `StationDefinition`, docking services |
| Runtime kapal, NPC, dunia | `w7.smali`, `u2.smali`, `m3.smali`, `g1.smali` | Pemunculan kapal, runtime combat, encounter | NPC simulation, authoritative world state dan collision |
| Save/state | `k1.smali` | SQLite untuk `SolarSystem`, `Planet`, dll. | Save schema tersendiri untuk remake; jangan gunakan save SQLite asli tanpa migrasi |

## Bukti penting untuk weapon

1. `h1.smali`, `B()V` menginisialisasi registry `Lh1;->H4:[Lak;` dan membangun weapon; array panjang 2000 **bukan berarti ada 2000 jenis senjata aktif**. Jumlah definisi valid dan statistik setiap weapon masih harus diekstraksi per ID.
2. `f8.smali`, `h([I)V` membaca array pilihan senjata dari `Lu7;->W:[Lv7;` / `Lv7;->b:[I` dan merujuk registry `H4`. Ini penghubung antara *kapal, posisi hardpoint dan senjata yang dipasang*.
3. `ak.smali` menyimpan referensi keluarga `D:Lq6;`, `E:Lei;`, `F:Lq4;`, `G/H:Lx4;`. Field numerik tidak boleh dibaca sebagai angka damage tanpa memahami konteks tiap subclass/keluarga.
4. `zj.smali`, `f(JLw7;)` membaca status `Lw7;->P:Z` (Targeting AI) dan memanggil `d(F)I` dalam perhitungan tracking. Salah satu cabang mengurangi waktu prediksi (`div-long` dengan 4) ketika Targeting AI tidak aktif. Artinya bantuan bidik meningkat dengan AI, tetapi *hit* tetap bergantung lintasan dan perubahan gerak target.
5. Nilai jarak telah muncul pada jalur projectile (`Lq6;->u:F`) dan state weapon controller (`Lzj;->m:F`) termasuk kode `f5.smali`. Integrasi *Auto Engage* harus memakai jangkauan **senjata yang benar-benar terpasang**, serta modifier Range Extension (ID 206) hanya bila berlaku untuk jenis tersebut. Perhitungan per tipe dan satuan masih perlu diverifikasi sebelum memakai angka final.
6. Pada Godot v0.2.3, `scripts/sol_3d.gd` metode `_fire_visual_laser()` memunculkan **satu** visual bolt di depan lambung: `ship.global_position - basis.z * (ship.ship_length * 0.56 + 1.6)` dan cooldown tetap `0.18`. `scripts/laser_bolt.gd` menandai laser sebagai **visual only**, belum damage/weapon database. Inilah mengapa tembakan masih keluar dari 'kepala kapal' dan belum cocok untuk Auto Fight.

### Model data usulan untuk senjata Godot (belum diimplementasikan)

`weapon_id`, `weapon_type`, `damage_channels`, `projectile_speed`, `effective_range`, `max_range`, `fire_interval`, `energy_per_shot`, `tracking_rate`, `firing_arc`, `spread`, `hardpoint_size`, `is_turret`, `guidance`, `effects`.

Aturan: target lock ≠ jangkauan tembak ≠ aim prediction ≠ damage. Targeting AI tidak menambah range; Range Extension tidak membuat peluru auto-hit; server nanti memeriksa damage.

## Bukti penting untuk Sol / planet

1. `h1.smali`, `b([Lk2;)V` menginisialisasi sistem `Sol` dengan ID `0`, memilih faksi indeks `2` (F.E.R.) dan tabel spawn.
2. `z6.smali`, `a()V` menginisialisasi setidaknya **Earth** dan **Luna** sebagai dua record bernama. Record Earth mencatat `J4=0`, `K4=0`, koordinat internal `i5=0`,`j5=0`, `k5=256`; Luna mencatat `J4=1`, `K4=0`, `i5=350`, `j5=-150`, `k5=64` (nilai literal dari source). Perlu penelusuran pemakaian `J4/K4/i5/j5/k5` agar tidak salah menyebutnya jarak atau radius fisik. Beberapa data planet/stasiun dan generasi khusus masih terselip pada metode ini.
3. `k1.smali` memiliki tabel SQLite `SolarSystem` dan `Planet` dengan `solarsystemid`, jadi relasi objek-sistem perlu direkonstruksi dari penyimpanan dan generator, bukan menebak dari nama tempat.
4. Resource source menyediakan berbagai sprite peta planet (`planetmap_habitable*`, `planetmap_large*`, `planetmap_uninhabitable*`, `planetmap_stations_1.png`, dll.). Ini **gambar 2D**, bukan texture 3D sferis siap pakai tanpa penyesuaian.
5. Prototipe v0.2.3 membuat Matahari dan delapan planet sebagai desain contoh 3D. **Susunan ini belum diverifikasi sebagai replika persis sistem Sol original**. Bedakan “Sol original SRPG4” dengan penambahan sistem tata surya realistis buatan kita.

## Inventaris aset visual yang terlihat pada source

Dalam folder `res/drawable/` (PNG):
- `ship_*.png`: **312** file (meliputi varian normal, damage, shield, dsb.; bukan 312 kapal unik).
- `planetmap_*.png`: **18** file.
- `weapons*.png`: **6** file (atlas visual senjata); `weapon_*.png`: **7** file.
- File tambahan antara lain `greenbeam.png`, `lance_beam.png`, `micromissile_trail_particle.png`, `torpedo_shield.png` serta beragam explosion/shield/exhaust atlas.

Terdapat 4 GLB kapal Astra dalam proyek Godot. Root node memiliki mesh dan placeholder thruster, **belum** ada node hardpoint senjata untuk tiap jenis weapon. `01_Oktagonal.glb` memiliki `Thruster_01`–`04`, `02_Carrier.glb` memiliki lima placeholder thruster, `03_Berlian.glb` tiga, dan `04_Freighter.glb` empat (sebagian nama memakai sufiks `.001` dan seterusnya).

## Prioritas pemetaan teknis sebelum Auto Fight

1. **Weapon database penuh:** inventaris ID valid beserta damage, projectile speed, range, cooldown, energy, hubungan weapon class; periksa clone upgrade/varian agar tidak duplikat.
2. **Hardpoint per kapal:** jumlah, ukuran, arah laras, sudut putar turret, attachment node di GLB, kategori gun/missile/fixed. Slot asli 2D **tidak otomatis memiliki posisi 3D**; perlu membuat Z dan orientasi.
3. **Range evaluator & collision:** definisikan satuan jarak dunia; status `IN RANGE/OUT OF RANGE`, hit validasi per senjata, benturan fisik bukan random hit chance.
4. **Planet, Sol dan stasiun:** relasi sistem `K4`, koordinat internal, ukuran/sprite, objek yang bisa di-landing/docking; desain radius baru untuk full 3D.
5. **Flight stats/size:** batas engine, inertia, turn rate dan skala 3D pada model Astra; pembandingan kamera pada world scale yang sama.
6. **Overhaul & upgrades:** katalog 31 item dan tujuh kelompok subsystem sudah diaudit terpisah; lanjut detail multiplier per tipe senjata.
7. **NPC/Auto Engage:** setelah senjata punya range dan mount benar, uji dummy target, tracking, manual fire, Auto Fire dan Cancel, baru Approach/Orbit/Keep Range.
8. **Docking:** gunakan rancangan Earth orbital station + hangar 3D, dan mega-dock untuk Titan; status docking dan combat saling eksklusif.

## Batas kepastian dan perubahan proyek

**Sudah dikonfirmasi:** keberadaan kelas dan jalur di atas, register weapon, sumber data Sol/Earth/Luna, inventaris file aset, visual-only laser dan empat model GLB.  
**Belum selesai:** tabel angka tiap weapon valid, posisi 3D setiap hardpoint, ukuran fisik planet, semua hubungan sistem-planet, formula collision persis, semua varian upgrade dan profil NPC. Tidak ada klaim semua mekanisme game original sudah direplikasi.

**Perubahan proyek Godot:** tidak ada. Pemetaan ini adalah dasar analisis untuk implementasi setelah kontrol, kamera dan satuan ukuran matang. Semua aset 2D pihak ketiga untuk distribusi publik membutuhkan izin pemegang hak cipta.