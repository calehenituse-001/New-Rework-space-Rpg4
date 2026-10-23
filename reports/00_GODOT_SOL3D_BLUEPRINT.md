# Space RPG-inspired Sol 3D — Blueprint Prototype v0.2

**Status:** rancangan disepakati sebagai *baseline untuk diuji*, belum merupakan build atau APK.

**Tujuan utama:** Membangun ulang sebuah pengalaman eksplorasi dan pertempuran ruang angkasa 3D penuh dalam Godot, dimulai dari satu sistem Sol, tetap menggunakan kontrol Android yang sederhana dan bernuansa arcade seperti Space RPG 4. Proyek terpisah dari APK dan framework smali sebelumnya.

## 1. Keputusan inti

- **3D sungguhan:** posisi `x,y,z`, rotasi tiga sumbu, planet berbentuk bola, kapal/menara/asteroid mesh 3D; tidak ada penguncian kapal ke satu bidang 2D.
- **Mode standar: Assisted Flight.** Pemain mengendalikan yaw dan pitch; thrust memberi percepatan ke arah haluan kapal; roll dibantu otomatis agar kamera tidak memusingkan.
- **Mode lanjutan (nanti): Free Flight.** Roll manual dan strafing dapat diaktifkan melalui pengaturan, tidak menjadi prasyarat memainkan game.
- **Rasa gerak:** lepas thrust = meluncur dengan inersia; brake = perlambat secara aktif; kapal tetap bisa bermanuver selama bergerak.
- **Lingkup v0.2:** bermain offline di Sol, bukan multiplayer atau sistem seluruh galaksi.

## 2. Kontrol sentuh mode Assisted

| Input layar | Perilaku |
|---|---|
| Joystick kiri horizontal | Yaw: putar haluan kiri/kanan. |
| Joystick kiri vertikal | Pitch: hidung kapal naik/turun, sehingga kapal dapat bergerak di tiga dimensi. |
| THRUST (tahan) | Percepatan ke arah haluan 3D; melepaskan tombol tidak menghentikan kapal mendadak. |
| BRAKE (tahan) | Aktifkan pengereman efektif terhadap vektor kecepatan; tahan lagi setelah hampir berhenti untuk reverse ringan (opsi konfigurasi). |
| FIRE (tahan) | Tembakkan senjata utama ke arah haluan/target terkunci. |
| TARGET (tap) | Pilih target dalam sudut pandang; tap ulang mengganti target. |
| DOCK (kontekstual) | Hanya saat dalam jarak docking dan kecepatan cukup rendah. |
| Geser area layar kosong sebelah kanan | Melihat sekitar tanpa membelokkan kapal, lalu kamera perlahan kembali mengikuti. |
| CAMERA | Berganti kamera Chase dekat / Chase jauh / Orbit view. |
| MAP | Tampilkan peta Sol interaktif; radar menampilkan informasi ketinggian relatif ke pesawat. |

**Catatan kunci:** Joystick bukan tombol bergerak instan seperti karakter darat; joystick mengatur arah hadap kapal. Ini mempertahankan thrust + inertia khas flight arcade, sambil membuka gerakan bebas 3D.

### Pengaman kontrol
- **Auto-roll/level** mengembalikan kemiringan (roll) secara perlahan, *bukan* menyetel pitch menjadi nol atau menahan gerakan pada bidang tertentu.
- **Input dead-zone** joystick dan perubahan arah yang dipercepat/diperlambat secara halus.
- **Auto-aim ringan** khusus target yang sudah dipilih dan berada dekat arah tembak; bukan aim-bot 360°.
- **Flight Assist OFF** menjadi opsi kelak: pemain dapat mengatur roll dan strafe lebih bebas.

## 3. Konsep layout HUD Android landscape

```
┌───────────────────────────────────────────────────────────────────────────┐
│ [SOL • F.E.R.]  Hull / Shield    Speed   Target: EARTH   [MAP] [PAUSE]  │
│                   [  ← arah target / jarak → ]                           │
│                                                                           │
│                      ✧                         ◉ EARTH                   │
│                           [ kapal pemain 3D ]                            │
│                                                                           │
│                                      [LOCK]   [DOCK saat dekat planet]   │
│   ( analog yaw/pitch )                     [BRAKE] [FIRE]  [THRUST]        │
│   [CAMERA]                                                               │
└───────────────────────────────────────────────────────────────────────────┘
```

- Pertahankan area tengah bersih; panel besar landing hanya muncul **setelah** docking.
- Tombol dibedakan bentuk/warna dan cukup berjauhan untuk multitouch.
- Radar atau minimap cukup kecil, dapat dibuka menjadi **peta 3D Sol** agar bisa melihat objek di atas/bawah pesawat.
- HUD mengadaptasi palet biru-gelap dan aksen hijau F.E.R.; tidak menyalin penuh interface asli.

## 4. Kamera 3D

**Awal:** Chase Camera, sekitar 15–25 satuan dunia di belakang kapal dan 4–7 satuan di atasnya, FOV sekitar 65–75 derajat (**nilai uji**, bukan data asli).

- Kamera mengikuti posisi kapal dengan smoothing dan dapat menyaksikan pitch saat menanjak/menukik.
- Kamera meredam roll kapal agar horizon UI tetap mudah dibaca.
- Saat mendekati Earth: kamera menarik keluar sedikit untuk menampilkan planet dan kapal bersamaan.
- Opsi Chase dekat / Chase jauh / Orbit; mode cockpit dapat menyusul setelah model kapal lebih detail.
- Kamera tidak boleh menembus planet saat mendekat; tambahkan collision obstruction sederhana.

