# Space RPG 4 0.9995 — Mapping Tahap 4: Torpedo, guidance, ECM dan preset kapal
**23 Oktober 2026 — audit statis sumber smali.zip; belum diuji runtime.**

## Temuan source
- Registry h1.B() berisi **18 senjata torpedo Lei** (ID 61–78), mount tag 4. Field Lei.f = speed mentah, Lei.k = lifetime, Lei.K = derived range, Lei.n = jeda peluncuran yang diambil f8. Semua angka adalah satuan internal 2D.
- Controller f8 membaca Lei.n / Lei.o / Lei.K. Jalur gi, fi, jj, k7 memakai opsi Lei.x, Lei.N, Lei.O, Lei.P, Lei.t, Lei.s; **arti persis parameter obfuscated ini belum dipastikan**. Perlu trace formula homing dan ECM sebelum menyatakan hasil final.
- Constructor Lei default n=4000, t=64, v=100, J=32, L=3, O=60, P=1, Q=20; factory dapat mengoverride nilai ini per senjata.
- Deskripsi katalog strings.xml: ECM mengganggu kemampuan torpedo mengunci target, Advanced Guidance mengurangi interferensi 50% dan mencegah kehilangan lock; Range Extension +30% untuk projectile kecuali torpedo/beam; Targeting AI memperbaiki akurasi turret saja. Deskripsi tidak otomatis membuktikan semua formula runtime.

## Crosswalk preset kapal (Lv7.b ↔ Lu7.X)
- **92 factory kapal**; **238 assignment preset Lv7.b** ditemukan.
- **102** preset terbaca langsung dengan parser konservatif (fill-array-data ditulis ke register yang langsung menjadi Lv7.b). **136** belum terbaca, bukan berarti tidak ada.
- Semua **102/102** preset yang terbaca memiliki panjang array yang sama dengan jumlah hardpoint asli dari Tahap 2. Ini validasi struktur, bukan verifikasi equip runtime.
- Nilai -1 = slot tidak berisi senjata.

| Ship | ID | Contoh weapon IDs dari preset |
| --- | ---: | --- |
| Boreas | 72 | [9,9,9,11,11,33,33,69,69] |
| Kharon | 73 | [9,9,9,9,11,11,11,11,11,33,33,69,69,69,69] |
| F.E.R. Battleship | 33 | [9,9,9,9,21,21,21,21,33,33,33,33,-1,69,69] |
| O.S.C. Cruiser | 40 | [9,9,9,21,21,21,40,40,40,71] |
| Iro Titan | 55 | [10,10,10,10,10,10,10,10,24,24,24,24,24,24,24,24,45,45,45,45,45,51] |

## Contoh torpedo (range kandidat = speed × umur)
| ID | Speed | Lifetime ms | Range candidate | Launch interval candidate |
| ---: | ---: | ---: | ---: | ---: |
| 61 | 150 | 10000 | 1500 | 4000 |
| 64 | 270 | 10000 | 2700 | 4000 |
| 66 | 72 | 18000 | 1296 | 4000 |
| 71 | 204 | 10000 | 2040 | 1000 |
| 72 | 192 | 10000 | 1920 | 400 |
| 76 | 150 | 8000 | 1200 | 20000 |
| 78 | 330 | 10000 | 3300 | 550 |

Range tersebut kandidat lintasan lurus, **bukan jangkauan efektif** guided torpedo yang berbelok/menghindar/terganggu ECM. Jeda peluncuran pun masih perlu uji terhadap burst, ammo, energi dan overload.

## Implikasi 3D
- Camera Lock, Target Lock dan AUTO ENGAGE terpisah.
- Tiap senjata dievaluasi mandiri: target valid, kelas mount cocok, range setelah modifier, firing arc yaw/pitch, occlusion hull dan obstacle, turrets tracking, cooldown, amunisi, energi.
- Torpedo punya steering/homing/ECM, bukan hitscan. CIWS tidak otomatis menyerang hull.
- Targeting AI hanya memperbaiki prediksi, tidak menambah range atau membebaskan firing arc.
- AUTO ENGAGE dapat memilih APPROACH, ORBIT, KEEP AT RANGE dan dapat dibatalkan; momentum/inertia tidak direset. Saat docking, auto combat batal.
- Deskripsi lore original menyebut beam tidak meleset. Implementasi beam dengan line-of-sight dan kemampuan turret dalam 3D adalah keputusan remake yang perlu dibedakan dari mekanik asli.

## Belum selesai
Formula steering torpedo, ECM/nebula, amunisi dan reload, 136 preset tersisa, damage ke armor/shield, relasi station/planet Sol. Tidak ada perubahan pada proyek Godot atau APK pada Tahap 4.
