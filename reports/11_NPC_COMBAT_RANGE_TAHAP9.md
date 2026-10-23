# Space RPG 4 Full 3D — Mapping Tahap 9: NPC Combat Controller, Weapon Range & Tactics

**Audit 23 Oktober 2026 | Source Space RPG 4 v0.9995(44)**  
Sumber primer: `smali.zip`, SHA-256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`. Seluruh temuan kode di bawah dari **pemeriksaan statis**, bukan tes runtime. **Belum ada perubahan APK atau Godot.**

## Temuan penting: kelas `Lm` adalah pusat keputusan tempur untuk controller `Lb`

Mapping sebelumnya sudah menemukan `Ly7` (threat set), `Lx7` (motion), dan `Lf8/Lzj` (senjata). Tahap ini mengidentifikasi jembatan di antaranya:

- `Lb` adalah container state/task per kapal dengan `LinkedList<Lk>`. `Lb.a(J)` mengeksekusi `Lk.g(J)` **pada task pertama** jika kapal masih aktif; `Lb.d(Lk;)` mendorong task baru ke depan dan menangguhkan task sebelumnya; `Lb.g()` melepas task dan melanjutkan tugas yang tertunda. Ini struktur **stack of orders**, bukan satu finite state enum global.
- `Lm extends Lk` memiliki jalur *combat decision* dalam `Lm.g(J)` (fisik baris ~1305–7206). Ia membaca `w7.m:Ly7` untuk target, `w7.n:Lf8` untuk senjata, `w7.j:Le8` untuk status kapal, `w7.l:Lx7` untuk navigasi, dan dapat memilih strategi manuver subclass `Lc`.
- `Lc` merupakan base maneuver planner; turunan `Ld, Le, Lf, Lg, Lh, Li` memakai kombinasi target, range, posisi `Lu6`, course `Lx7` dan timer. **Nama taktik human-readable tiap subclass belum dipastikan**; jangan langsung melabelinya Orbit/Keep Range/Flee original.

## 1. Seleksi target tidak acak terus-menerus: ada scoring & periode evaluasi

Dalam `Lm.g(J)`:

1. Timer `Lm.b` dikurangi delta tick. Ketika `<=0`, nilainya direset ke **`300 + Random.nextInt(300)`**. HashMap skor `Lm.d` dan ArrayList kandidat `Lm.e` dikosongkan lalu diisi lagi dari `Ly7.c` (LinkedHashSet ancaman).
2. Setiap kandidat dinilai dengan `Lm.h(Lw7;FF)[F` (fisik baris ~7208–7464). Hasil pertama disimpan di `Lm.d` dan daftar diurutkan memakai comparator `Ll`. Nilai `Lh1.y(target)` yang dipakai di skor bukan damage langsung dan perlu trace jika ingin menamai kekuatan/kapasitas secara resmi.
3. `Lm.h` membagi jarak ke beberapa pita relatif terhadap **`Lf8.r` milik target**: `0.8×r`, `1.2×r`, `1.7×r`, `2.2×r`; memberi faktor skor **1, 0.85, 0.55, 0.15, 0.05** untuk zona-zona itu. Kandidat yang **sedang membidik kapal ini** menerima bonus dari `Lu7.f × 2` dalam cabang tertentu.
4. Jika target utama `Ly7.b` tidak hidup, berkoordinat NaN/Infinity, atau terlalu jauh dari `Lf8.r`, ada jalur mengganti/membersihkan target. Ini juga merupakan guard terhadap state tidak valid.
5. Timer retarget kedua `Lm.c` direset menjadi **`5000 + Random.nextInt(10000)`** saat habis; ia memanggil `Lm.k()` untuk memilih target dari kandidat yang dinilai.
6. `Lm.k()` (baris ~10070–10482) memiliki beberapa jalur: dapat memprioritaskan target berkelas tertentu (`Lu7.f <= 1` bila owner class 0), mengevaluasi jarak **squared**, dan membandingkan nilai `Lh1.y`. Jadi klaim “NPC selalu memilih musuh terdekat” tidak benar untuk semua cabang.

**Catatan tick:** konstanta di source telah diverifikasi; interpretasinya sebagai milidetik aktual masih perlu konfirmasi laju `Lb.a(J)` dan runtime (meskipun pola kode menyarankan ms).

## 2. `Lf8.r` bukan radius sensor umum: berasal dari senjata

Pemeriksaan `Lf8.h([I)V`, sekitar fisik baris **~5008–5029**:

```text
for each installed mount zj:
    if mount exists and mount.a != 2:
        f8.r = max(f8.r, mount.m)
```

- Ini membuktikan `Lf8.r` adalah **agregat nilai range mount non-tipe-2** yang digunakan pada perhitungan pertempuran, dan **bukan jarak sensor universal**. `Lzj.m` diinisialisasi dengan default 300 tetapi dapat ditimpa factory senjata; tracing final modifiers/turret slot tetap diperlukan.
- `Lm.g` membandingkan `own f8.r` dengan `1.3 × target f8.r`, juga mengevaluasi skor kapal via `Lh1.y`. Hal ini mendukung adanya pilihan manuver relatif terhadap **keunggulan jangkauan dan kemampuan kapal**.
- **Jangan port f8.r sebagai max firing range semua senjata sekaligus di Godot.** Pada remake 3D, setiap turret tetap punya `effective_range` sendiri, yaw/pitch firing arc dan hull occlusion. Nilai maksimum hanya membantu keputusan jarak navigasi, bukan izin semua turret menembak.

**Trace senjata terpisah:** `Lf8.a(J)` (fisik baris ~60–410) mengurangi cooldown array, mengiterasi `Lf8.b:[Lzj`, lalu memanggil `Lzj.f(JLw7;)` menggunakan `Ly7.b` sebagai target jika status cocok. Combat decisions di `Lm` **berbeda dari update per-mount dan tembakan**. Turret tanpa target tidak boleh diasumsikan otomatis bisa menembak.

## 3. Perilaku evasive asli ternyata lebih kompleks dari yang diduga di Tahap 8

Dalam `Lm.g` (fisik ~2430–2692), setelah pengecekan flags/task, ada cabang terkait:
- `Le8.b < 0.35 × Le8.f` (jalur hull/health candidate; nama parameter harus dibuktikan silang).
- `Lb8.b < 0.1 × Lb8.d` (shield candidate), dalam kombinasi dengan ambang pertama.
- `Le8.c < 0.5 × Le8.g` (another defense component candidate).
- `Le8.b < 0.15 × Le8.f` dan perbandingan besaran ancaman `Lm.f` terhadap `Lm.h`.
- `Le8.d:[F` (array kondisional kerusakan/per-bagian) dan `Le8.e` pada branch lanjut.

Cabang ini memicu perubahan jalur manuver/menjauh pada beberapa kondisi, sehingga **benar ada taktik defensif bersyarat di combat AI original**. Tetapi ini **belum membuktikan NPC memiliki state “FLEE selamanya hingga keluar sistem”**. Sebagian operasi juga bergantung pada opsi `La.a`, `Lm.n`, class `Lu7.f`, dan status fighter/parent.

`Lm.i(Lw7;F)Z` (baris ~8947–9462) mengecek `Lf8.p/q/r/u`, status target, class `Lu7.f`, waktu `Lm.o`, dan stat `Le8`, sehingga tampak sebagai **predikat syarat taktik/weapon**, bukan probabilitas hit langsung. Semantik bitmask `Lf8.q` dan `Lf8.u` masih harus dipetakan sebelum memberi nama “missile confidence” atau “aggro”.

## 4. Strategi gerak bersifat multi-mode dan menghormati fisika 2D

- `Lm.g` dapat menginstansiasi `Li, Lf, Ld, Le, Lg`; sebagian manuver lain memakai `Lh`. Semua adalah `Lc` subclass yang membuat keputusan orientasi, kecepatan, steering berdasarkan posisi target relatif dan `Lf8.r`.
- `Lm.j(Z)` (~9756–9968) membangun vektor dari owner ke target atau centroid beberapa ancaman, lalu menyimpan arah di `Lm.j` menggunakan `Lsj.c(FF)`. Ini bukti **perhitungan arah grup**; bukan posisi/kecepatan 3D.
- `Li.b(JLw7;FF)` dapat menulis `x7.k/l` dan `x7.e`, dan `Ld.b` memodifikasi `x7.k/l` (target course). `x7.a(J)` dan `x7.b()` melakukan physics movement terpisah.
- Saat port 3D, terjemahkan sistem tujuan menjadi **steering controller 3-sumbu**, bukan langsung set posisi kapal. Motion yang lebih lambat untuk capital ships harus dipertahankan; angular velocity/hull collision adalah elemen Godot baru.

## 5. Desain yang disepakati untuk AUTO ENGAGE Godot 3D

**Tidak** menyalin NPC AI 2D apa adanya. `PlayerAutoEngage` dan `NpcCombatBrain` memakai `WeaponMount3D`, `TargetSelector`, `ShipFlightController3D` bersama, tetapi otoritas perintah berbeda.

```text
player target lock (bukan camera lock)
  -> AUTO ENGAGE (opt-in)
  -> pilih Approach / Keep at Range / Orbit
  -> hitung feasible band dari SEMUA mounted weapons + modifier
  -> flight assist mengejar posisi tanpa teleport / reset inertia
  -> per weapon: range, arc yaw/pitch, hull raycast, cooldown, energy, ammo
  -> projectile uses physical lead + collision; beam / missile specialized
  -> CANCEL / joystick override / docking / target invalid -> stop auto navigation
```

**Firing arc 3D:** mount di sisi kiri boleh hanya `180°` relatif bidang lokalnya; target di kanan belakang yang tertutup hull tidak dapat ditembak bahkan jika `range` sesuai. Targeting AI memperbaiki leading, **bukan tambah range, bukan bypass occlusion, bukan auto-hit**. Jangan raycast mengecualikan seluruh kapal penembak jika perlu mendeteksi hull obstruction: gunakan mask dan first-hit check yang benar dari muzzle.

**Pilihan jarak:** `effective_range` senjata jauh (missile) tidak boleh membuat cannon pendek menembak terlalu awal. Profil navigasi mengevaluasi damage potensial/senjata yang memiliki arc dan LOS, tetapi tidak memaksakan kapal selalu berada di range terpanjang.

## 6. Tabel tes Tahap 9 — SEMUA BELUM DIJALANKAN

| ID | Kasus | Harapan |
|---|---|---|
| T9-A | NPC 3 ancaman | Kandidat diurutkan ulang hanya pada tick evaluasi, bukan setiap frame |
| T9-B | Target mati / NaN / Infinity | Tidak freeze, bersihkan lalu retarget |
| T9-C | Kapal punya missile 2500 + cannon 500 (contoh unit Godot) | Cannon baru aktif <=500 dan arc valid |
| T9-D | Turret kiri, target kanan | Tidak menembak melewati lambung walau target di bawah max range |
| T9-E | Pemain menerbangkan ship dengan inertia lalu Cancel Auto | Kembali manual tanpa snap/stop |
| T9-F | NPC low defense | Pilihan manuver defensif, **bukan** despawn acak |
| T9-G | Perbandingan weapon ranges NPC | AI mencoba mempertahankan jangkauan efektif ketika taktiknya sesuai |
| T9-H | NPC target berpindah cepat & AI targeting | Leading lebih baik; tetap bisa miss |
| T9-I | Carrier fighter dan parent | Target/sensor tidak menempel atau ikut transform parent saat bebas |
| T9-J | Retarget ticks & save/load | Deterministik sesuai seed/server saat multiplayer |

## 7. Belum selesai / Tahap 10

- Validasi nama `Le8.b/f/c/g`, `Lb8.b/d` dan arti `Lm.i` mask `Lf8.q`, `f8.u`.
- Penelusuran formal tiap `Lc` subclass untuk menamai taktik asli secara aman.
- Trace `Lzj.f`: cek range per turret, azimuth, cooldown/energi/amunisi, beam vs torpedo sebelum implementasi.
- Trace area deteksi sensor yang **berbeda** dari range senjata dan event retaliation.
- Sesudah mapping cukup, buat uji **satu NPC 3D + dua hardpoint 3D** (kiri dan atas), di proyek Godot **hanya atas permintaan implementasi eksplisit**.

**Kesimpulan:** Tahap 9 mengubah dua asumsi sebelumnya: `f8.r` terbukti agregat range senjata, bukan radius sensor, dan NPC original memiliki *combat maneuver planning* plus cabang defensif bersyarat—tetapi belum dipastikan sebagai automatic flee antar-sistem. Laporan ini menyatakan klaim original vs rancangan remake secara terpisah.