## 5. Sol sebagai lingkungan 3D

**Versi awal playable:** Sun, Earth, Moon, Mars sebagai mesh sphere (material 3D), background bintang, orbit visual ringan, satu kapal F.E.R. dan penanda dock Earth.

**Ekspansi Sol:** Mercury, Venus, Jupiter, Saturn, Uranus, Neptune; detail stasiun/pos dan tata-letak berdasarkan hasil pemetaan ulang bila datanya ditemukan. Jarak dibuat terkompresi untuk gameplay dan performa; angka awal tidak boleh diklaim sebagai koordinat vanilla.

- Planet mempunyai radius mesh 3D, atmosfer/glow tipis, dan ikon jarak.
- Tidak perlu simulasi orbit astronomi realistis pada v0.2.
- Jarak sangat besar ditangani dengan skala gameplay dan/atau local origin, bukan memaksakan 1:1 kilometer.
- Docking Earth: mendekat + memperlambat, tekan DOCK -> panel planet -> UNDOCK di spawn point aman.

## 6. Strategi kapal sprite 2D menjadi 3D

Sumber `.zip` yang diberikan berisi banyak `.png` termasuk atlas kapal, efek dan planet; dalam entri yang terdaftar tidak terlihat model `.glb`/`.gltf`/`.obj` siap impor.

**Sprite asli hanya menjadi referensi bentuk dari atas**, bukan data mesh 3D lengkap. Konversi membutuhkan pemodelan bagian samping, bawah, dan ketebalan lambung.

1. **Proof-of-flight:** kapal F.E.R. geometris low-poly yang memiliki sayap/badan 3D, bukan sprite datar.
2. **Rekonstruksi siluet:** hull, engine block, armor plating, cockpit/bridge, serta tempat hardpoint mengikuti desain kapal 2D.
3. **Model final:** ekspor dari Blender ke `.glb` untuk Godot; kerjakan LOD, collision shape sederhana, engine nozzle dan damage/shield effects.
4. **Bentuk kapal besar:** modular (hull/turret/hangar/thruster); carrier dan dreadnought dapat memakai banyak mesin kecil dan efek hanggar.

Untuk prototipe pribadi gunakan aset hanya yang tersedia dengan izin; untuk distribusi publik, ganti atau lisensikan aset pihak ketiga.

## 7. Fondasi teknis Godot

- **Godot 4 + GDScript**, renderer **Mobile** untuk Android Vulkan yang cocok, dan preset **Compatibility** sebagai fallback untuk perangkat yang memerlukan GL.
- `SolWorld (Node3D)` -> `Sun`, `Planets`, `Starfield`, `PlayerShip`, `CameraRig`, `HUD`.
- `PlayerShip` menggunakan `CharacterBody3D` dengan `velocity` 3D kustom (gravity = 0), akselerasi thrust, brake dan rotasi mulus; pilihan awal agar lebih mudah diprediksi/direplikasi dibanding simulasi rigidbody rumit.
- `Planet3D` menyimpan `name, faction, position, radius, docking_radius`; `DockingSystem` dan `ShipStats` dipisahkan dari visual.
- Gameplay state tidak ditaruh di HUD. Perintah input nantinya bisa dikirim ke server, dan server memutuskan physics, collision, damage, docking dan inventory.

## 8. Angka awal untuk tuning, bukan nilai vanilla

- Kamera FOV: 70°; smoothing posisi kamera: sedang; collision kamera: aktif.
- Kecepatan putar yaw/pitch dibatasi dan bertambah halus sesuai defleksi joystick.
- Thrust pada kapal uji cukup untuk mengubah kecepatan secara jelas dalam 1–2 detik.
- Lepas thrust: 90–100% kecepatan awal bertahan setidaknya beberapa detik (tergantung drag ringan), sehingga terasa meluncur.
- Brake: lebih kuat dari drag saat thrust dilepas.
- Target kinerja: 60 FPS di perangkat Android yang memadai; mode grafis ringan dan opsi resolusi tersedia. **Target, bukan hasil benchmark.**

## 9. Kriteria lulus v0.2

- [ ] APK/proyek Godot berhasil dibuka di Android landscape.
- [ ] Kapal dan Earth keduanya benar-benar 3D, dapat dilihat dari berbagai sisi.
- [ ] Kapal bisa meninggalkan bidang datar: pitch naik/turun menyebabkan perubahan koordinat Y.
- [ ] Analog kiri dan THRUST bisa dipakai bersamaan tanpa input hilang.
- [ ] Saat thrust dilepas, kapal meluncur; ketika BRAKE ditekan, kapal melambat.
- [ ] Kamera mengejar kapal tanpa putaran mendadak.
- [ ] Earth docking hanya terbuka pada jarak dan kecepatan yang memenuhi syarat.
- [ ] UNDOCK mengembalikan ke dunia 3D tanpa posisi spawn tertanam di planet.
- [ ] Starfield dan planet tidak menyebabkan penurunan frame rate besar pada perangkat tes.

## 10. Setelah v0.2 stabil

**v0.3:** laser dan lock target, damage, shield, satu NPC dummy.  
**v0.4:** komponen Sol lainnya, peta, stasiun dan interface docking.  
**v0.5:** dua pemain di Sol, identitas objek independen, pergerakan server-authoritative, sinkronisasi orientasi 3D.

---

**Keputusan batas cakupan:** desain ini belum mengklaim source Godot 3D sudah dibuat atau dites; v0.2 ini khusus blueprint agar implementasi berikutnya tidak melenceng ke pseudo-3D.