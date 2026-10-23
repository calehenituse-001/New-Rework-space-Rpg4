# SOL 3D — Flight Prototype v0.2 (Godot 4)

**Prototipe kode sumber Godot, bukan APK.** Ini penerusan konsep v0.1 ke **dunia 3D penuh** dan menggunakan empat model kapal `.glb` yang kamu kirim dari Astra. Tidak ada mode multiplayer pada v0.2. Source proyek dibuat di sini dan telah diperiksa secara statis; **belum dijalankan dalam Godot editor atau diuji pada Android** di lingkungan penyusunan ini. Laporkan error merah dari tab Debugger jika ada.

## Yang sudah disiapkan

- Empat kapal 3D asli dari `4_Kapal_Hijau_3D_Godot_v01.zip`, dapat diganti saat bermain lewat **SHIP ▶** / angka **1–4**. Bentuk dan tekstur bawaan Astra tertanam di dalam `.glb`.
- Matahari, Mercury, Venus, **Earth**, Moon, Mars, Jupiter, Saturn, Uranus, Neptune sebagai **objek mesh 3D** dengan tekstur procedural khusus prototipe, collision sederhana, label, atmosfer tipis Earth, cincin Saturnus, bintang latar, serta panduan lintasan hias.
- Gerak kapal 3D sesungguhnya pada **sumbu X/Y/Z**. Joystick analog untuk **yaw/pitch**, THRUST mempercepat searah haluan, lepas THRUST mempertahankan inersia, BRAKE memperlambat. Bank kecil visual saat berbelok (tidak mengacaukan orientasi fisika).
- Kamera yang mengikuti kapal (tiga jarak/sudut), target planet bergantian, panel peta jarak, HUD speed/altitude, tombol docking Earth serta **UNDOCK**, dan laser visual demonstrasi.
- Masukan multitouch Android: analog + THRUST/FIRE/BRAKE dapat dipegang bersamaan, menggunakan event sentuh terpisah. Keyboard desktop juga tersedia.

## Cara membuka (Android / PC)

1. Ekstrak **seluruh** ZIP ini menjadi sebuah folder; **jangan** jalankan ZIP tanpa ekstraksi.
2. Jalankan **Godot Engine 4.3 atau lebih baru** (direkomendasikan 4.5.x). Gunakan Godot **Editor untuk Android** jika tidak ada komputer; editor tersebut terpisah dari file game.
3. Di Project Manager Godot, pilih **Import** lalu arahkan ke file `project.godot` dalam folder tersebut.
4. Tunggu sampai keempat model GLB selesai diimpor. Tekan ▶ **Run Project / Play**.
5. Pada Android, sebaiknya gunakan orientasi landscape dan izinkan akses folder proyek.

Jika Godot Android Editor tidak mengizinkan akses folder tempat ZIP diekstrak, pindahkan folder hasil ekstrak ke lokasi yang bisa dipilih melalui file manager/editor, lalu import `project.godot`.

## Kontrol Android

| Kontrol | Aksi |
|---|---|
| Analog kiri ↔ | Belok kiri/kanan (yaw) |
| Analog kiri ↑ ↓ | Hidung kapal naik/turun (pitch); koordinat Y berubah saat thrust |
| THRUST (tahan) | Mempercepat ke depan sesuai orientasi kapal |
| BRAKE (tahan) | Mengurangi momentum (tidak otomatis mundur pada versi ini) |
| FIRE (tahan) | Laser visual latihan, belum damage / AI |
| TARGET | Mengganti planet target pada HUD |
| CAMERA | Kamera dekat / jauh / tinggi |
| SHIP ▶ | Ganti salah satu empat model kapal 3D |
| MAP | Daftar jarak planet dari pemain |
| DOCK EARTH | Muncul saat dekat Earth dan kecepatan kurang dari 52 |
| UNDOCK | Keluar dari tampilan Earth dan respawn aman di dekat Earth |

**Keyboard desktop:** W/S=pitch, A/D=yaw, SPACE=thrust, SHIFT=brake, F=laser, E=dock/undock, C=kamera, TAB=target, 1–4=ganti kapal. Mouse dapat digunakan untuk menekan tombol HUD atau menggerakkan analog.

## Uji cepat disarankan

1. Setelah scene tampil, kamu berada sekitar Earth, kapal menghadap ke arah planet.
2. Tahan **THRUST** dan putar analog. Cek kecepatan naik lalu kapal tetap meluncur setelah THRUST dilepas.
3. Dorong analog atas/bawah sambil thrust. Cek nilai **Y** di HUD berubah, membuktikan perpindahan benar-benar 3D.
4. Tekan **SHIP ▶** untuk melihat 4 model berbeda, kamera dekat/jauh lewat **CAMERA**.
5. Mendekat ke Earth perlahan, tahan **BRAKE**. Saat cukup dekat dengan speed <52, tombol **DOCK EARTH** muncul. Tekan, lalu **UNDOCK**.
6. Tahan **FIRE** untuk memeriksa laser visual, serta buka **MAP** untuk daftar jarak.

## Batasan saat ini

- Planet **bukan** koordinat/ukuran asli Space RPG 4. Ini **layout demonstrasi yang dikompresi**; jangan dianggap hasil mapping akurat dari APK.
- Konten game original seperti commerce, shipyard, story, quest, NPC, damage dan save belum dibuat. Dock Earth saat ini cuma UI mockup.
- Laser belum memakai sistem tumbukan, damage atau server. Tidak ada jaringan / multiplayer.
- Kapal memakai *box collision* kasar, bukan collision mesh detail. Belum ada AI, navigasi warp, atau optimasi LOD.
- Efek glow thruster memakai node `Thruster_*` yang disediakan model Astra. Jika ada import transform berbeda, efeknya mungkin perlu disesuaikan.
- Target performa Android perlu diuji di perangkat nyata. Pilihan renderer **GL Compatibility** mengikuti proyek penampil Astra untuk kompatibilitas ponsel.

## Struktur file

```
project.godot                 ← Import ini!
scenes/main.tscn             ← Scene utama
scripts/sol_3d.gd            ← Tata Sol, input, kamera, docking
scripts/flight_ship.gd       ← Controller kapal 3D dengan inertia
scripts/virtual_joystick.gd  ← Kontrol sentuh/multitouch
scripts/flight_hud.gd        ← HUD Android dan docking mockup
scripts/laser_bolt.gd        ← Laser visual
models/*.glb                 ← Empat kapal 3D dari Astra
assets/planets/*.png         ← Peta warna planet procedural baru
```

## Asal berkas dan penggunaan

Model-model kapal berasal dari ZIP Astra yang **kamu unggah**; sudah disalin tanpa modifikasi data GLB. Tekstur planet di proyek ini dibuat secara prosedural khusus prototipe (tidak diekstrak dari aset game orisinal). Proyek ini adalah **proyek terpisah** yang terinspirasi Space RPG 4, bukan port engine asli. Jangan distribusikan aset yang tidak kamu miliki haknya tanpa izin.

Dibangun agar nantinya mudah menambahkan **multiplayer server-authoritative**, tetapi belum mengimplementasikannya.