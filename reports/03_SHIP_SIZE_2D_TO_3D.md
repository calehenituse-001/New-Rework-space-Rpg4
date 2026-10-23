# Audit ukuran kapal — Space RPG 4 v0.9995(44) & Prototype Godot Sol3D v0.2.3

**Status:** analisis sumber; **tidak mengubah game atau membuat build baru**. Data diambil dari berkas pengguna `smali.zip`, paket aset Space RPG 4 v0.9995(44), dan ZIP prototipe Godot v0.2.3.

## Inti temuan
- Template asli `h1.smali` mengisi `Lu7;->n:I` dan `Lu7;->o:I` dengan nilai berbeda menurut kapal. Berdasarkan perbandingan sprite dan penggunaannya pada runtime, keduanya merupakan **dimensi desain/render/collision 2D**, biasanya panjang (n) dan lebar (o) orientasi kapal; **bukan meter/kilometer kanonis**.
- Empat model Astra dalam GLB memang memiliki panjang dasar **hampir sama ~19,75–19,98 unit GLB**. Pada Godot, masing-masing **sudah diberi faktor skala berbeda** oleh `set_model()`—jadi ukurannya *di world-space* tidak sama.
- Alasan semuanya **terlihat sama besar di layar** ialah kamera di `sol_3d.gd` memakai `distance = clamp(ship_length * 2.7, 42, 125) * camera_zoom`. Saat ganti kapal, kamera ikut mundur sesuai panjang kapal; akibatnya proporsi kapal terhadap layar nyaris konstan.
- `Lu7;->A:F` pada statistik asli menunjukkan kemampuan terkait `Mass`/peralatan, **bukan konversi aman ke massa fisik 3D**. Hitbox perlu mengikuti geometri model 3D masing-masing, bukan hanya angka Mass atau ukuran file tekstur.
- Empat rancangan Astra adalah **kapal custom**, sehingga tidak punya pasangan nama/ID 1:1 yang bisa diklaim sebagai asli. Perlu mapping kelas referensi terpisah.

## Contoh angka ukuran dari Smali original (nilai relatif internal 2D)
| Kapal original | ID | n (panjang) | o (lebar) | Max velocity (B) | Turn rate (y) | Factory `h1` |
|---|---:|---:|---:|---:|---:|---|
| F.E.R. Fighter | 79 | 25 | 20 | 115 | 150 | `p0()` |
| F.E.R. Corvette | 37 | 50 | 20 | 100 | 180 | `m0()` |
| F.E.R. Frigate | 36 | 80 | 30 | 60 | 75 | `s0()` |
| F.E.R. Destroyer | 35 | 110 | 40 | 40 | 52 | `o0()` |
| F.E.R. Cruiser | 34 | 150 | 60 | 35 | 40 | `n0()` |
| F.E.R. Battleship | 33 | 240 | 100 | 30 | 30 | `h0()` |
| F.E.R. Battleship Mk2 | 89 | 240 | 100 | 30 | 30 | `i0()` |
| T.A. Battlecarrier | 90 | 200 | 140 | 34 | 31 | `k2()` |
| Kynigos | 71 | 64 | 32 | 85 | 90 | `Q1()` |
| Boreas | 72 | 150 | 60 | 40 | 40 | `N1()` |
| Kharon | 73 | 250 | 100 | 35 | 34 | `J1()` |
| Ares | 74 | 350 | 210 | 22 | 20 | `R1()` |
| Hermes | 94 | 410 | 178 | 35 | 28 | `S1()` |
| Iro Titan | 55 | 380 | 200 | 30 | 20 | `s1()` |
| Void Titan | 64 | 380 | 180 | 34 | 24 | `Q2()` |
| Quilyotida TI | 59 | 380 | 180 | 30 | 20 | `W1()` |
| Bulk Freighter | 14 | 100 | 100 | 30 | 27 | `N0()` |
| F.T. Conveyor | 15 | 120 | 80 | 54 | 40 | `O0()` |
| Tanker | 28 | 150 | 64 | 40 | 48 | `T0()` |
| Super Tanker | 29 | 200 | 100 | 35 | 35 | `S0()` |
| Salvager | 92 | 120 | 80 | 35 | 40 | `i1()` |
| Colony Ship | 75 | 60 | 35 | 35 | 63 | `M1()` |

