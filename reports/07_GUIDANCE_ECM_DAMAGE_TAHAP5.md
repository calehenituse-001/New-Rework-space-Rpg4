# Space RPG 4 — Mapping Tahap 5: Guidance, ECM dan Damage Pipeline
**Audit 23 Oktober 2026 | Source v0.9995(44) | STATIC ONLY — belum diuji runtime**

Sumber: `smali.zip` SHA-256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`, `public.xml` dan `strings.xml` dari source decompile. Lanjutan dari [Tahap 4](06_TORPEDO_PRESETS_TAHAP4.md). Angka internal 2D bukan meter/km, dan mapping ini tidak memodifikasi Godot / APK.

## Temuan baru yang berhasil ditelusuri
| Bukti | Makna dan status |
|---|---|
| `h1.z2()`, item **207**, `h1.E` | **ECM**. Crosswalk melalui `La4.a` dan string title: CONFIRMED |
| `h1.K2()`, item **208**, `h1.F` | **Targeting AI**: upgrade turret accuracy, bukan bonus jangkauan: CONFIRMED katalog |
| `h1.u2()`, item **216**, `h1.N` | **Adv. Guidance**: CONFIRMED |
| `w7.smali` item `h1.E` → `Lw7.Q` | Flag ECM dipasang pada target: CONFIRMED static |
| `w7.smali` item `h1.N` → `Lf8.m` | Flag Advanced Guidance pada kapal penembak: CONFIRMED static |
| `gi.b(Lw7;)I` | Spawn torpedo menggunakan gangguan sistem, ECM target, Adv Guidance shooter: CONFIRMED static |
| `Lfi.v`, `Lfi.w`, `Lfi.x` | Intensitas gangguan, offset arah, timer; trace `fi.k(J)`: kuat, exact semantics seluruh cabang perlu uji |
| `Lfi.A` | Flag dari `f8.m` untuk melewati cabang defleksi/loss tertentu; trace `gi.b` + `fi.k`: kuat |
| `Lei.N` | Flag klasifikasi guidance dipakai UI `jj.e()` dan runtime `fi`, namun pemetaan true/false per senjata belum final |
| `Lei.O` | Faktor koreksi belokan di `fi.n()`; besaran unit/konversi Godot belum final |
| `Lk7 extends Lfi` | Torpedo khusus punya cabang update/steering sendiri, jangan pukul rata seluruh tipe |

## Jalur interferensi yang terlihat dari source
Pada `gi.smali`, method `b(Lw7;)I`, sekitar baris fisik 610–650:

1. Membaca `k8.l` dari sistem aktif (**kandidat komponen gangguan Nebula Haze**, masih perlu cross-check).
2. Menambahkan **30** jika target `w7.Q` (ECM) aktif.
3. Membagi **2 menggunakan integer division** jika penembak `f8.m` (Adv Guidance) aktif.
4. Melewatkan nilai integer tersebut ke argumen terakhir `fi.<init>(Lw7;Lei;FFFFI)` (termasuk saat membangun varian `k7`).
5. Menyalin `f8.m` ke `fi.A` untuk pengaruh tambahan selama penerbangan.

Sketsa logika, bukan kode verbatim:

```text
interference = system_interference
if target_has_ecm: interference += 30
if launcher_has_advanced_guidance: interference = integer_divide(interference, 2)
torpedo = launch(interference)
torpedo.advanced_guidance = launcher_has_advanced_guidance
```

`fi.k(J)` memakai random offset dan periode gangguan; cabang tanpa `fi.A` dapat memicu penyimpangan besar (offset ±100) dan cabang gangguan lock. Ini konsisten dengan deskripsi Adv Guidance, tetapi **jangan menyimpulkan probabilitas loss final atau '100% hit'** tanpa trace penuh/runtime. `fi.n()` memakai posisi target dan faktor `Lei.O` untuk perubahan arah, kemudian memasukkan offset `fi.w`. Subkelas `k7` punya perlakuan khusus.

## Jalur damage yang sudah terbukti
`fi.m(Lw7;)I` memanggil `Le8;->b(FFFFFF)I`. Empat argumen float berasal dari kanal `Lei.g/h/i/j`, dua lainnya posisi tumbukan relatif terhadap orientasi target. Ada cabang efek khusus berdasarkan `Lei.a`, serta pengecualian untuk relasi kapal tertentu. **Belum diverifikasi** kanal mana damage hull/shield/armour/penetration dan modifier akhir, jadi jangan mengunci interpretasi sampai `e8.smali` ditelusuri.

## Semua 18 torpedo dan batas range
Registry tahap 4 memberi torpedo dasar ID **61–78**. `range_candidate_internal = speed_internal × lifetime_ms / 1000` adalah estimasi **lintasan lurus**, bukan range efektif guided; jangan samakan dengan hit chance. Guidance flag per ID belum terbukti, sehingga **tidak dikarang**. Lampiran CSV di `data/torpedo_18_tahap5.csv` mencatat kecepatan/lifetime/range kandidat beserta uncertainty.

## Implikasi desain Godot (bukan mekanika original terverifikasi)
- **CAMERA LOCK**, **TARGET LOCK**, dan **AUTO ENGAGE** adalah tiga subsistem independen.
- AUTO ENGAGE hanya mengendalikan navigasi jika diperintahkan, dapat dibatalkan kapan pun tanpa menghilangkan inertia, dan berhenti saat docking.
- Tiap senjata: target valid, rentang efektif setelah modifier, arc yaw/pitch, line of sight, occlusion lambung 3D, kemampuan turret tracking, cooldown, ammo/energi; baru fire.
- Missile guidance perlu status TRACKING / INTERFERED / LOST_LOCK / EXPIRED / IMPACT. ECM dan Adv Guidance memengaruhi guidance, **bukan bypass jarak**.
- Hit dan damage 3D ditentukan collision; server authoritative nantinya memvalidasi hit/damage. AI targeting turret meningkatkan kualitas prediksi, bukan auto-hit.
- CIWS tidak dihitung otomatis sebagai senjata ofensif terhadap hull. Beam/hitscan butuh logika terpisah.

## Rencana tes (belum dilakukan)
| Uji | Target ECM | Penembak Adv Guidance | Harapan |
|---|---|---|---|
| T5-A | off | off | interference lingkungan |
| T5-B | on | off | interference +30 |
| T5-C | on | on | hasil T5-B dibagi 2 secara integer |
| T5-D | off | on | interference lingkungan dibagi 2 |
| T5-E | variasi | variasi | musuh bermanuver bisa tetap menghindari torpedo |
| T5-F | variasi | variasi | hardpoint tertutup badan kapal tidak menembak |

## Belum selesai
- Tracing `Le8.b` hingga pembagian hull/shield/armour dan modifier damage, ammo/burst/cooldown akhir.
- Identifikasi guidance full/semi per ID 61–78 + varian `k7` dari factory.
- Konfirmasi `k8.l` di Sol dan sistem nebula lain.
- **136** preset loadout dengan propagasi register rumit dari Tahap 4.
- **Tahap 6 selanjutnya:** mapping Sol, Earth, Luna, planet, station/landing & docking original vs usulan EVE-style.

**File lokal lengkap:** `SRPG4_Mapping_Tahap5_Guidance_ECM_Damage.zip` (arsip laporan dan crosswalk). APK/source Smali asli tidak diunggah ke repo publik ini.
