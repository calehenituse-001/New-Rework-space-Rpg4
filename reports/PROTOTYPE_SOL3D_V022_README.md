# Space RPG 4-inspired — SOL 3D Flight v0.2.2

**Isi ZIP ini proyek Godot 4, bukan APK.** Impor `project.godot` melalui Godot Editor Android, tunggu pengimporan GLB, lalu tekan Play. Mulai dengan mengimpor ke folder baru agar tidak tertimpa oleh v0.2.1.

## Yang berubah dari v0.2.1

**1. Kapal benar-benar berbeda ukuran dan karakteristiknya.** Model GLB Astra yang semula sekitar 20 unit panjang semuanya kini diskalakan sesuai class. Dimensi collision sederhana disesuaikan. Kecepatan, akselerasi, rem, kelincahan yaw/pitch, dan respons analog juga berbeda. Ini **tuning prototipe terinspirasi perbedaan kelas Space RPG 4**, belum pengambilan stat resmi untuk empat desain Astra.

| Model Astra | Class uji | Panjang dalam game | Max speed (unit/s) | Acceleration | Yaw deg/s | Pitch deg/s |
|---|---|---:|---:|---:|---:|---:|
| Berlian | Interceptor | 15 | 115 | 43 | 100 | 84 |
| Oktagonal | Frigate | 22 | 85 | 29 | 66 | 54 |
| Freighter Modular | Heavy Freighter | 32 | 43 | 16 | 35 | 28 |
| Carrier Garpu | Carrier | 42 | 32 | 11 | 21 | 17 |

**2. Kamera:**
- Swipe satu jari pada area kosong sisi kanan-atas/tengah layar untuk orbit kamera mengelilingi kapal (Free Look / ORBIT, yaw 360° dan pitch ±73°). Gerakan kapal tidak diarahkan oleh kamera.
- Cubit dua jari di area look untuk zoom; tombol **+** dan **−** di bawah juga mengatur jarak kamera.
- Double tap di area look atau tombol **RESET** untuk mengembalikan posisi kamera dan zoom default.
- Tombol **CAMERA** berputar antara **CHASE → FREE LOOK → TARGET**.
- Kamera otomatis memperlebar jarak untuk kapal lebih besar. Target camera menggeser fokus ke planet terpilih.

**3. Kontrol lebih halus:**
- Analog kiri: horizontal belok (yaw), vertikal angkat/turunkan hidung (pitch), *bukan* gerak mendadak ke arah joystick.
- Joystick memiliki deadzone, kurva analog, dan kontrol multi-touch tetap aktif bersamaan dengan THRUST/FIRE.
- THRUST maju, BRAKE memperlambat, FIRE menembak. Lepas THRUST tetap meluncur (inertia).
- Tombol **ASSIST** = batas pitch ±82° dan kembali tegak (default ON); matikan untuk rotasi lebih bebas.
- Tombol **PITCH STD/INV** = normal/invert sumbu pitch.
- Collider besar mengikuti kelas saat berganti model; posisi laser mengikuti panjang lambung.

**4. Gamepad Bluetooth/USB (mapping generik Xbox/Android):**
- Left stick: yaw / pitch.
- Right stick: Free Look.
- RT: thrust, LT: brake, RB: fire.
- LB: ganti target, X: ganti kamera, Y: ganti kapal.
- B: reset kamera, A: dock / undock.
- Mapping tombol bisa berbeda antargamepad Android. `PAD: CONNECTED` akan muncul di HUD bila terdeteksi.

**5. Keyboard/mouse editor:**
- W/A/S/D atau panah: pitch/yaw; Space thrust; Shift brake; F fire.
- Klik-drag pada area kosong tengah-kanan layar: Free Look; scroll: zoom.
- C camera, R reset camera, Tab target, E dock, 1–4 pilih kapal, I invert pitch, V flight assist.

## Fokus tes Android
1. Buka proyek baru dan Play; pastikan tidak ada merah pada tab Script/Debugger.
2. Tekan SHIP ▶ sambil diam dan bandingkan skala: Carrier harus jauh lebih besar daripada Berlian.
3. Bandingkan 10 detik THRUST setiap kapal (HUD SPEED); interceptor harus jauh lebih cepat, carrier terasa lambat.
4. Sambil THRUST + analog kiri, swipe area kosong kanan: kamera harus mengorbit tanpa mengubah arah kontrol kapal.
5. Tekan RESET, CAMERA bergantian, ZOOM +/- , ASSIST, PITCH.
6. Jika memiliki gamepad, coba dua analog + RT/LT/RB.
7. Coba docking ke Earth dengan kecepatan rendah; ambang aman docking diatur relatif class.

## Catatan
- Proyek **belum dijalankan/di-build** langsung dengan Godot di lingkungan pembuat paket. Pemeriksaan struktur dan integritas GLB dilakukan, tetapi hasil runtime di perangkat masih perlu uji.
- Koordinat / ukuran planet masih dummy, bukan Sol vanilla. Tidak ada combat damage, NPC, multiplayer, landing economy, maupun save.
- Empat file GLB milik pengguna tetap disertakan tanpa mengubah geometri asli (diskalakan lewat Godot saat runtime).
- Performa Godot Editor Android dan kontrol gamepad bisa bervariasi menurut perangkat; screen ke-2 adalah Play area Godot Editor bukan APK final.