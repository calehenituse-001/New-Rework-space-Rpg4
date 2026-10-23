# F.E.R. Carrier — Rekonstruksi 3D untuk Godot v0.1 + Flight v0.2.8

**24 Oktober 2026 (sekitar malam 23–24 Oktober WIB).** Ini **asset 3D GLB asli**, bukan gambar konsep, dihasilkan dari model prosedural. **Belum dites runtime Godot Android**; sudah diuji validitas ZIP/GLB secara statis.

## Referensi yang digunakan
- Sprite asli dari archive user Space RPG 4 v0.9995: `res/drawable/ship_fer_battleship_carrier.png` sebagai referensi **tampak atas**.
- Konsep 3D F.E.R. Carrier yang telah disetujui dalam percakapan: lambung memanjang, anjungan bertingkat, area carrier/flight deck, dua sayap tengah, dan 5 engine belakang, merah gelap/LED teal.
- Bagian samping/bawah dibangun berdasarkan interpretasi teknis karena sprite asli hanya top-down; ini bukan model 3D official ataupun hasil scan.
- Kontrol game mengambil proyek Godot v0.2.7 (camera dan scale); versi integrasi menjadi **v0.2.8** dengan kapal kelima, tidak mengganti semua 4 kapal lama.

## Asset build
- Model: `FER_Carrier.glb`, **13** mesh/kelompok PBR, **18.976 triangle**, orientasi haluan `-Z`, atas `+Y`, ekor `+Z`.
- Ukuran native hasil hitung GLB: panjang **112,465**, lebar **73,2265**, tinggi **24,8** unit (bukan meter asli); di demo flight dikalibrasi menjadi panjang **78u** (faktor ~0,6935).
- Bukan sprite ekstrusi: keel bawah bertingkat, nacelle sisi, wedge-wing ber-volume, housing engine+nozzle melingkar, hangar bay terbenam dengan dekorasi fighter kecil, vent/panel teratur, dan strip LED emissive.
- **5 marker thruster** dan **3 marker hardpoint** + **2 marker hangar** tersedia pada `ships/FER_Carrier.tscn`.
- Script `BUILD_SOURCE.py` mereproduksi prosedural GLB via numpy+trimesh. Belum ada `.blend` karena Blender tidak tersedia pada lingkungan penyusunan.

## Paket yang dihasilkan (attachments percakapan, BUKAN di repository laporan)
1. `FER_Carrier_3D_Model_v01.glb` — asset mentah untuk import di Godot.
2. `FER_Carrier_Godot_3D_Showroom_v01.zip` — project standalone Godot 4.x, kamera mobile/desktop orbit, tombol TOP/FRONT/REAR/SIDE, import `.glb` otomatis.
3. `SRPG4_Sol3D_v0_2_8_FER_Carrier_3D.zip` — game prototype v0.2.7 ditambah kapal ke-5 F.E.R. Carrier, default ship baru, posisi mount test khusus, kamera limit lebih jauh.
4. `FER_Carrier_3D_Preview_v01.png` — preview render software, untuk QA desain saja.

## Perubahan kode flight
- `scripts/flight_ship.gd`: tambah path `res://ships/FER_Carrier.tscn`, profile ke-5, default index 4. Speed/turn v0.2.8 adalah **parameter TEST**, bukan statistik perkapal original.
- `scripts/sol_3d.gd`: keyboard 5 pilih kapal ke-5; kamera follow max clamp 240 (dari 125) agar kapal besar tidak terpotong.
- `scripts/combat_manager.gd`: jika model index 4, gunakan koordinat dua side cannon dan dorsal beam sesuai lokasi 3D native yang diskalakan.
- `scripts/combat_mount.gd`: untuk kapal ini gunakan 5 AABB approx proxy area hull (bukan satu AABB seluruh lebar sayap), agar sidemount tidak dianggap otomatis tertutup semua. **Tetap approximate, bukan pemeriksaan triangular mesh final.**
- Auto Fire, Auto Engage, Target NPC, kamera two-state, dan sistem Sol tidak sengaja diubah.

## Kekurangan / tahap validasi
- Hasil yang terlihat di Godot perlu dinilai: apakah siluet top match F.E.R. sprite dan tampak samping tidak terkesan sprite tebal; preview software tidak setara dengan in-engine lit scene.
- Kamera, tombol, stat ship, collider-box, orbit planet/ship scale masih parameter demo.
- Hangar belum berfungsi membawa fighter, tidak ada model rig animasi docking.
- Belum ada teksur PBR baku atau permukaan UV buatan artist, rig turret physics, LOD.
- Untuk di Android, perlu buka `project.godot`, impor GLB, main scene dijalankan, kemudian cek warna, kamera, frame rate dan muzzle posisi. **Belum dijalankan runtime**, jangan menganggap produksi siap.

## Rencana setelah tes
1. Evaluasi screenshot Godot top/front/rear/side, revisi geometri jika belum konsisten.
2. Kalibrasi scale kapal/planet/weapon visual bersama user, gunakan data original 92 kapal untuk rasio kelas.
3. Lanjut hardpoint senjata GLB 6 model dan integrasi Ship Outfitting setelah docking tersedia.
4. Tambah rig hangar/fighter dan LOD untuk Android.

Demi permintaan repo yang hanya laporan, **model 3D GLB dan ZIP tidak diunggah ke GitHub**.
