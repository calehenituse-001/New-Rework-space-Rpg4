# Audit Upgrade dan Rancangan Auto Engage — Space RPG 4 v0.9995 (44)

Tanggal: 23 Oktober 2026. Basis audit: `smali.zip` (635 entri); `Space RPG 4_v0.9995(44)_src (3).zip` (789 entri). Data dibaca langsung dari berkas yang dilampirkan pengguna. **Tidak ada perubahan pada proyek Godot.**

## Batasan dan metode verifikasi

- Katalog upgrade: `smali/h1.smali` (factory item ke `La4;` / array `Lh1.c0`), nama dan deskripsi `res/values/strings.xml` (ID teks dipetakan lewat `res/values/public.xml`).
- **ID**, **harga awal** (`La4.f`), **mass** (`La4.d`), dan **batas pemasangan** (`La4.j`) diekstrak dari factory. Batas pemasangan juga diuji secara logika di `smali/d6.smali` dengan perbandingan `Lv5.b` terhadap `La4.j`.
- Runtime efek: `smali/w7.smali` menerapkan flag dan komponen upgrade; `smali/zj.smali`, `smali/f5.smali`, `smali/a5.smali`, `smali/r6.smali`, `smali/z4.smali` memiliki sebagian logika senjata.
- Angka-angka yang berasal dari **deskripsi game** tidak otomatis berarti sudah terbukti berlaku sama pada setiap tipe proyektil. Perlu pengujian game untuk memastikan hasil lintas senjata.
- Ini merupakan catatan untuk **rancangan remake Godot**, bukan salinan engine siap jalan. Keterangan legal aset dari game pihak ketiga tetap perlu diperhatikan sebelum distribusi publik.

## Inventaris 31 upgrade / hangar yang ditemukan

| ID | Upgrade | Harga awal | Mass | Batas | Efek menurut data asli |
|---:|---|---:|---:|---:|---|
| 200 | Engine Overdrive | 40,000 | 20 | 1 | Boost sementara melampaui kecepatan maksimum. |
| 201 | Engine Additives | 25,000 | 10 | 1 | Peningkatan performa engine sekitar +20%; ilegal di banyak wilayah. |
| 202 | Weapon AMP | 30,000 | 15 | 1 | +10% kerusakan tiap tembakan (kecuali torpedo), +20% konsumsi daya. |
| 203 | Extra Armour | 25,000 | 25 | 4 | +30 armour pada setiap sisi per unit. |
| 204 | Enhanced Armour | 45,000 | 20 | 1 | +2 Armour Rating (AR), meningkatkan kemampuan menahan damage per hit. |
| 205 | Ultra Armour | 80,000 | 25 | 1 | +3 Armour Rating (AR). |
| 206 | Range Extension | 30,000 | 15 | 1 | Deskripsi: +30% jangkauan senjata proyektil standar, tidak berlaku untuk torpedo / beam. Implementasi punya beberapa jalur multiplier; cek runtime per jenis. |
| 207 | ECM | 38,000 | 10 | 1 | Mengurangi akurasi guidance torpedo musuh, kemungkinan hilang lock; mempersulit scan. |
| 208 | Targeting AI | 20,000 | 10 | 1 | Meningkatkan akurasi turret melalui prediksi pergerakan target; tidak menjamin hit. |
| 209 | Safety Overrides | 20,000 | 10 | 1 | +20% projectile weapon damage; peluang 0,3% tiap tembakan terjadi overload (power terkuras). |
| 210 | Grav Sensors | 15,000 | 5 | 1 | Radar indikator arah/ukuran kasar objek di luar tampilan; tanpa klasifikasi warna. |
| 211 | Analytic Sensors | 25,000 | 15 | 1 | Radar lanjutan dengan analisis signature dan klasifikasi warna. |
| 212 | Mass-UP | 45,000 | -50 | 3 | Konversi permanen 50 ton cargo space menjadi 50 ton mass capacity; tak bisa dibalik. |
| 213 | Adaptive Shielding | 50,000 | 20 | 1 | Mengurangi damage yang masuk ke shield sebesar 15%. |
| 214 | FTL Optimisation | 15,000 | 10 | 1 | Memungkinkan FTL di titik 30% lebih dekat ke pusat sistem. |
| 215 | Auxiliary Power | 100,000 | 25 | 1 | Unit daya tambahan yang dapat dialihkan ke shield, weapon, engine atau recharge. |
| 216 | Adv. Guidance | 20,000 | 10 | 1 | Mengurangi dampak ECM/nebula terhadap torpedo 50%; pencegahan kehilangan lock (menurut deskripsi). |
| 217 | Survey Equipment | 10,000 | 10 | 1 | Survei planet baru untuk memperoleh Survey Data. |
| 218 | Fuel Pod | 15,000 | 10 | 2 | Menambah kapasitas fuel setara dua lompatan per pod. |
| 219 | Fighter Hangar | 100,000 | 70 | 3 | Hangar 3 F.E.R. fighters; cocok menghadapi target kecil. |
| 220 | Bomber Hangar | 125,000 | 80 | 3 | Hangar 3 F.E.R. bombers; serangan bom tanpa guidance ke target besar. |
| 221 | Fighter Hangar | 100,000 | 70 | 3 | Hangar 3 T.A. fighters. |
| 222 | Bomber Hangar | 125,000 | 80 | 3 | Hangar 3 T.A. bombers. |
| 223 | Fighter Hangar | 100,000 | 70 | 3 | Hangar 3 O.S.C. fighters. |
| 224 | Bomber Hangar | 125,000 | 80 | 3 | Hangar 3 O.S.C. bombers. |
| 225 | Type1-F Hangar | 95,000 | 65 | 3 | Hangar 3 Type1-F fighters. |
| 226 | Type1-B Hangar | 115,000 | 75 | 3 | Hangar 3 Type1-B bombers. |
| 227 | Merlin Hangar | 105,000 | 70 | 3 | Hangar 3 Merlin fighters (faksi pemain). |
| 228 | Kingfisher Hangar | 130,000 | 80 | 3 | Hangar 3 Kingfisher bombers (faksi pemain). |
| 229 | Gressian E-FTL | 30,000 | 10 | 1 | Meningkatkan efisiensi FTL kapal lebih besar dari size 1; menurunkan konsumsi fuel. |
| 240 | Fetcher Drone | 75,000 | 50 | 2 | Drone otomatis mengambil loot saat tidak ada musuh di sekitar. |

