# v0.2.9 — F.E.R. Earth Orbital Dock & Ship Outfitting (Godot)

**Tanggal:** 24 Oktober 2026  
**Status:** proyek Godot telah dimodifikasi dari v0.2.8 dan paket ZIP dibuat serta diuji integritasnya. **Belum diuji runtime di Godot Android** karena Godot executable tidak tersedia di lingkungan pengerjaan.

## Keputusan pengguna
- Semua perubahan kamera v0.2.7 dinyatakan **aman/lulus tes** oleh pengguna; kode kamera tidak diubah lagi.
- Pengerjaan seni/sprite/model kapal **ditunda**. Tidak ada kapal baru atau perubahan geometri lama di versi ini.
- Lanjut mekanisme game yang sudah direncanakan: orbital docking dan Ship Outfitting.

## Implementasi baru
1. **Orbital dock F.E.R. dekat Earth**, dibentuk dari primitive 3D Godot: core, dua cincin station, docking arms, emissive guide, docking portal dan nameplate. **Bukan** model original SRPG4.
2. **Pemeriksaan docking:** jarak ke station <= 170 unit Godot dan kecepatan < `min(38, ship.max_speed*0.37)`; tidak lagi memeriksa landing point permukaan Earth sebagai satu-satunya kondisi docking.
3. **Animasi pendekatan** 1,1 detik sebagai posisi tween, dilanjutkan perpindahan ke **ruang hangar 3D** dekoratif. Ini **bukan** pathfinding/traffic dan docking animation final.
4. Saat docking: pemain berhenti, Auto Engage dan Auto Fire dimatikan, combat target dilepas, kamera SHIP CAM tetap normal.
5. **Ship Outfitting dalam hangar:** tiga mount test `PORT`, `STARBOARD`, `DORSAL`, masing-masing bisa dipilih `CANNON`, `BEAM`, atau `EMPTY`. Pilihan mengubah **logic penembakan sungguhan** (tipe, range, cooldown, damage) melalui rebuild mount, bukan hanya mengubah label UI.
6. **Penyimpanan lokal per ship index** lewat `user://fer_outfitting_v029.json`. Save tidak mengklaim kompatibel dengan save original ataupun multi-server. Validasi multiplayer nantinya harus server-authoritative.
7. **Tombol Repair development test** untuk shield/hull. Undock mengembalikan kapal ke area station dengan loadout.
8. Enemy respawn ditahan selama pemain docked supaya AI tidak muncul di dalam hangar.

## File yang ditambah / diubah
- `scripts/orbital_station.gd`: mesh station.
- `scripts/hangar_interior.gd`: mesh hangar 3D statis.
- `scripts/outfit_save.gd`: baca/tulis loadout JSON lokal dan validasi kind.
- `scripts/combat_manager.gd`: loadout per-ship, reconfigure mount, repair development.
- `scripts/flight_hud.gd`: 3 OptionButton slot persenjataan, repair, undock.
- `scripts/sol_3d.gd`: integrasi station/hangar/docking cinematic; SHIP CAM logic disalin tanpa modifikasi.
- `scripts/combat_npc.gd`: respawn pause selama docking.
- `project.godot`: nama versi 0.2.9; `README_TEST_DOCK_OUTFITTING_v029_ID.md` dan `CHANGELOG_v0_2_9.txt`.

## Cara pengujian di Android (BELUM DILAKUKAN)
1. Buka proyek ZIP di Godot Editor Android lalu ▶ Play.
2. Di lokasi awal dekat Earth, temukan objek bernama F.E.R. ORBITAL DOCK; saat kapal diam tombol **DOCK STATION** seharusnya tampil.
3. Tekan tombol dan tunggu animasi pendekatan singkat. HUD Hangar → PORT/STBD/DORSAL.
4. Pasang `EMPTY` untuk satu mount; undock; lock Pirate NPC, nyalakan AUTO FIRE; jumlah tembakan harus turun untuk mount yang dikosongkan.
5. Dock ulang dan pasang BEAM/CANNON. Bandingkan karakter tembakan.
6. Tutup, buka kembali proyek dan cek pilihan slot tersimpan di user://.
7. Periksa tombol TARGET NPC, CAMERA, AUTO FIRE dan AUTO ENGAGE di luar dock tetap seperti versi sebelumnya.
8. Kirim screenshot error parser/output atau foto hangar bila ada masalah.

## Batasan, risiko dan berikutnya
- Tidak ada Blender/GLB custom station, portal door yang bergerak, collision station penuh atau autopilot approach yang memakai kinematika asli. Ini fixture untuk menguji state dock/undock.
- Belum ada Commerce, inventori, harga item, volume cargo, Shipyard/Garage, alat upgrade original, amunisi/energy/overload, atau mod multiplayer.
- Tiga slot adalah **fixture test**, bukan mapping final semua hardpoint game original atau hak equip semua slot untuk semua tipe kapal.
- `user://` bersifat lokal; server nantinya harus memvalidasi equip, remove, persistence, damage.
- Static checks yang dilakukan: referensi preload, keberadaan project.godot/scenes, sinyal callback, kelengkapan 5 GLB, bracket count untuk script changed, ZIP CRC. **Bukan tes Godot/GDScript parser resmi**.
- Tahap berikutnya **setelah tes Android:** debugging layout & docking, shop/inventory yang mengaitkan jenis senjata ke data mapping original, lalu cinematic approach dengan flight physics.

**Artefak percakapan:** `SRPG4_Sol3D_v0_2_9_EarthDock_Outfitting.zip`. Tidak diunggah ke repository yang memang khusus dokumentasi.
