# Status pengujian Space RPG 4 Full 3D — 23 Okt 2026

## Teruji melalui pengguna di Godot Android
- Prototype Sol 3D v0.2.1 berjalan setelah perbaikan script; model GLB muncul, kapal dapat terbang dan menembak.
- v0.2.2 diuji; ditemui analog terlalu kecil, jarak FIRE/BRAKE sempit, mode kamera salah arah, dan blur/jitter saat thrust.
- v0.2.3 berisi perbaikan UI kamera LOCK/joystick; pengguna menyebut versi terbaru sudah lumayan bagus.
- **Belum terbukti teruji**: NPC + AUTO ENGAGE, turret nyata, docking 3D, torpedo ECM, server multiplayer Godot, serta skala planet baru.

## Desain yang telah disetujui
Full 3D Godot, flight assist + inertia, kamera stabil/free-look, skala kapal menurut kelas dan source original, ukuran planet raksasa fiksi. Target Lock untuk musuh, AUTO FIRE/AUTO ENGAGE yang bisa dibatalkan, predictive lead (Targeting AI bukan auto-hit), weapon range dan firing arc 3D plus occlusion hull, docking orbital/hangar ala EVE, pilihan hangar capital/Titan mega dock.

## Lanjutan
1. Lengkapi torpedo/ECM/guidance, damage, reload, preset ship.
2. Peta posisi & aset Sol/Earth/Luna, fasilitas docking dan planet.
3. Integrasi dua turret 3D + target dummy untuk collision dan predictive aim.
4. AUTO ENGAGE, NPC, docking, baru multiplayer server-authoritative.

Source APK, Smali penuh dan model GLB tidak disalin ke repo laporan publik ini.
