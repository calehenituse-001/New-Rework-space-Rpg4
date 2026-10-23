# Audit Weapon Models GLB Alignment v0.1.1 — 23 Oct 2026

## Konteks
Pengguna mengunggah `SRPG4_Weapons3D_Detail_v01.zip` (6 rekonstruksi model senjata: Blaster, CIWS, Medium Cannon, Irochaser, Heavy Cannon, Ion Cannon) karena desain disebut tampak **miring**. Paket Work semula berisi `.blend`, Blender generator, GLB, previews, dan Godot Workshop. Kuota Work habis sebelum dilakukan pemeriksaan lanjutan.

## Hasil pemeriksaan lokal (bukan klaim visual sudah identik sprite)
- Keenam file GLB valid dibaca lewat Trimesh.
- Pada keenam GLB, transform **YawPivot/PitchPivot netral** (matriks rotasi identitas).
- Senjata tunggal: semua `Muzzle_01` berada tepat di sumbu **X=0**.
- Senjata ganda: `Muzzle_01/02` tercatat simetris **X=±nilai sama**, Y/Z serupa.
- Generator Blender `barrel()` memodelkan laras lurus di sepanjang sumbu +Y Blender → -Z Godot.
- *Kesimpulan terbatas:* tidak terdeteksi miring **secara geometris pada pusat laras** dalam pose netral; tampak serong pada gambar perspektif/top mungkin berasal dari rotasi tampilan, NLA preview atau kamera. Ini **tidak membuktikan setiap komponen mesh sempurna simetris**. Perlu orthographic front/top untuk mengetahui bagian mana yang perlu dibetulkan.
- Informasi ukuran/tris: Blaster ~18.5k, CIWS ~21.1k, Medium Cannon ~19.7k, Heavy Cannon ~27.2k, Irochaser ~22.0k, Ion Cannon ~20.8k triangles masing-masing.

## Patch non-destruktif dibuat
`SRPG4_Weapons3D_Alignment_Check_v0_1_1.zip` tersedia sebagai attachment percakapan (tidak diunggah ke GitHub report-only). Tidak mengubah geometri GLB atau Blender master; hanya memperbarui `godot/scripts/workshop.gd` dengan **3D Bebas, Depan 0°, Atas 0°, Samping 0°** dan view neutral yaw/pitch, plus laporan `README_ALIGNMENT_v0_1_1_ID.md` dan `validation/ALIGNMENT_CHECK_v01_1.json`.

**Penting:** paket ZIP sudah diuji integritas arsip dan scene graph; **belum runtime-tested di Godot Android** sebab executable Godot/Blender tidak tersedia di lingkungan inspeksi.

## Langkah selanjutnya
1. Pengguna membuka workshop di Godot Android dan menguji orthographic front/top.
2. Jika nampak offset satu bagian dalam view 0°, identifikasi senjata dan bagian yang melenceng, lalu patch `source/build_weapons.py`/mesh dan ekspor ulang GLB tanpa mengubah silhouette original yang diinginkan.
3. Setelah model dinyatakan rapi, mapping muntah ke `WeaponMount3D` dan slot/hardpoint kapal dari source original, serta optimisasi mesh LOD untuk Android.

**Catatan:** semua aset asli model dan sprite tidak disalin ke repo dokumentasi publik.