Contoh validasi pada gambar sprite original (PNG transparan):
| Sprite PNG | Dimensi kanvas PNG | Area piksel tak transparan (lebar×tinggi) |
|---|---:|---:|
| `ship_player_frigate.png` | 64×64 | 34×59 |
| `ship_player_cruiser.png` | 256×256 | 82×189 |
| `ship_player_battleship.png` | 256×256 | 160×243 |
| `ship_player_titan1.png` | 512×512 | 206×348 |
| `ship_fer_frigate.png` | 128×128 | 38×82 |
| `ship_fer_battleship.png` | 256×256 | 158×245 |
| `ship_fer_corvette.png` | 64×64 | 22×44 |
| `ship_ta_battleship_carrier.png` | 256×256 | 130×172 |
| `ship_irohaem_titan.png` | 512×512 | 194×370 |

**Peringatan:** ukuran kanvas PNG (mis. 512×512) **bukan** ukuran nyata kapal. Kanvas mengandung banyak transparansi. `n/o` juga merupakan ukuran desain dalam ruang game 2D, bukan ukuran dunia nyata. Bentuk alpha, hitbox, dan artwork dapat berbeda.

## Kondisi empat kapal 3D pada tester saat ini
| Model 3D | Kelas tester | Panjang dasar GLB (arah Z) | Panjang world-space sekarang | Kecepatan maksimum tester | Jarak kamera CHASE (zoom=1) |
|---|---|---:|---:|---:|---:|
| Berlian | Interceptor | 19.87 | 15 | 115 | 42.0 |
| Oktagonal | Frigate | 19.75 | 22 | 85 | 59.4 |
| Freighter Modular | Heavy Freighter | 19.86 | 32 | 43 | 86.4 |
| Carrier Garpu | Carrier | 19.98 | 42 | 32 | 113.4 |

Pada panjang 22, 32, dan 42, rasio panjang kapal terhadap jarak kamera adalah sekitar **0,37** untuk semuanya; 15/42 juga ~0,36. Inilah penyebab utama kesan keempat kapal sama besar saat berganti model, **meskipun skala sebenarnya sudah berbeda**.

## Rekomendasi 3D untuk proyek (BELUM diterapkan)
1. Gunakan basis ukuran internal dari `n/o` original sebagai referensi **rasio**, bukan meter nyata. Untuk prototipe, konversi sementara **panjang 3D = n × 0,25** Godot unit (angka desain yang dapat diubah global).
2. Pilih pasangan kelas *benchmark* agar empat desain Astra mempunyai skala yang konsisten. Jangan memberi mereka ID kapal original tanpa pemetaan desain resmi.
3. Tentukan lebar/tinggi memakai proporsi model 3D hasil inspeksi bounding box GLB, tidak mengubah proporsi model secara paksa kecuali disetujui.
4. Camera CHASE gunakan rumus yang memberi variasi framing (misalnya **jarak = max(28, 1,5×panjang kapal + 20)** untuk uji awal, dengan smoothing dan zoom manual). Hindari kamera selalu 2,7×panjang jika ingin pemain merasakan perbedaan besar kapal.
5. Buat `comparison camera` opsional saat ganti kapal, dengan jarak world-space konstan, untuk membandingkan ukuran. Saat gameplay normal, kamera harus adaptif supaya kapal raksasa tetap nyaman dikendalikan.
6. Pisahkan visual model, collider (compound/convex sederhana), jarak docking, HUD lock-on, dan physics yaw/accel. Semua pakai dimensi dunia yang konsisten, bukan viewport pixel.
7. Ubah ukuran Earth/Moon/stasiun dan jarak Sol dalam langkah terpisah; **radius Earth 23 unit di tester sekarang** akan absurd kecil dibanding Titan asli versi 3D ~95 unit jika konversi 0,25 diterapkan.

