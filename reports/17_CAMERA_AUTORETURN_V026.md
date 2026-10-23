# Godot Sol3D Combat v0.2.6 — Camera Auto Return on NPC Death

**Tanggal:** 23 Oktober 2026.  
**Status:** ZIP proyek Godot v0.2.6 dibuat, pemeriksaan statis selesai dan integritas ZIP lulus. **Belum diuji langsung di Godot Android.**

## Keluhan pengguna
AUTO FIRE dan AUTO ENGAGE sudah bekerja baik; targeting NPC dari v0.2.5 juga terlihat lebih baik. Tetapi ketika Pirate Interceptor mati saat kamera sedang TRACK NPC, kamera terasa **terkunci lurus** dan tidak bisa mengikuti/bergerak hingga tombol CAMERA ditekan lagi. Selain itu ukuran visual kapal tampak tidak proporsional, dan pengguna menanyakan potensi membangun model kapal 3D dari sprite Space RPG 4 asli.

## Root cause kamera
`scripts/sol_3d.gd:_update_camera` melakukan:

```gdscript
if camera_index == 3 and (combat == null or not combat.can_engage()):
    camera_index = camera_before_track
```

Saat TRACK NPC dimulai setelah mode CAM LOCK (2), `camera_before_track = 2`. Setelah NPC menjadi inactive saat mati, kamera dipulihkan ke mode **2 (LOCK world direction)** alih-alih **0 (CHASE)**, mengunci orientasi tetap. Ini menjelaskan laporan pengguna bahwa kamera tidak dapat mengikuti kapal hingga CAMERA ditekan.

## Perbaikan v0.2.6
- Method baru `_release_npc_track_after_target_lost()` memindahkan kamera TRACK NPC → **CHASE mode 0** saat target hancur/inactive/unlock.
- Clear stale touch-look state dan fokus tracking, reset `camera_before_track = 0`.
- Saat pemain menekan UNLOCK dalam TRACK, kamera juga kembali CHASE.
- **Manual CAM LOCK tanpa target tracking tidak diubah.**
- **Manual TRACK OFF saat target masih valid** tetap mengembalikan mode sebelumnya.
- Pilihan AUTO FIRE/AUTO ENGAGE, damage, laser, target NPC, collision, physics, default ukuran kapal tetap **tidak berubah**.

File ZIP yang dibuat: `SRPG4_Sol3D_Combat_v0_2_6_CameraAutoReturn.zip`; **tidak diunggah ke repo laporan**. File tersedia sebagai lampiran chat.

## Tes Android wajib (belum dijalankan)
1. TARGET NPC → TRACK NPC → AUTO FIRE/AUTO ENGAGE hingga Pirate mati: HUD menjadi `CAM: CHASE`, tanpa menekan CAMERA.
2. Uji dari urutan CAMERA→LOCK→TRACK NPC→musuh mati: tidak kembali ke CAMERA LOCK.
3. Tes FREE LOOK/CHASE tanpa track dan CAM LOCK manual: tidak berubah setelah musuh hancur.
4. NPC respawn: kamera tetap CHASE sampai TRACK dipilih lagi, tidak snap sendiri.
5. AUTO FIRE, AUTO ENGAGE, CANCEL, joystick tetap berfungsi.

## Next: Kalibrasi skala dan model kapal dari sprite
Prototipe sebelumnya memakai kelas custom Astra: Interceptor panjang 15, Frigate 22, Freighter 32, Carrier 42 unit Godot. Model GLB sendiri panjang asli sekitar 20 unit, tetapi kamera mengikuti `ship_length * 2.7`; karena kamera kapal besar lebih jauh, perbedaan ukuran tidak terasa dan berpotensi tampak aneh. Planet masih skala demo.

Rekonstruksi model kapal dari sprite original **bisa dilakukan sebagai model 3D sungguhan**, tetapi gambar hanya menunjukkan pandangan atas. Ketinggian lambung, underside, bentuk hangar, dan volume perlu dirancang/ditinjau; jangan mengklaim detail 3D yang tak ada pada sprite sebagai asli. Strategi: pilih satu kapal F.E.R. sebagai referensi, tetapkan siluet/rasio panjang:lebar, bangun low/poly midpoly 3D berketebalan aktual dengan normal/UV, tambah hardpoint turret, thruster/hangar, dan uji top-view terhadap sprite asli. Ekspor GLB untuk Godot dan sumber model teredit (mis. Blender script/.blend bila tersedia). **Belum membuat model konversi 3D dalam patch ini.**
