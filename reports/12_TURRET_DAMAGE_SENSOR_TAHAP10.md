# Mapping Tahap 10 — Weapon Fire Controller, Firing Arc, Energy, Damage dan Sensor

**23 Oktober 2026 | Space RPG 4 v0.9995(44)**. Audit ulang langsung dari Smali asli `smali.zip` SHA-256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`. Ini **pemeriksaan statis, bukan uji runtime**. Source/A PK asli tidak disalin ke repositori publik ini; Godot tidak diubah.

Lanjutan: [Tahap 9 — NPC Combat](11_NPC_COMBAT_RANGE_TAHAP9.md), [Tahap 3 — Weapon Registry](05_WEAPONS_TAHAP3.md), [Tahap 2 — Hardpoint](04_HARDPOINT_TAHAP2.md).

## 1. Penemuan inti: **tracking dan firing adalah dua langkah berbeda**

- `Lw7.n -> Lf8` menyimpan weapon manager. `Lf8.h([I)V` membangun mount controller dari slot `Lu7.X` enam angka dan loadout `Lv7.b` (ID `-1` berarti kosong).
- `Lf8.a(J)V` mengurangi cooldown `Lf8.c` dan memanggil `Lzj.f(JLw7;)V` untuk **aim/transform tracking** tiap mount.
- **Yang memilih senjata siap tembak justru `Lf8.g()V`.** Ia membuat `TreeMap` berdasarkan timer ready `Lzj.n`, memproses mount menurut urutan, dan memanggil `Lzj.b(Lw7;)I`. Return code **2** menyebabkan penghentian loop salvo pada jalur itu.
- `Lf8.d()V` secara terpisah mengeksekusi `zj.b` untuk mount dengan `zj.a == 2`; ini penting agar CIWS/defense tidak ikut menghitung DPS anti-hull dari semua turret biasa.
- Subclass `Lzj`: `La5/Lf5` untuk keluarga `Lq6` projectile (beberapa perilaku spesial), `Lz4` untuk `Lq4` beam, `Lgi` untuk `Lei` torpedo. Untuk damage projectile fisik digunakan `Lr6` dan torpedo `Lfi` (plus subclass khusus).

### Source anchors (baris fisik dari ekstraksi decompile)
| File | Metode | Titik mulai |
|---|---|---:|
| `f8.smali` | `a(J)` / `g()` / `h([I)` | 60 / 970 / 1325 |
| `zj.smali` | `<init>` / `f(JLw7;)` | 67 / 1113 |
| `a5.smali` | `b(Lw7;)` | 486 |
| `z4.smali` | `b(Lw7;)` | 457 |
| `gi.smali` | `b(Lw7;)` | 272 |
| `e8.smali` | `a(J)` / `b(FFFFFF)` | 2145 / 3431 |
| `b8.smali` | `a(J)` | 20 |

## 2. Turret: actual 2D firing arc, bukan bebas berputar 360°

Di `Lzj`:
- `e/f`: offset mount dalam local 2D.
- `o`: pusat sektor turret relatif ke heading kapal.
- `p`: lebar sektor; `zj.f` menghitung batas `o ± p/2`, menormalkan sudut ±180°, dan memutar `Lu6.i` menuju target dalam limit.
- `y` dan modifier `f8.i` ikut mengatur laju rotasi; `z` termasuk status keselarasan aiming yang dipakai beberapa subclass `a()`; timer `n/l` untuk siap tembak.

**Contoh verified Tahap 2:** dua hardpoint O.S.C. Cruiser mempunyai arc **180°** menghadap sisi berlawanan. Data **2D tidak memiliki elevasi/pitch atau raycast hull 3D**.

**Keputusan untuk Godot:** setiap mount punya `Marker3D` base dan muzzle, limit yaw + pitch, tracking speed, serta pemeriksaan garis tembak. Raycast untuk memeriksa **occlusion oleh lambung sendiri tidak boleh mengecualikan seluruh collider kapal penembak**. Hindari muzzle dimulai di dalam hull; abaikan hanya bentuk turret/muzzle yang tepat bila aman.

## 3. Weapon range dan penembakan sebenarnya

`La5.b(Lw7;)I` untuk projectiles:
1. `La5.a()` menguji cooldown `zj.n > zj.l`, status keselarasan turret (atau pengecualian mekanis) dan batas salvo tertentu.
2. Butuh energi `Lb8.b > Lq6.m` serta cooldown group `Lf8.c[La5.R] <= 0`.
3. Pada cabang target terkunci, dihitung **`distance_squared(target,muzzle) < (zj.m)^2 + 2500`**, bukan jarak +2500. Kondisi ini spesifik cabang method dan tidak boleh diterjemahkan menjadi range semua senjata.
4. Menembak mengurangi power `q6.m`, dengan faktor **1.2** saat `f8.j` aktif; timer mount reset dan cooldown group diset dari `a5.G`. Bila `f8.l` aktif, terdapat cabang random **nextInt(333) == 1** yang dapat menyebabkan cooldown **2000** dan drain power penuh. Jangan menafsirkan ini sebagai peluang overload universal tanpa menguji prasyarat.
5. Null-target/manual-fire dan fixed-arc branches punya aturan berbeda.

`Lz4.b(Lw7;)I` untuk beam juga mengecek kesiapan, energy (konstanta **2** pada cabang utama), dan perbandingan `distance_squared < (zj.m)^2 + 2500` jika target dikunci. Beam membuat objek efek tembak `Lr4` dengan perubahan sudut yang mencakup `Random.nextGaussian()`; tidak benar mengklaim beam mustahil meleset dalam setiap kondisi runtime hanya dari deskripsi item.

`Lgi.b(Lw7;)I` untuk torpedo pada jalur ini mensyaratkan target. Untuk kategori `Lei.x`, kode mengevaluasi jarak minimum relatif **1.5 × panjang kapal** dan geometri orientasi depan. Tidak ada perbandingan maksimum `zj.m^2+2500` sederhana dalam method tersebut; jangkauan efektif dipengaruhi lifetime dan steering `fi`/subclass. **Di Godot 3D, tetapkan launch/lock max range secara eksplisit per missile**, jangan menyimpulkan misil boleh menembak tanpa batas hanya karena source mengecek kondisi lain.

## 4. Damage pipeline lebih jelas daripada Tahap 5

`Lw7` membuat state `Le8` dari `Lu7`:

| State | Sumber | Interpretasi static |
|---|---|---|
| `e8.b / e8.f` | `Lu7.w` | Shield aktif/maksimum; jalur regenerate `e8.a(J)` |
| `e8.c / e8.g` | `Lu7.v` | Hull aktif/maksimum |
| `e8.d[] / e8.h[]` | `Lu7.k[]` | Armor 4 sektor 2D aktif/maksimum |
| `e8.e` | `Lu7.l` | Faktor absorption/resist armor; unit/penamaan katalog masih perlu cross-check |
| `b8.b / b8.c / b8.d` | sistem power kapal | Energi aktif/regen/maksimum; `b8.a(J)` menghitung regen |

`Le8.b(FFFFFF)I` menerima empat kanal damage `p1..p4` dan dua nilai posisi impact `p5,p6` relatif kapal. `Lfi.m(Lw7;)I` meneruskan `Lei.g/h/i/j`; `Lr6` meneruskan kanal `Lq6` sendiri. Dari instruksi yang ditelusuri:

- `p1` (setelah modifier) mengurangi shield `e8.b`. Jika shield tembus, sisa negatif dipakai pada tahap berikutnya.
- Titik impact 2D `p5/p6` menentukan satu dari empat indeks armor `e8.d[index]`. Nilai `p3/p4` ikut rumus resist/armor; **belum ditetapkan nama fisik tiap kanal senjata secara final**.
- Damage yang lolos ke `e8.c` mengurangi hull. Ketika menembus ke dalam, ada loop yang dapat mengurangi integritas subsystem **`Lkh.b`**, memilih subsystem dengan RNG, dan mengubah efektivitas subsystem `Lkh.c`. Proses kehancuran kapal memanggil `w7.q()`.
- Ada pengecualian/player/fighter modifier, pengali **0.85**, **0.75**, dan **0.7** dalam beberapa cabang; belum sah menyematkannya pada upgrade tertentu hanya berdasarkan nilai literal.
- Shield dan energy adalah **berbeda**: senjata menggunakan energi `b8.b` untuk firing, damage melewati shield `e8.b`, lalu armor/hull.

Untuk 3D, tentukan sektor armor dari transform dan hit position/normal objek dalam local space. Hal ini merupakan desain baru; original memakai quadrant di bidang 2D.

## 5. Sensor ≠ jangkauan senjata

`Lf8.r` (dari Tahap 9) adalah **maksimum range agregat** mount tertentu (`zj.m`, dengan pengecualian weapon type 2) yang digunakan AI untuk menilai sasaran. Dari penelusuran sejauh ini **belum ada bukti numerik lengkap** bahwa `f8.r` adalah deteksi radar/sensor universal. Jangan menyalinnya sebagai `sensor_range` Godot.

Pisahkan **detection range**, **lock range**, **weapon effective range per mount**, dan **rendering distance**. Mapping khusus Grav Sensors/Analytic Sensors, NPC aggro range, dan LOS sensor masih pending.

## 6. Ketentuan AUTO ENGAGE untuk Godot 3D (desain disepakati)

```text
Target Lock (bukan Camera Lock)
  -> user toggles AUTO ENGAGE
  -> choose APPROACH / KEEP AT RANGE / ORBIT
  -> navigate with ship inertia/turn rate
  -> for each mount:
     valid/legal target and compatible equipped weapon
     target within THIS weapon range after allowed upgrades
     yaw & pitch facing, lead prediction and turret tracking
     muzzle ray not blocked by own hull/obstacle
     cooldown, power, ammo, launch conditions satisfied
     -> fire; physical projectile can miss, beam/missile have own models
  -> joystick/CANCEL/docking/invalid target immediately end auto NAVIGATION
```

Targeting AI improves prediction, **not hit guarantee, not weapon range, not penetration of own hull**. Range Extension applies only to eligible weapons per source catalog. Weapon damage and collision will ultimately be server-authoritative for multiplayer.

## 7. Test matrix — semua masih BELUM dijalankan
| ID | Test | Expected |
|---|---|---|
| T10-A | Side turret arc 180° target behind hull | Turret cannot fire through body |
| T10-B | Locked target beyond max range | No fire, regardless of aim |
| T10-C | Missile long range + cannon short range | Missile may fire first; cannon waits |
| T10-D | Evasive fast interceptor | Lead predictor still misses on sudden turn |
| T10-E | Low power or overload | Proper cooldown, no fire |
| T10-F | Shield depletes then armor/hull damage | Separate damage states respected |
| T10-G | Side turret 3D pitch outside range | No fire even if distance eligible |
| T10-H | Torpedo ECM/homing | Can diverge or miss |
| T10-I | Invalid destroyed NPC target | Safe retarget, no crashes |
| T10-J | Cancel Auto during thrust | Manual control with inertia preserved |

**Selanjutnya:** trace lengkap active sensor detection + `e8` damage modifiers & `zj` variant semantics, kemudian **satu Battleship vs satu Interceptor dengan dua mount nyata di Godot** setelah persetujuan implementasi. **Tidak ada build/test yang dilakukan pada Tahap 10.**
