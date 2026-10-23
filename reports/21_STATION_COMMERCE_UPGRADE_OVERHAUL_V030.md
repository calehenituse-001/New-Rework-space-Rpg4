# v0.3.0 — Commerce, Upgrade & Overhaul untuk F.E.R. Orbital Station

**24 Oktober 2026.** Kelanjutan Godot v0.2.9, setelah user menyatakan kamera v0.2.7/v0.2.8 aman. **Status: implementasi kode + ZIP dibangun; BELUM dites di Godot Android (runtime)**. Kamera dan navigasi AUTO ENGAGE diverifikasi tidak berubah dari source v0.2.9 melalui perbandingan fungsi statis.

## Artefak
- **Project ZIP:** `SRPG4_Sol3D_v0_3_0_Commerce_Upgrade_Overhaul.zip` (lampiran percakapan, **bukan** di-upload ke repository yang khusus untuk laporan).
- SHA-256 ZIP: `b6e4724d7beb26f53477bce3d2bfa70b4e99be04c51114c26f904b9653fdfaaa`
- 53 file, ZIP CRC valid. Semua sumber `preload(res://...)` yang literal ada. Kelima aset model kapal tetap ada. **Tidak ada eksekusi Godot/GDScript parser** dalam lingkungan pembuatan.

## Fitur yang ditambahkan

| Fasilitas | Fungsi yang dikodekan |
|---|---|
| OUTFITTING | Tiga hardpoint TEST Port/Stbd/Dorsal; memasang hanya dari inventory; melepas senjata kembali ke inventory; konfigurasi per-ship tetap user://fer_outfitting_v029.json |
| COMMERCE | Beli Cannon / Beam memakai credits; stok terbatas dan inventory global; jual hanya item yang sedang tidak dipasang; 60% nilai beli |
| UPGRADE | Targeting AI improve projectile lead (tanpa auto-hit), Adaptive Shielding mengalikan kanal shield 0.85, Range Extension +30% hanya Cannon prototype |
| OVERHAUL | Shield maximum 460→575 dan Hull maximum 900→1080, masing-masing satu kali untuk kapal terkait |
| REPAIR | Berbayar berdasarkan damage shield/hull yang hilang; mengisi sesuai maksimum aktif |
| SAVE | Saldo/inventory/stok/upgrades/overhauls di user://fer_station_services_v030.json; slot outfitting memakai save v0.2.9 yang sama |

### Angka fixture (BUKAN harga/stat ekonomi original Space RPG 4)
- Starter 18.000 Cr. Cannon 850 Cr (stock 12), Beam 1.350 Cr (stock 8).
- Targeting AI 2.400 Cr, Adaptive Shielding 3.200 Cr, Range Extension 2.700 Cr.
- Shield Capacity overhaul 2.400 Cr, Hull Reinforce overhaul 2.600 Cr.
- REPAIR = ceil(shield damage × 0.4 + hull damage × 0.8) Cr.
- Ini **gameplay test** untuk membuktikan alur transaksi dan equipment. Belum ada integrasi 79 senjata/92 kapal original, shop faksi, cargo, restock, atau multiplayer authoritative.

## Perlindungan terhadap versi sebelumnya
- Membuat ZIP baru, tidak menimpa ZIP v0.2.9.
- `config/name` Godot disamakan dengan v0.2.9 sehingga path `user://` kompatibel saat dijalankan di lingkungan yang sama.
- Fungsi `_update_camera`, `_change_camera`, `_start_target_camera`, `_release_npc_track_after_target_lost`, serta joystick orbit/aim tidak berubah secara tekstual; `CombatManager.apply_autonav` juga tidak berubah.
- Perubahan combat terlokalisasi pada penerapan upgrade (Targeting AI, cannon range, shield damage modifier, maksimum pertahanan).

## Perubahan source
- **Baru:** `scripts/station_economy.gd` — local state + transaksi, per-ship upgrades/overhaul, price/stock, repair.
- `scripts/flight_hud.gd` — tab 4 halaman dan tombol shop/upgrades; UI status dan credits.
- `scripts/sol_3d.gd` — integrasi transaksi saat docking dan hardpoint inventory.
- `scripts/combat_manager.gd` — konfigurasi projectile range, Targeting AI dan apply equipment/overhauls.
- `scripts/flight_ship.gd` — max shield/hull dan modifier Adaptive Shielding.
- `README_TEST_COMMERCE_UPGRADE_v030_ID.md`, `CHANGELOG_v0_3_0.txt`, `STATIC_VALIDATION_v030.json`.

## Alur tes Android (SEMUA BELUM DILAKUKAN)
1. Jalankan project via Godot 4.x Android, masuk Earth Orbital Dock v0.2.9.
2. COMMERCE beli 1 Cannon dan 1 Beam; credits/inventory/stock harus berubah.
3. OUTFITTING tukar BEAM ke mount Port, lalu kosongkan dan isi ulang; item harus masuk/keluar inventory dengan benar.
4. Coba equip saat stock inventory kosong → perubahan ditolak dan selection kembali ke awal.
5. UPGRADE beli Targeting AI/Adaptive Shield/Range Ext; uji perbedaan lead/range/kanal shield.
6. OVERHAUL beli Shield Cap/Hull; status per ship, satu pembelian setiap jenis.
7. REPAIR setelah menerima damage; harus ditolak bila saldo kurang dan charge sesuai damage.
8. Undock, AUTO FIRE/AUTO ENGAGE/TARGET NPC/CAMERA tetap bekerja.
9. Tutup/buka Godot; cek persistence. Ganti ship: upgrades/overhaul per-ship; credits dan inventory bersama.
10. Kirim screenshot Output dan nomor baris jika parser error atau crash.

## Batasan
- Tidak ada uji runtime di Android: **jangan sebut fitur sudah lulus pengujian sebelum user memverifikasi**.
- Economy/outfitting sekarang masih tersimpan pada dua file JSON; jika proses mati persis saat perpindahan slot, operasi antarfile belum transaksional.
- GUI dibangun dinamis dan belum diuji tampilan Android untuk semua resolusi.
- Multiplayer, animasi mechanic docking/repair, per-hull armor original, pilihan stok planet original, misison dan save game dunia masih tahap selanjutnya.

**Selanjutnya sesudah tes:** perbaiki runtime jika diperlukan, baru mapping katalog Commerce/Upgrade/Overhaul original untuk mengubah test fixtures menjadi item resmi dan menambah stock/reputation/merchant.
