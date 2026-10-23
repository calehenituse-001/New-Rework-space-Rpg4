# Godot Sol3D v0.3.0 — Earth Station Commerce, Upgrade & Owned Loadout Test

**24 Oktober 2026.** Lanjutan dari [v0.2.9 Orbital Dock](20_EARTH_DOCK_OUTFITTING_V029.md). **ZIP proyek Godot benar-benar dibuat** dalam percakapan: `SRPG4_Sol3D_v0_3_0_Commerce_Upgrade_Test.zip` (±6,49 MB, 52 berkas; 13 script GDScript). **Belum diuji runtime di Godot Android**. Yang sudah diperiksa ialah integritas ZIP, referensi `res://` preload, file model/scene, indentation, sambungan fungsi dan sinyal.

## Scope pekerjaan
Pengguna telah menyatakan kamera dua mode aman. Tidak ada perubahan model kapal, planet, fisika navigasi, targeting/camera lock, AUTO FIRE maupun AUTO ENGAGE. Fokus berikutnya membangun ekonomi dan upgrade **minimal** pada fasilitas orbital station Earth, supaya setup senjata tidak bebas tanpa inventory.

### New: Ship Outfitting menggunakan persediaan senjata asli dalam prototipe
- Tab `OUTFITTING` tiga slot test tetap ada: PORT / STARBOARD / DORSAL.
- Memasang `BEAM` atau `CANNON` membutuhkan item di **gudang**. Mengganti/mengosongkan slot mengembalikan senjata lama ke gudang, dan menarik senjata baru dari gudang.
- Kapal Astra/FER dalam game tester masih masing-masing memperoleh default `CANNON,CANNON,BEAM`, bukan loadout lengkap Space RPG 4 original. Sistem slot dari mapping belum diaplikasikan sepenuhnya.
- Setelah undock, jenis `BEAM`, `PROJECTILE`, atau `EMPTY` mengubah sistem mount/tembakan sungguhan, bukan sekadar nama di UI.

### New: Commerce
- Tab `COMMERCE` menyediakan `CANNON` (320 credits) dan `BEAM` (580 credits).
- Saldo awal **5000 credits**; stok gudang senjata mulai nol (senjata yang sudah terpasang pada tiap kapal bukan stok terpisah).
- Pembelian mengurangi saldo, mengisi gudang dan menyimpan semuanya. Tanpa saldo, pembelian ditolak.
- Harga, saldo dan jenis dua barang **hanya TEST FIXTURE**, **bukan** klaim angka/stat original.

### New: Upgrades per ship
- Tab `UPGRADES`: Targeting AI (harga demo 1250 credits, inspirasi upgrade original ID 208) dan Adaptive Shielding (1800 credits, ID original 213).
- Targeting AI meningkatkan leading proyektil `prediction_factor` dari **0.72 menjadi 0.92** dalam script 3D. **Tidak menambah range dan tidak memberi hit 100%.** Keputusan belok lawan tak bisa diprediksi sempurna.
- Adaptive Shielding mengurangi damage kanal shield selama shield masih aktif dengan faktor **0.85**, sebagai **aproksimasi** jalur original `e8.m`, bukan formulasi lengkap HP/armor/penetration.
- Upgrade tersimpan **per indeks kapal**, langsung dipasang saat dibeli. Tidak ada addon upgrade yang dapat dipindah antarship dalam test ini.

### Local save
- New `scripts/station_services.gd` menyimpan `credits`, `inventory`, `loadouts`, `upgrades` dalam `user://fer_station_services_v030.json`.
- Jika file v0.3.0 belum ada, data loadout v0.2.9 `user://fer_outfitting_v029.json` dibaca untuk kompatibilitas ringan; tidak mengubah save lama.
- Belum menggunakan server authoritative, save anti-cheat, pembelian item dari database original, disk reconciliation/jurnal transaksi, atau inventory barang lainnya.

## File utama
- `scripts/station_services.gd` — status credits/gudang, buy/equip/upgrade, save/load.
- `scripts/combat_manager.gd` — pembelian hanya saat docked, reconfigure mounted cannon/beam, apply ship upgrades.
- `scripts/combat_mount.gd` — baseline leading 0.72 dan AI 0.92.
- `scripts/flight_ship.gd` — flag adaptive shield untuk damage channel sementara.
- `scripts/flight_hud.gd` — 3 tab station UI, credits, stock, tombol beli.
- `scripts/sol_3d.gd` — station callbacks dan sinkronisasi HUD.
- `README_TEST_STATION_v030_ID.md`, `CHANGELOG_v0_3_0.txt`.

## Tes untuk pengguna Godot Android — **BELUM DIJALANKAN**
| ID | Langkah | Kriteria |
|---|---|---|
| V030-A | Masuk dari spawn, DOCK STATION | Hangar tampil, tiga tab UI |
| V030-B | Equip BEAM ke PORT dengan stok 0 | Ditolak, pemilihan UI kembali |
| V030-C | COMMERCE beli BEAM 580 Cr | Saldo 5000→4420, stok BEAM 0→1 |
| V030-D | Equip BEAM ke PORT | BEAM −1 stok, CANNON +1 stok |
| V030-E | Ubah slot ke EMPTY | Senjata dilepas dan masuk gudang |
| V030-F | Beli Targeting AI | Saldo turun; flag ON; prediksi lebih kuat tetapi masih bisa miss |
| V030-G | Beli Adaptive Shield | Flag ON; shield-channel damage berkurang saat shield aktif |
| V030-H | UNDOCK, lock NPC, AUTO FIRE/AUTO ENGAGE | Camera/physics/combat lama tidak regress |
| V030-I | Tutup/buka Godot | Saldo, stok, loadout dan upgrades tersimpan |
| V030-J | Pindah ship dan dock lagi | Upgrade dan loadout sesuai ship indeks; tidak disatukan |

## Batasan dan kejujuran validasi
Godot executable tidak tersedia di lingkungan perakitan. Build ZIP **tidak sama dengan** scene berhasil dijalankan. Sebelum lanjut Overhaul/shipyard/full Commerce, perlu pengujian di Godot Android termasuk parsing GDScript, tampilan tab pada ukuran layar ponsel, dan pembelian yang bertahan saat restart. Godot 4.7.2 Android adalah lingkungan user.

**Repo hanya untuk laporan**; proyek ZIP dan aset custom tidak diunggah ke repo.

## Next setelah tes
1. Perbaiki error Godot bila ada.
2. Pisahkan inventory store dan equipped state saat pindah kapal melalui Garage resmi (sekarang switch SHIP di luar dock hanya fixture tester).
3. Mapping kategori upgrade dan Overhaul yang belum diterapkan.
4. Baru tambahkan stok/restock/shipyard berdasarkan source original dan persetujuan skala gameplay.
