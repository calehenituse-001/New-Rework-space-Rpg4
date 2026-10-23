# Mapping Tahap 6 — Sol, Earth, Luna, Agrodome & Docking 3D
**Audit statis: 23 Oktober 2026 | Space RPG 4 v0.9995(44)**. Belum diuji runtime. Kode dan prototipe Godot tidak diubah.

## Penemuan utama
Source `h1.smali`, method `b([Lk2;)V`: Sol merupakan sistem **ID 0**, pemilik **faksi ID 2 (F.E.R.)**, posisi peta galaxy **(0,0)**. Array bobot spawn mentah (sebelum transformasi `h1.h`) adalah Independent 45, F.E.R. 45, T.A. 5 — jangan tafsirkan langsung sebagai peluang akhir runtime.

Source `z6.smali`, method `a()V`: objek bernama dengan `Ll6.K4 = 0` yang teridentifikasi pada generator adalah empat, bukan delapan planet tata surya modern.

| Objek asli | ID `J4` | Sistem `K4` | Posisi `i5,j5` | Ukuran render `k5` | Shipyard `t5` | Commerce `u5` | Trade Centre `v5` |
|---|---:|---:|---:|---:|---|---|---|
| Earth | 0 | 0 | (0,0) | 256 | Ya | Ya | Ya |
| Luna | 1 | 0 | (350,-150) | 64 | Tidak | Tidak | Tidak |
| Agrodome 1 | 87 | 0 | (-500,-300) | 128 | Tidak | Tidak | Tidak |
| Agrodome 2 | 88 | 0 | (-380,-410) | 128 | Tidak | Tidak | Tidak |

Nilai merupakan **koordinat/parameter gambar 2D internal**, bukan meter atau jari-jari planet fisik. Source lain/runtime/story/save lama mungkin menambah/mengubah objek. Keempat objek ini adalah baseline yang terkonfirmasi dari generator; bukan klaim daftar semua objek di semua kondisi gameplay. Prototype Godot dengan Mercury/Venus/Mars/Jupiter/Saturn/Uranus/Neptune adalah perluasan demo, **belum terbukti identik dengan Sol original**.

## Crosswalk field
- `Lk8.f/b/c/h/i/n` = nama sistem, ID, pemilik, koordinat galaxy dan bobot spawn (source `h1`).
- `Ll6.N4/J4/K4/i5/j5` = nama, ID, ID sistem, posisi internal 2D (source `z6`).
- `Ll6.k5` = parameter ukuran; `o5.smali` melewatkan nilainya ke constructor render `w6` sebagai parameter dimensi X/Y.
- `Ll6.U4` = indeks style/atlas; `o5` mengindeks hasil `m3.p`. Earth 4, Luna 7, kedua Agrodome 11.
- `Ll6.W4/X4/V4` disalurkan via `sj.g` sebagai parameter UV/material; **jangan** dianggap ukuran planet fisik.
- `Ll6.t5/u5/v5` mengisi slot UI Shipyard/Commerce/Trade Centre di `dj.smali` (indeks 3/4/5). Label diperiksa melalui `public.xml` + `strings.xml`.
- `Ll6.O4/Q4/l5` masih belum dipastikan maknanya. Nilai mentah dicatat di dataset.

## Kaitan docking original dan desain remake
- Original memakai `z6` untuk generator dunia/layanan, `k1` untuk save/load state SQLite, `j2` untuk planet/landing, `dj` untuk tab menu, `o5` untuk instance grafis 2D. Permission dan syarat cerita dapat menolak landing; jangan hanya memeriksa jarak.
- Earth punya Shipyard, Commerce, Trade Centre dalam flag statis. Luna dan Agrodome tidak menyalakan tiga flag tersebut; ini tidak berarti tidak punya fungsi cerita/layanan lain.
- **Tambahan remake yang direncanakan:** orbital station F.E.R. dekat Earth, auto-approach → align → hangar 3D → menu fleet/commerce/shipyard → undock. Jangan mengklaim sistem visual docking ini sudah ada di original.
- Titan dan capital ships dapat memakai mega-dock khusus. Saat docking, navigasi AUTO ENGAGE dimatikan; status ditetapkan server authoritative nantinya.

## Aset dan kebutuhan 3D
- Source ZIP memiliki **18 atlas `planetmap_*.png`**: habitable 1–4, uninhabitable 1–4, large 1–3, supergiant, blackhole/core, stations, playerstation, gressian orbital. Mayoritas 1024×1024; beberapa 512×512.
- Ada `planet_outpost.png` dan `planetary_shield.png`. Atlas gambar 2D **tidak otomatis menjadi texture UV sphere 3D**. Model, material dan UV harus disiapkan terpisah.
- **Skala usulan, bukan data asli:** Titan panjang 95 unit Godot; Moon diameter 80; Earth diameter 1600; jarak/orbit harus diatur ulang. Ini mewujudkan permintaan Titan > Moon tetapi Earth jauh lebih besar dari Titan, dan berbeda dari rasio original `k5` Earth:Luna = 256:64.

## Mapping belum selesai
1. Misi dan syarat muncul/interaksi kedua Agrodome.
2. Stok aktual Earth (shop, overhaul, repair, shipyard) dan gate progress.
3. Radius landing asli, syarat permission/story, serta save/restore saat dock.
4. Korespondensi UV `U4/W4/X4/V4` ke subregion atlas yang benar.
5. Perancangan prefab Sol 3D (source_2d dan world_3d dipisah), stasiun/hangar, performa Android, lalu tes.

**Artefak lokal lebih lengkap:** `SRPG4_Mapping_Sol_Tahap6.zip` memuat laporan versi panjang, JSON empat objek Sol, inventaris aset, dan ekstraktor. Tidak ada APK, Smali lengkap, atau aset gambar game asli di repositori laporan publik.