# Godot Sol3D Combat Harness v0.2.4 — Implementasi test, 23 Oktober 2026

**Status: Proyek ZIP Godot dibuat, sudah diperiksa struktur statis, tetapi BELUM diuji jalan di Godot Android.** Ini bukan APK dan bukan build final. Berangkat dari file *yang telah diuji pengguna* `SRPG4_Sol3D_Flight_v0_2_3.zip`. ZIP v0.2.3 tidak ditimpa.

## Hasil implementasi kode saat ini
- Menambahkan node `CombatSystem` terpisah dari kamera dan fly controls.
- Ship player memakai **Astra Carrier Garpu sebagai _kapal berat proxy_**, karena belum tersedia model battleship F.E.R. yang cocok untuk integrasi. Ship button tetap memilih model keempat kelas lain; di setiap perubahan ship, mount dipasang ulang sesuai collision box.
- **2 cannon projectile 3D** (PORT dan STBD), masing-masing memiliki sektor horizontal 180° relatif sisi, batas elevasi, tracking speed, cooldown, range mandiri, damage, dan origin tepat dari muzzle. Ada **1 dorsal beam hitscan** independen.
- Firing gate: target valid dan terkunci, range tiap senjata, yaw dan pitch dalam batas, turret sudah melacak target, perkiraan halangan lambung sendiri via AABB, line of sight ke target, lalu fire. Projectile bergerak fisik dengan raycast per tick; lawan yang tiba-tiba mengubah arah bisa lolos dari titik lead.
- **Pirate Interceptor 3D** menggunakan model Berlian, manuver/zigzag di depan player, health + shield, mempunyai satu cannon 3D sendiri, respawn setelah 4 detik. Ini **AI test stub, bukan port AI SRPG4 original**.
- HUD: baris kedua tombol `LOCK NPC`, `AUTO FIRE`, `AUTO`/`CANCEL`, counter shots/hits dan shield/hull. Tombol Camera LOCK/Target Planet semula tetap terpisah dari lock NPC.
- AUTO Engage mengarahkan kapal untuk mendekati lawan (~79 game units), menggunakan fly control asli yang menjaga momentum, dan menembak lewat hardpoint. Joystick/BRAKE atau tombol CANCEL membatalkan AUTO Engage, tidak menghentikan inertia. AUTO FIRE merupakan saklar mandiri: bila ON manual, cancel nav tidak mematikannya; jika AUTO Engage saja tanpa AUTO FIRE mandiri, cancel juga menghentikan penembakan otomatis.
- EARTH DOCK masih menggunakan popup lama, tidak ada hangar/outfitting persisten. Outfitting menjadi fitur berikutnya setelah combat test.

## Statistik sementara, BUKAN angka resmi Space RPG 4

| Mount | Range (unit Godot) | Speed | Damage | Cooldown | Yaw arc |
|---|---:|---:|---:|---:|---:|
| Player Port Cannon | 132 | 212 | 12 | 0.40 s | 180° |
| Player Starboard Cannon | 132 | 212 | 12 | 0.40 s | 180° |
| Player Dorsal Beam | 98 | hitscan | 4 | 0.70 s | 270° |
| NPC Dorsal Cannon | 126 | 172 | 7 | 0.78 s | 210° |

Ketiga turret player memiliki elevasi -22°..75°; NPC -25°..80°. Ini pilihan *test fixture* untuk konfirmasi user; visual mount berupa mesh sederhana, belum custom model turret resmi, belum menerapkan database 79 senjata/92 ships secara penuh.

## Struktur file baru/berubah
- `scripts/combat_manager.gd` — lock, auto fire, nav, test loadout 3 mount, counters.
- `scripts/combat_mount.gd` — hardpoint gimbal + arc/elevation + own-hull AABB occlusion + projectile/hitscan.
- `scripts/combat_projectile.gd` — collision via PhysicsRayQueryParameters3D tiap physics tick.
- `scripts/combat_npc.gd` — interceptor NPC dummy bergerak, shield/hull, respawn.
- `scripts/sol_3d.gd` — hubungkan HUD dengan CombatSystem dan input autopilot.
- `scripts/flight_hud.gd` — tombol lock/auto/action.
- `scripts/flight_ship.gd` — shield/hull test.
- `README_TEST_COMBAT_v0_2_4_ID.md` — cara tes.
- `CHANGELOG_v0_2_4.txt`.

## Langkah tes Android untuk pengguna
1. Extract ZIP project, buka `project.godot` dengan **Godot 4.x Editor Android** lalu jalankan `scenes/main.tscn`.
2. Temukan Interceptor musuh di depan kapal Carrier; tekan `LOCK NPC`.
3. Coba `FIRE` manual, `AUTO FIRE`, lalu `AUTO` dan `CANCEL`.
4. Periksa senjata side tidak menembak ke belakang/sisi hull, dan dorsal beam tidak memiliki travel time seperti cannon.
5. Periksa projectile bisa meleset terhadap Interceptor yang berzigzag; shield/hull UI berkurang jika terkena.
6. Gerakkan joystick atau tekan BRAKE ketika AUTO aktif; kapal kembali manual tanpa kehilangan inertia.
7. Ganti SHIP lalu cek apakah mount dipasang ulang; coba DOCK EARTH agar auto engagement dihentikan.
8. Laporkan **screenshot error Godot berikut nomor baris** bila terjadi parser error, karena container tidak memiliki executable Godot untuk pengujian runtime.

## Hal yang belum diimplementasikan
- Tidak ada instalasi senjata melalui DOCK → Outfitting, inventory, save loadout, Garage 3D atau pembayaran.
- Tidak ada missile/ECM, semua ID senjata asli, sistem ammo/energy/overload lengkap, orbit mode, 3D station, multiplayer.
- **Own-hull occlusion memakai AABB collision proxy**, belum per-triangle mesh atau collider multiparts. Kualitasnya harus dites.
- Tidak ada tes compile/scene/render di Android; *jangan laporkan fitur sudah 100% bekerja sebelum diuji oleh pengguna.*

## Rencana setelah tes lolos
1. Verifikasi locking, tracking, projectile dan turret occlusion di Godot Android.
2. Tambahkan mount geometry sesuai detail model kapal (termasuk custom F.E.R. Battleship).
3. Hubungkan data weapon/hardpoint dari mapping stage 2–11 sebagai loadout terstruktur.
4. Implementasi dock station 3D → Ship Outfitting → pemasangan weapon fisik, persistensi inventory/loadout.
5. Siapkan server-authoritative multiplayer setelah sistem lokal stabil.

Arsip ZIP yang diserahkan ke pengguna: `SRPG4_Sol3D_Combat_v0_2_4.zip` (tetap sebagai attachment percakapan, **tidak** diunggah ke repo laporan yang diminta hanya memuat dokumentasi).