**Catatan Mass-UP:** `La4.d` menyimpan `-0x32` = −50. Itu bukan biaya mass negatif biasa, melainkan hasil konversi cargo menjadi mass; perlu menerapkan aturan pembelian tersendiri.

## Subsystem Overhaul (berbeda dari upgrade tambahan)

Di `smali/kh.smali` fungsi `f(I)` terdapat **7 kelompok** subsystem: **Weapons**, **Engines**, **Torpedoes**, **Hull**, **Shields**, **Thrusters**, **Powerplant**. Factory varian subsystem di `smali/h1.smali` (`i(IIF)` memanggil `m/n/o/p/q/r/s` menurut kelompok).

| Subsystem | Dampak pada remake Godot |
|---|---|
| Weapons | Statistik tembakan: damage, akurasi/tracking per turret, timing/rate; pilihan varian mesti mengikuti kemampuan weapon asli. |
| Engines | Akselerasi/kecepatan maksimum kapal, termasuk pengaruh damage subsystem. |
| Torpedoes | Fire rate/guidance dan atribut terkait torpedo. |
| Hull | Kapasitas hull dan ketahanan komponen. |
| Shields | Kapasitas, recharge rate dan delay sampai mulai isi ulang. |
| Thrusters | Turn rate serta kelincahan rotasi kapal 3D. |
| Powerplant | Kapasitas energi dan output/recharge untuk senjata serta modul tambahan. |

Referensi label fitur UI ada di `strings.xml`: `ui_planet_subsystem_spec_*` (contoh `engines_maxvelmod`, `thrusters_turn`, `powerplant_capacity`, `powerplant_chargerate`, `shields_timeuntilrecharge`). Jenis model yang tersedia berbeda antarplanet. **Nilai setiap tier overhaul belum semuanya diuraikan pada audit ini**.

## Temuan penting: Targeting AI asli ternyata menggunakan prediksi

Dalam `smali/h1.smali`, upgrade **ID 208**, `Lh1.F` mengaktifkan `Lw7.P = true` ketika dipasang (`smali/w7.smali`). Di `smali/zj.smali`, metode `f(JLw7;)` mengolah pergerakan target dan waktu terbang peluru melalui `d(F)I` untuk memperkirakan lokasi sasaran. Pada salah satu cabang untuk turret, ketika `Lw7.P` **false**, nilai waktu prediksi dibagi **4** sebelum dipakai menghitung offset bidik. Saat `Lw7.P` **true**, pembagian itu tidak dilakukan. Ini mendukung interpretasi bahwa **tanpa upgrade tetap ada lead terbatas**, sedangkan AI memperkuat prediksi. Bukan hit chance 100%.

Perilaku lengkap berbeda menurut senjata, status musuh/NPC, nebula serta logika di kelas anak turret. Jangan menganggap semua senjata memakai pengganda prediksi identik. Target masih bisa mengubah arah, tracking turret terbatas, projectile mempunyai waktu tempuh, dan collision menentukan hit.

