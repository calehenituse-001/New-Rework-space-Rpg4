# Mapping Tahap 7 — NPC, Factions, Spawn, AI Combat (Space RPG 4 v0.9995)
**23 Oktober 2026** | Audit statis `smali.zip` SHA256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`. **Belum diuji runtime; tidak ada APK/Godot yang dimodifikasi.**

## Arsitektur NPC original hasil trace
AI tidak berada dalam satu kelas. `w4.smali` membuat registry faksi; `k2.smali` menyimpan hubungan; `y7.smali` menangani set ancaman/target; `m3.smali` menghasilkan NPC per faksi dan formasi; `g1.smali` memicu wave/encounter; `x7.smali` mengatur kursus/motion; `w7.smali` menjadi runtime ship; `d8.smali` mengelola fighter dari hangar; `zj/f5/z4` menangani targeting turret/projectile/beam.

| Class / method | Temuan terverifikasi secara statis |
|---|---|
| `w4.pswitch_4` baris ~4248–4836 | Constructor 12 faksi ID 0–11, label dan string relasi Y/N; ID 8 memakai register label `Independent` yang reused |
| `k2.c(Lk2;)`, `a(I)`, `b(I)` | Hasil `true` jika `this.e[other.b]=='Y'`; mutator `a` menetapkan Y dan `b` menetapkan N. Relasi **berarah dan bisa berubah** |
| `y7.h(Lw7;Z)` baris ~2230–2372 | Cek hubungan faksi dengan kondisi player/parent, lalu dapat memasukkan target ke set |
| `y7.b(Lw7;)` dan `a(J)` | Threat LinkedHashSet; perambatan respons dan cleanup referensi target tak valid |
| `y7.g()` | Salah satu jalur pemilihan dari candidate target set memakai random; bukan bukti semua target dipilih acak |
| `m3.n(Lk2;Lu2;FFZI)` | Cabang ID faksi, random ship-template selection (Lu7), termasuk pemilihan grup |
| `m3.l/m/k` | Pembuatan array ship state La8, posisi formasi sebagian random, registrasi instance w7 |
| `g1.a(J)` | Timer/condition encounter wave dan spawn via m3 |
| `x7.a(J)`, `w7.k(J)` | Motion/course controller dan update state/turret kapal, bukan keseluruhan battle decision |
| `d8.a(J)` | Hangar-fighter controller, fighter child ship, link parent `w7.I/J`, counter dan situasi encounter |
| `zj/f5/z4` | Turret tracking/prediction senjata; harus terpisah dari AI manuver dan hardpoint 3D |

## Faksi dan matriks hubungan (source literal)
`k2.c` membaca 'Y' di indeks ID faksi target. Pemakaiannya di `y7.h` terkait penambahan target, tetapi **tidak semua nilai Y otomatis berarti kapal akan langsung menembak**; ada pemeriksaan state dan kondisi lain. Jangan asumsikan matriks simetris.

| ID | Faksi | Mask Y/N mentah (panjang 19–20) |
|---:|---|---|
| 0 | Player | `NNNNNYYYNNYYYYYYYYY` |
| 1 | Independent | `NNNNNNNNNNNNNNNNNNN` |
| 2 | F.E.R. | `NNNYYYYYNNYYYYYYYYY` |
| 3 | T.A. | `NNYNYYYYNNYYYYYYYYY` |
| 4 | O.S.C. | `NNYYNYYYNNYYYYYYYYYY` |
| 5 | Pirate | `NNYYYNYYYYYYYYYYYYYY` |
| 6 | Irohaem | `NNNNNYNYNNYYYYYYYYYY` |
| 7 | Quilentya | `NNNNNYYNNNYYYYYYYYYY` |
| 8 | Independent (ID 8, label from reused register) | `NNNNNYNNNNNYNNNNNNN` |
| 9 | Kalo | `NNNNNNNNNNNYNNNNNNN` |
| 10 | Void Raiders | `YYYYYYYYYYNYYYYYYYYY` |
| 11 | Death | `YYYYYYYYYYYNYYYYYYYY` |

**Caveat:** 12 entri teridentifikasi, sementara beberapa mask mempunyai karakter posisi 12–19. ID8 tampaknya memakai label `Independent` yang sama melalui register yang digunakan ulang; penamaan finalnya perlu validasi runtime. Event cerita/misi dapat mengubah relasi melalui mutator `k2.a/b`.

## Sol dan encounter
Dari Mapping Tahap 6, owner Sol adalah F.E.R. (faction ID 2); *raw spawn weights* **Independent 45, F.E.R. 45, T.A. 5**. Jumlah 95 bukan probabilitas akhir. Distribusi final masih bisa dipengaruhi transformasi, penyaringan waktu/cerita, dan pilihan kapal dari `m3`. Jangan masukkan 45%,45%,5% secara mutlak ke Godot.

## Rancangan Godot (USULAN, bukan port persis)
- `FactionDefinition`: hubungan directional Y/N yang dapat diubah event/save.
- `EncounterSpawner`: distribusi setelah normalisasi, filter safe-zone, level, faction, formation; bukan memunculkan armada acak setiap frame.
- `NpcShipBrain`: state `PATROL/ALERT/ENGAGE/REPOSITION/DISENGAGE` (state buatan). **FLEE original belum terbukti.**
- `ShipFlightController3D`: yaw, pitch, roll, inertia, max speed, turn rate dan nav path.
- `WeaponMount3D`: valid target, range per senjata+modifier, yaw/pitch firing arc, hull occlusion, turret tracking, cooldown, energi dan ammo.
- `FighterWingController`: launch/return fighter untuk Carrier, tiap fighter independen dan punya parent.
- `PlayerAutoEngage`: terpisah dari NPC brain, `TARGET LOCK` maupun `CAMERA LOCK`. Perintah APPROACH/ORBIT/KEEP RANGE untuk combat saja, bisa dibatalkan joystick atau CANCEL tanpa reset momentum. Docking memutus AUTO ENGAGE.

Tidak ada tembakan 100%: proyektil fisik memakai leading, Targeting AI menambah kualitas prediksi bukan jaminan hit; beam dan torpedo beda model, ECM berlaku untuk guidance. Kapal berat lebih mudah mengenai target karena collision, tidak dengan bonus hit chance otomatis.

## Rencana tes (BELUM DIUJI)
`T7-A` faksi F.E.R. vs Pirate; `T7-B` sesama faksi; `T7-C` target mati/hilang; `T7-D` fighter carrier spawn/parent; `T7-E` turret samping blocked by hull; `T7-F` cancel auto dan inertia; `T7-G` max range per hardpoint; `T7-H` encounter Sol dan filter cerita/safe zone.

**Yang belum dipetakan:** deteksi sensor/aggro range original, urutan pemilihan target lengkap, flee/disengage asli, prioritas escort/formation, transformasi raw spawn weight menjadi peluang final. Target Tahap 8 ialah trace decision tree/aggro sebelum uji NPC Godot.

**Dataset:** [faction registry](../data/faction_registry_12_stage7.json) dan [matrix CSV](../data/faction_relation_matrix_12x12_stage7.csv). Tidak ada Smali lengkap/APK/aset game original di repo ini.
