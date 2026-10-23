# Godot v0.2.7 — Unified SHIP CAM, AUTO TRACK target, Planet Scale Preview

**Tanggal:** 23 Oktober 2026.  
**Artefak:** `SRPG4_Sol3D_Combat_v0_2_7_Camera_PlanetScale.zip`, dibuat berdasarkan v0.2.6 yang telah diuji sebagian pengguna. ZIP ada dalam lampiran percakapan, **tidak diunggah ke repository khusus laporan**.  
**Validasi:** inspeksi kode, konsistensi struktur scene, integritas ZIP dan pengecekan matematis planet sphere; **belum dites dijalankan pada Godot Android 4.7.x.**

## Keputusan kamera dari masukan user
- Gabungkan **FREE CAMERA** dan **LOCK ke kapal sendiri** menjadi satu **SHIP CAM**: kamera bergerak mengikuti posisi kapal, mempertahankan arah pandang stabil ketika kapal bermanuver, tetapi tetap dapat diorbitkan lewat usapan layar/analog kanan, plus zoom/pinch.
- Tekan **TARGET NPC** atau **LOCK NPC** membuat combat target terkunci **dan** kamera langsung TRACK NPC.
- Tombol **CAMERA** beralih `SHIP CAM ↔ TRACK NPC` tanpa membatalkan target lock, AUTO FIRE atau AUTO ENGAGE.
- Tombol **UNLOCK** (bukan CAMERA) yang benar-benar membebaskan combat target. Musuh hancur/lost otomatis memulihkan SHIP CAM yang tetap bisa di-swipe. Setelah respawn tidak otomatis men-track lagi.
- Hapus tombol `LOCK OFF` dan `TRACK NPC` yang kini redundant, tetap ada `RESET VIEW`, zoom dan ASSIST.
- Target planet hanya bagian `MAP / NEXT PLANET`. Kamera/target combat tidak dikaitkan dengan navigasi planet.

### Source Godot changed
- `scripts/sol_3d.gd`: kamera dua state (0 SHIP CAM, 1 TRACK NPC), input dan return setelah target hilang, planet-scale coordinates, kamera far plane, re-position Earth spawn/undock berdasarkan jari-jari Earth.
- `scripts/flight_hud.gd`: tombol/teks kamera digabung, status NAV dan TARGET tetap terpisah.
- `README_CAMERA_SCALE_v0_2_7_ID.md`, `CHANGELOG_v0_2_7.txt`, `PLANET_SCALE_PREVIEW_v0_2_7.json`.

## Planet scale (Godot units, usulan skala gameplay kompresi sci-fi)
| Objek | Radius v0.2.6 | Radius v0.2.7 | Diameter v0.2.7 |
|---|---:|---:|---:|
| Sun | 52 | 2050 | 4100 |
| Mercury | 9 | 82 | 164 |
| Venus | 15 | 180 | 360 |
| Earth | 23 | 230 | 460 |
| Moon | 6 | 32 | 64 |
| Mars | 13 | 155 | 310 |
| Jupiter | 45 | 1230 | 2460 |
| Saturn | 39 | 1050 | 2100 |
| Uranus | 25 | 620 | 1240 |
| Neptune | 26 | 610 | 1220 |

- Carrier Astra panjang 42u → Earth diameter kira-kira **10.95×** Carrier, Jupiter diameter **58.57×** Carrier.
- Titan future proposed length 95u > Moon diameter 64u namun jauh lebih pendek daripada Earth diameter 460u. **Ini tidak mengikuti rasio astronomi fisik**, dan angka Titan baru konsep.
- Orbit positions dibangun ulang (contoh: Earth (4400,0,0), Moon (4860,25,-28)); test analitik sphere dan radius ring Saturn terdekat menyisakan jarak positif sekitar **199.53u** antara Earth dan Moon, jadi tidak ada overlap radius collision pada tabel statis.
- Camera far clip 50,000u; bintang dipindah lebih jauh; kapal spawn relatif di luar radius Earth untuk menghindari muncul di dalam planet. Hyperwarp/LOD/station masih perlu penyesuaian pada langkah berikutnya.
- Ini hanya **referensi visual** terinspirasi cinematic space / EVE Online, **bukan** meniru nilai meter/diameter resmi EVE atau SRPG4; planet 3D demo juga **bukan** sistem Sol original v0.9995 (yang memiliki Earth/Luna/Agrodome).

## Tes yang perlu dilakukan user
1. TARGET NPC -> kamera otomatis TRACK NPC; AUTO ENGAGE masih berfungsi.
2. CAMERA -> SHIP CAM, target tetap LOCKED. Swipe untuk orbit walau kapal bermanuver.
3. CAMERA lagi -> TRACK NPC, tidak butuh TARGET ulang.
4. UNLOCK -> combat target release dan kamera kembali SHIP CAM.
5. Bunuh Pirate saat kamera TRACK -> otomatis SHIP CAM dapat di-swipe tanpa tekan CAMERA.
6. Periksa Earth tampak lebih besar dari Carrier dan tidak ada spawn/collision di dalam planet.
7. Undock dan kamera/auto fire tetap berfungsi; periksa kinerja Android dan jarak render untuk planet luar.

## Catatan aset kapal dan batas pengembangan
- Model Astra lama **belum diganti**. Ukuran panjangnya tetap 15/22/32/42u dari fixture sebelumnya; masalah framing semua kapal terasa hampir sama besar masih butuh eksperimen camera-distance per kelas.
- Untuk sprite SRPG4 → mesh 3D: wajib mempertahankan siluet tampak atas dan proporsi referensi; bangun lambung bertingkat/kurva volume sesuai faksi, detail bagian samping, underside, front/rear dan nacelle terpisah, **jangan hanya ekstrusi sprite**. Bagian bawah adalah desain baru yang harus dinilai bersama user, bukan detail resmi terbukti dari sprite.
- **Belum dibuat model kapal SRPG4 3D baru** pada patch v0.2.7.
