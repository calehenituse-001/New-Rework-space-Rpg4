# Sol — Godot Prototype 0.1

**Status: kode sumber awal, belum diekspor sebagai APK dan belum diuji di runtime Godot pada lingkungan pembuatan ini.**

Ini adalah proyek **mandiri Godot 4.x (GDScript)** untuk bereksperimen dengan *satu sistem* bernama Sol sebelum migrasi fitur Space RPG 4 yang lebih besar. Tidak ada komponen multiplayer aktif pada v0.1. Tidak ada aset/kode/suara game asli yang dikemas di sini. Planet, jarak, orbit dan kapal adalah gambar prosedural/placeholder untuk menguji kamera dan gerakan, **bukan replika akurat** sistem Sol versi asli.

## Cara membuka di Android

1. Instal **Godot Editor 4.7.2** untuk Android (atau Godot 4.7.x untuk komputer).
2. Ekstrak ZIP hingga folder `Sol_Godot_Prototype_v0_1` berisi `project.godot`.
3. Buka Godot Editor > **Import** > pilih `project.godot` > **Import & Edit**.
4. Tekan tombol **Run Project (▶)**. Mode landscape direkomendasikan.
5. Jika hendak membuat APK, instal export templates di Godot, buat preset **Project > Export > Add > Android**, atur identitas aplikasi dan ekspor. Hasil APK belum termasuk dalam arsip ini.

## Kontrol

| Platform | Fungsi | Input |
|---|---|---|
| Android | Berputar kiri/kanan | Geser joystick kiri/kanan |
| Android | Maju + belok bersamaan | Geser joystick diagonal ke atas |
| Android | Dorongan lurus | Tahan THRUST kanan |
| Android | Memperlambat | Tahan BRAKE |
| Android | Tembakan visual | Tahan FIRE |
| Android | Zoom | `+` dan `-` kanan |
| Semua | UI docking | Dekati Earth lalu tekan DOCK EARTH / UNDOCK |
| PC | Putar | A/D atau panah kiri/kanan |
| PC | Maju | W atau panah atas |
| PC | Rem | S atau Shift |
| PC | Tembak | Spasi |
| PC | Zoom | E/Q, +/- atau roda mouse |
| PC | Dock/undock | F |

Gerakan kapal berinersia: setelah melepas thrust, kapal masih meluncur; rotasi tetap bisa dilakukan saat meluncur. Docking adalah **mockup UI**, bukan landing scene sungguhan. Tembakan baru efek visual tanpa collision/damage.

## Folder dan arsitektur

- `data/sol.json` — satu-satunya data tata letak prototipe. Setiap planet punya ID, orbit, sudut, ukuran, warna.
- `scripts/backdrop.gd` — starfield dan garis orbit.
- `scripts/planet.gd` — planet dan matahari yang digambar prosedural.
- `scripts/ship.gd` — flight model + sinyal perintah tembak, mudah dipisah dari sumber input nanti.
- `scripts/shot.gd` — peluru visual.
- `scripts/hud.gd` — multitouch joystick, kontrol Android, minimap, docking UI.
- `scripts/sol_world.gd` — load planet, spawn player, hubungkan scene.
- `scenes/sol.tscn` — scene utama.

## Langkah setelah tes pertama

1. Uji project benar-benar terbuka di Godot Android, FPS normal, joystick + multitouch + zoom.
2. Sesuaikan layout Sol dari data asli jika tersedia, baru ganti placeholder dengan aset yang berizin.
3. Uji transisi landing yang nyata dan kembali ke Sol.
4. Buat 2 player network dengan state server authoritative: klien mengirim input, server menghitung transform, klien menerima snapshot state.
5. Tambah combat, NPC, save, lalu sistem lain setelah arsitektur dua-player stabil.

## Catatan sumber referensi

Dari laporan mapping sebelumnya diketahui *Sol berada di wilayah F.E.R.*. Namun file sumber game asli yang memuat *koordinat planet* tidak tersedia langsung dalam pembuatan berkas ini. Karena itu `sol.json` tidak mengklaim penempatan orbital yang sama dengan versi asli.