### Saran mapping model Astra ke analog ukuran original (untuk diskusi, bukan kanon)
| Model custom | Pembanding dari game | n original | Panjang Godot jika ×0,25 | Catatan |
|---|---|---:|---:|---|
| Berlian | F.E.R. Fighter | 25 | 6.25 | Peran fighter/interceptor ringan; bukan model yang sama |
| Oktagonal | F.E.R. Frigate | 80 | 20 | Proporsi model GLB mendekati siluet sempit frigate |
| Freighter Modular | F.T. Conveyor | 120 | 30 | Acuan freighter sedang; heavy freighter dapat lebih besar |
| Carrier Garpu | T.A. Battlecarrier | 200 | 50 | Acuan carrier besar; identitas custom tetap dipertahankan |

Perbandingan resmi tambahan: Iro Titan n=380 → 95 unit, sedangkan Kynigos n=64 → 16 unit: panjang **5,94×**. Contoh fighter F.E.R. n=25 → 6,25 unit: Iro Titan **15,2×**. Ini hanya rasio dari ukuran internal asli yang dikonversi seragam.

## Appendix — semua 92 definisi pabrik kapal bernama yang ditemukan dalam `h1.smali`
| ID | Nama | n | o | Velocity B | Turn y | Mass stat A | Method |
|---:|---|---:|---:|---:|---:|---:|---|
| 4 | Crate | 30 | 20 | 80 | 150 | 10 | `j1()` |
| 6 | Courier | 50 | 35 | 65 | 63 | 20 | `P0()` |
| 7 | Mustang | 50 | 35 | 120 | 120 | 30 | `Z0()` |
| 8 | Reliant | 50 | 35 | 60 | 78 | 20 | `g1()` |
| 9 | Reliant Mk2 | 50 | 35 | 60 | 93 | 20 | `h1()` |
| 10 | Wander | 55 | 35 | 75 | 78 | 60 | `o1()` |
| 11 | Dawnstar | 55 | 35 | 65 | 82 | 80 | `Q0()` |
| 12 | Gryllidae | 55 | 35 | 70 | 120 | 50 | `U0()` |
| 13 | Bruiser | 50 | 36 | 65 | 72 | 50 | `M0()` |
| 14 | Bulk Freighter | 100 | 100 | 30 | 27 | 20 | `N0()` |
| 15 | F.T. Conveyor | 120 | 80 | 54 | 40 | 50 | `O0()` |
| 16 | Prospero | 64 | 48 | 60 | 72 | 50 | `f1()` |
| 17 | Onager | 80 | 60 | 45 | 54 | 140 | `b1()` |
| 18 | Nightstalker | 80 | 60 | 60 | 63 | 150 | `n1()` |
| 19 | Intrepid | 80 | 80 | 45 | 60 | 130 | `V0()` |
| 20 | F.T. Brabant | 80 | 80 | 40 | 45 | 120 | `L0()` |
| 21 | Venture | 104 | 68 | 50 | 28 | 220 | `m1()` |
| 22 | Valiant | 128 | 80 | 45 | 47 | 230 | `l1()` |
| 23 | Pioneer | 128 | 80 | 40 | 47 | 200 | `c1()` |
| 24 | Odyssey | 128 | 80 | 30 | 32 | 350 | `a1()` |
| 25 | Princeps Navis | 190 | 160 | 30 | 30 | 400 | `d1()` |
| 26 | Sovereign | 200 | 80 | 35 | 30 | 370 | `k1()` |
| 27 | Small Tanker | 80 | 55 | 50 | 63 | 10 | `R0()` |
| 28 | Tanker | 150 | 64 | 40 | 48 | 20 | `T0()` |
| 29 | Super Tanker | 200 | 100 | 35 | 35 | 30 | `S0()` |
| 30 | I.M. Reliant | 50 | 35 | 80 | 93 | 20 | `Y0()` |
| 31 | I.M. Onager | 80 | 60 | 60 | 54 | 140 | `W0()` |
| 32 | I.M. Princeps | 190 | 160 | 36 | 30 | 400 | `X0()` |
| 33 | F.E.R. Battleship | 240 | 100 | 30 | 30 | 500 | `h0()` |
| 34 | F.E.R. Cruiser | 150 | 60 | 35 | 40 | 380 | `n0()` |
| 35 | F.E.R. Destroyer | 110 | 40 | 40 | 52 | 150 | `o0()` |
| 36 | F.E.R. Frigate | 80 | 30 | 60 | 75 | 70 | `s0()` |
| 37 | F.E.R. Corvette | 50 | 20 | 100 | 180 | 30 | `m0()` |
| 38 | F.E.R. Runner | 80 | 30 | 145 | 75 | 50 | `j0()` |
| 39 | O.S.C. Battleship | 200 | 140 | 28 | 20 | 600 | `v1()` |
| 40 | O.S.C. Cruiser | 200 | 100 | 32 | 27 | 400 | `A1()` |
| 41 | O.S.C. Destroyer | 110 | 90 | 36 | 36 | 250 | `B1()` |
| 42 | O.S.C. Frigate | 60 | 50 | 48 | 48 | 100 | `E1()` |
| 43 | O.S.C. Corvette | 30 | 35 | 80 | 100 | 40 | `z1()` |
| 44 | T.A. Battleship | 200 | 140 | 34 | 31 | 530 | `j2()` |
| 45 | T.A. Cruiser | 180 | 100 | 37 | 43 | 390 | `o2()` |
| 46 | T.A. Destroyer | 128 | 60 | 45 | 54 | 180 | `p2()` |
| 47 | T.A. Aggressor | 70 | 70 | 37 | 42 | 200 | `i2()` |
| 48 | T.A. Scorpion | 62 | 46 | 60 | 75 | 80 | `t2()` |
| 49 | T.A. Frigate | 90 | 40 | 60 | 80 | 80 | `s2()` |
| 50 | T.A. Corvette | 40 | 24 | 105 | 165 | 30 | `n2()` |
| 51 | Dreadhulk | 150 | 100 | 50 | 54 | 350 | `F1()` |
| 52 | Scythe | 120 | 70 | 60 | 75 | 180 | `G1()` |
| 53 | Raider | 70 | 50 | 80 | 128 | 70 | `H1()` |
| 54 | Skiff | 32 | 20 | 105 | 180 | 30 | `I1()` |
| 55 | Iro Titan | 380 | 200 | 30 | 20 | 990 | `s1()` |
| 56 | Iro Battleship | 255 | 112 | 34 | 30 | 570 | `p1()` |
| 57 | Iro Cruiser | 184 | 92 | 37 | 40 | 300 | `q1()` |
| 58 | Iro Destroyer | 96 | 56 | 45 | 52 | 180 | `r1()` |
| 59 | Quilyotida TI | 380 | 180 | 30 | 20 | 940 | `W1()` |
| 60 | Quilyoboda BC | 174 | 100 | 35 | 36 | 520 | `T1()` |
| 61 | Quilyofirda FR | 66 | 44 | 65 | 86 | 100 | `V1()` |
| 62 | Quilyocorda CV | 30 | 20 | 100 | 150 | 30 | `U1()` |
| 63 | Kalo | 96 | 56 | 140 | 140 | 200 | `u1()` |
| 64 | Void Titan | 380 | 180 | 34 | 24 | 950 | `Q2()` |
| 65 | Void Battlecruiser | 174 | 100 | 40 | 40 | 580 | `N2()` |
| 66 | Void Interdictor | 174 | 100 | 24 | 28 | 340 | `P2()` |
| 67 | Void Frigate | 90 | 40 | 65 | 84 | 80 | `O2()` |
| 68 | Illuminator | 256 | 100 | 85 | 45 | 200 | `D()` |
| 69 | F.E.R. Flagship | 240 | 100 | 30 | 30 | 500 | `r0()` |
| 70 | Homebase | 240 | 100 | 15 | 30 | 100 | `t1()` |
| 71 | Kynigos | 64 | 32 | 85 | 90 | 80 | `Q1()` |
| 72 | Boreas | 150 | 60 | 40 | 40 | 320 | `N1()` |
| 73 | Kharon | 250 | 100 | 35 | 34 | 600 | `J1()` |
| 74 | Ares | 350 | 210 | 22 | 20 | 1150 | `R1()` |
| 75 | Colony Ship | 60 | 35 | 35 | 63 | 20 | `M1()` |
| 76 | Ruinous Predator | 325 | 218 | 30 | 9 | 700 | `E()` |
| 77 | Ruinous Executor | 346 | 400 | 20 | 4 | 900 | `C()` |
| 79 | F.E.R. Fighter | 25 | 20 | 115 | 150 | 10 | `p0()` |
| 80 | T.A. Fighter | 25 | 20 | 120 | 140 | 10 | `q2()` |
| 81 | O.S.C. Fighter | 25 | 20 | 105 | 140 | 10 | `C1()` |
| 82 | F.E.R. Bomber | 25 | 20 | 70 | 100 | 10 | `k0()` |
| 83 | T.A. Bomber | 25 | 20 | 75 | 90 | 10 | `l2()` |
| 84 | O.S.C Bomber | 25 | 20 | 65 | 90 | 10 | `x1()` |
| 85 | Type1-F Fighter | 25 | 20 | 105 | 140 | 10 | `v0()` |
| 86 | Type1-B Bomber | 25 | 20 | 64 | 100 | 10 | `t0()` |
| 87 | Merlin | 25 | 20 | 120 | 155 | 10 | `O1()` |
| 88 | Kingfisher | 25 | 20 | 74 | 105 | 10 | `K1()` |
| 89 | F.E.R. Battleship Mk2 | 240 | 100 | 30 | 30 | 560 | `i0()` |
| 90 | T.A. Battlecarrier | 200 | 140 | 34 | 31 | 550 | `k2()` |
| 91 | O.S.C. C-Battleship | 200 | 140 | 28 | 20 | 600 | `w1()` |
| 92 | Salvager | 120 | 80 | 35 | 40 | 100 | `i1()` |
| 93 | Retriever | 25 | 20 | 74 | 105 | 10 | `f0()` |
| 94 | Hermes | 410 | 178 | 35 | 28 | 940 | `S1()` |
| 95 | Hyperblock Probe | 30 | 30 | 1 | 150 | 1 | `d2()` |
| 96 | Rhino Princeps | 190 | 160 | 30 | 30 | 400 | `e1()` |
| 118 | Condor | 150 | 100 | 40 | 44 | 240 | `x0()` |

**Cakupan dan batasan:** 92 method factory `()Lu7;` bernama dalam `smali/h1.smali`, bukan bukti jumlah akhir seluruh varian runtime. Sebagian fighter, bomber, homebase, dan entitas khusus tidak boleh diasumsikan kapal yang dapat dibeli pemain. Registry `s8.smali` dan operasi khusus dapat membuat varian lain. Dimensi 2D tidak mengungkap dimensi tinggi 3D; tinggi harus ditentukan berdasarkan model Astra.

## Checklist sebelum implementasi
- Verifikasi pilihan mapping empat kapal custom → peran/kategori final.
- Tetapkan skala dunia terhadap planet, projectiles, dan jarak senjata.
- Modifikasi kamera tanpa mengubah mode LOCK stabil yang sudah diminta.
- Tampilkan siluet empat kapal berdampingan menggunakan kamera perbandingan.
- Hitbox mengikuti bounding mesh dan ditest saat docking/manuver.
- Setelah skala disepakati baru ubah Godot; tahap ini hanya audit.