## Temuan penting: jarak maksimum nyata, bukan sekadar target lock

- **Range Extension (ID 206)** = `Lh1.D`, mengaktifkan `Lf8.k` melalui `w7.smali`.
- Teks item mengklaim **+30% jangkauan proyektil** dan mengecualikan torpedo/beam.
- Dalam beberapa jalur code projectile/weapon terlihat **×1,3** (`smali/a5.smali`, `smali/r6.smali`, `smali/z4.smali`), sementara satu jalur (`smali/f5.smali`) terlihat **×1,4** untuk field jarak `Lzj.m`. Ini perlu diverifikasi per kelas senjata saat mengimplementasikan ulang. Tidak tepat memaksakan angka tunggal untuk semua senjata.
- Sistem lock sensor, jangkauan senjata, jarak tembak efektif, batas arc turret, dan kemampuan mendahului target adalah **lima aturan berbeda**.

## Kesepakatan desain AUTO ENGAGE untuk remake full-3D

1. Pemain tetap bisa bebas mengendalikan kapal secara manual. Pilih sasaran dan tekan AUTO ENGAGE untuk mengaktifkan autopilot **khusus combat**.
2. Kapal mendekat hanya sampai **jarak optimal** yang berasal dari senjata terpasang, bukan jarak hardcoded satu angka. Kecepatan diredam secara halus untuk menghindari overshoot.
3. Setiap hardpoint memiliki `weapon_range`, `effective_range`, `tracking_rate`, `firing_arc`, `projectile_speed`, `capacitor_cost`, `cooldown`, `damage_profile` dan opsi guide/homing. Penerapan upgrade dilakukan ke parameter yang tepat.
4. Turret menembak **hanya bila target berada dalam jangkauan senjata itu**, arah bidik berada dalam firing arc, senjata siap, dan energi/ammo cukup. Lock boleh tetap aktif ketika OUT OF RANGE; menekan FIRE tidak menciptakan hit otomatis.
5. Jika beberapa jenis senjata berbeda terpasang, autopilot memilih jarak utama berdasarkan kelompok senjata yang diprioritaskan, tidak menganggap seluruh turret harus memiliki jangkauan sama. HUD memperlihatkan jumlah senjata siap vs di luar jangkauan.
6. **Targeting AI (ID 208)** meningkatkan estimasi titik intersepsi (lead aim) dan respons estimasi, tidak menambah jarak dan tidak membuat peluru teleport. Gerak musuh, latency, tracking turret, collision dan spread tetap relevan.
7. **Projectile:** hit ditentukan simulasi peluru dan collision. **Beam/hitscan:** dampaknya di jalur sinar saat ditembak; masih bisa gagal jika turret tidak melacak. **Missile/torpedo:** panduan sendiri, ECM/nebula/Adv. Guidance memengaruhi guidance.
8. Pilihan **APPROACH / KEEP AT RANGE / ORBIT** hanya bekerja dalam konteks pertempuran. Jaga jarak aman dari planet, station, dan batas docking.
9. **CANCEL AUTO** memulihkan kontrol pemain segera tanpa menghentikan inertia. Gerakan joystick aktif yang disengaja dapat membatalkan autopilot navigasi (opsional di Settings). Kamera LOCK / FREE LOOK tetap independen.
10. Jika sasaran hancur, warp keluar, atau tidak terdeteksi: Auto Engage berhenti menyerang, kapal tetap aman; opsi re-target bisa dibuat nanti.
11. Multiplayer dirancang **server-authoritative**: kapal, jarak, validitas target, projectiles/hit, damage, upgrade terpasang dan status docking diperiksa server.

## Docking — keputusan desain tetap

Terbang normal menuju Earth → pilih stasiun → masuk zona approach → DOCK → assist memperlambat, menyelaraskan, masuk ke hangar 3D → menu Commerce / Garage / Shipyard / Outfit / Overhaul / Repair → UNDOCK. Kapal ukuran Titan menggunakan **orbital mega-dock** alih-alih hangar kecil. Auto Engage dan auto docking tidak aktif bersamaan. Fitur masih tahap desain.

## Urutan kerja mendatang (belum dibangun)

1. Kontrol analog, kamera LOCK, CAMERA dan FREE LOOK yang nyaman.
2. Skala kapal/planet dan sistem jarak.
3. Weapon stats dan range evaluator dari data asli.
4. NPC target dummy + lock-on HUD + predictable ballistic projectile.
5. AI Targeting upgrade + auto approach / keep range / orbit + CANCEL.
6. Stasiun Earth & 3D docking; kemudian server-authoritative dua pemain.