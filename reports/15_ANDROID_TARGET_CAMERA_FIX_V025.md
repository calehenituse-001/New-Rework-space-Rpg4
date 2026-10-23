# Feedback Android v0.2.4 → Target NPC / Camera Track Fix v0.2.5
**23 Oktober 2026 | Status:** pengguna telah menguji v0.2.4. Patch v0.2.5 dibuat dari arsip ZIP proyek, diperiksa file dan ZIP, **belum diuji runtime di Godot Android**.

## Hasil tes yang pengguna laporkan
- AUTO ENGAGE sudah berhasil membuat kapal mengejar musuh.
- AUTO FIRE dan AUTO ENGAGE dinilai berjalan baik.
- Tombol **TARGET** di atas malah mengganti navigasi target antara planet (Jupiter/Earth/dll.), bukan mengunci kapal musuh.
- Camera LOCK dirasakan mengarah ke planet. Pemeriksaan sumber v0.2.4: `_change_target()` memang mengiterasi `TARGET_SEQUENCE` berisi planet, sedangkan kamera `LOCK` mengunci **arah pandang world-space**, bukan track target. Keduanya membuat UX membingungkan, meski implementasi kamera LOCK sendiri tidak memakai koordinat planet langsung.

## Penyebab dari source v0.2.4
`sol_3d.gd:_change_target()` memakai `TARGET_SEQUENCE = [Earth,Moon,Mars,...]`; tombol `TARGET` memanggilnya. Combat sebenarnya mempunyai tombol terpisah `LOCK NPC`, lalu `CombatSystem.enemy`. Jadi dua jenis "target" berbagi label yang ambigu.

## Perbaikan v0.2.5 (Godot project ZIP, tidak di-upload ke repo laporan)
- Tombol **TARGET NPC** sekarang mengunci Pirate Interceptor melalui `CombatSystem.select_primary_target()`, bukan memilih planet.
- Tombol **MAP** punya tombol **NEXT PLANET / NAV** untuk navigasi; HUD menampilkan **NAV PLANET** dan tidak lagi menyebut planet sebagai combat target. Keyboard `N` tetap dapat mengubah NAV; `TAB` mengunci NPC.
- **TRACK NPC** tombol kamera baru (di bawah): kamera dari posisi stabil pada kapal pemain mengarahkan pandangan ke titik framing antara kapal dan NPC target. **Tidak** mengikuti planet atau mengubah arah terbang. Tekan lagi untuk kembali ke mode kamera sebelumnya.
- **CAM LOCK** tetap merupakan stabilizer orientasi yang independen dari combat lock, seperti desain sebelumnya. Mode CAMERA cycle juga dapat mencapai TRACK NPC ketika NPC lock valid.
- Ketika NPC hancur/respawn atau di-UNLOCK, TRACK NPC otomatis kembali ke mode sebelumnya; tidak menunjuk target kosong.
- Auto Engage dibatasi agar tidak mulai ketika enemy NPC masih inactive/respawn, tetapi rumus flight/mount/shot v0.2.4 dipertahankan.
- Test patch disimpan sebagai artefak percakapan `SRPG4_Sol3D_Combat_v0_2_5_TargetNPC_CameraTrack.zip` dan tidak diunggah ke repository yang dikhususkan untuk laporan.

## Checklist Android v0.2.5 (BELUM dijalankan)
1. Tekan TARGET NPC dan pastikan target = Pirate, bukan planet.
2. Tekan TRACK NPC; gerakkan kapal atau tunggu enemy zigzag; fokus kamera harus mengikuti area NPC.
3. Tekan LOCK untuk pandangan world-stable, terpisah dari target NPC.
4. MAP → NEXT PLANET mengubah NAV, tidak membatalkan AUTO FIRE/AUTO ENGAGE atau mengubah TRACK NPC.
5. AUTO FIRE → AUTO → CANCEL → joystick override tetap normal, inersia terjaga.
6. Saat musuh destroyed/respawn, tracking tidak terkunci pada ruang kosong.
7. Periksa tombol di viewport Godot Android landscape, terutama TRACK NPC dekat PITCH STD.
8. Jika parser error, kirim tangkapan layar Output/Debugger nomor baris.

**Catatan:** Belum ada multi-NPC selector, AI station/docking 3D, atau outfitting weapon. TARGET NPC saat ini memilih satu NPC test.
