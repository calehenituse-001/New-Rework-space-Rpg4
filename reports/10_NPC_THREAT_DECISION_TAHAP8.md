# Space RPG 4 Full 3D — Mapping Tahap 8: NPC Threat, Combat Decision & Flight

**Tanggal audit:** 23 Oktober 2026  
**Referensi:** Space RPG 4 v0.9995(44), `smali.zip` SHA-256 `f42dbd7ebab9e1ec50a5fe3a5f1b9d638b1102575f36596e0b928478295129d5`  
**Status:** pemeriksaan statis Smali. **Belum ada pengujian runtime, build Godot, atau APK baru.**

Lanjutan [Tahap 7: NPC/Faction](09_NPC_FACTIONS_SPAWN_TAHAP7.md). Laporan Tahap 8 memisahkan **perilaku yang benar-benar didukung instruksi sumber**, **interpretasi yang belum pasti**, dan **desain baru untuk Godot**.

## 1. Penemuan utama: AI punya event _damage/retaliation_, bukan sekadar radius otomatis

`Ly7` menyimpan kapal pemilik `a:Lw7`, target saat ini `b:Lw7`, dan sekumpulan target/ancaman `c:LinkedHashSet`, plus penghitung `d/e/f` dan timer `g:J`.

- `y7.b(Lw7;)V` (baris fisik ~295–476): setelah pemeriksaan state `q2.o` dan `u0.c.c`, menambahkan kandidat ke set. Jika kandidat bukan flag `H` dan tidak sudah menyimpan kapal asal dalam threat-set, method dapat **memanggil target.b(self)** sebagai respons balik. Ia juga memberi callback ke objek pendukung yang sudah tertaut. Ini menunjukkan rantai retaliation/support, bukan hanya aturan faksi.
- `y7.d(Lw7;)V` (~589–1607): jalur masuk kejadian interaksi/serangan, dengan pengecekan kapal hidup `w7.z`, faktor faksi, player/parent, penghitung dan _cooldown_. Untuk sebagian kasus, nilai `d`/`e` naik dan dibatasi **20** atau turun sampai **-10**; `e > 4` dan timer `g <= 0` membuka jalur respons kelompok/sekutu tertentu. Setelah memproses jalur itu `g` diisi **0x7D0 = 2000** unit waktu. **Jangan menyimpulkan setiap 2 detik NPC otomatis scanning**, karena timer ini adalah _cooldown_ di cabang spesifik method.
- `y7.d()` juga menambah `f` dan memakai threshold lokal **3 / 16 / 24** pada cabang berbeda saat mempertimbangkan apakah kandidat baru ditambahkan. Ini adalah _branch thresholds_, **belum sah diberi label “aggro range”, “flee HP”, atau probabilitas agresi final**.
- `y7.h(Lw7;Z)V` (~2230–2379): mengecek faksi lewat `k2.c`, dengan urutan yang bisa berbeda ketika unit berstatus `H`/memiliki parent `J`, lalu memanggil `b` untuk memasukkan ancaman. Bisa meneruskan respons ke sisi lain dengan panggilan rekursif ber-guard `p2=true`.
- `y7.a(J)V` (~28–101 instruksi efektif, sebelum baris 295): mengurangi timer `g`, lalu memeriksa `LinkedHashSet` dan bisa menghapus kandidat `null`, `w7.z=false` (tidak aktif), atau referensi diri; implementasi menghapus **paling satu** kandidat pada satu pemanggilan jalur tersebut.
- `y7.g()V` (~1979–2079): memanggil `Random.nextInt(size)` dan berjalan melintasi `LinkedHashSet`. **Perhatian saat port:** indeks acak dimulai 0, sedangkan loop menambah hitungan sebelum membandingkan; pada sampel statis saat indeks 0, assignment `Ly7.b` mungkin tidak terjadi. Method ini dipanggil antara lain oleh `yc` (misi/script), **jangan asumsikan ini algoritma targeting normal seluruh NPC**. Perlu uji perilaku/konteks pemanggil.

### Bukti pemanggil event damage
Method `y7.d(Lw7;)` dipanggil dari jalur seperti `y4.smali` (~1292, ~1305), `r6.smali` (~2425), `fi.smali` (~9262), serta `b5/o2/r4`. Jalur-jalur tersebut terkait penanganan hit/serangan senjata. Karena itu sebagian respons ancaman bersifat **event-driven ketika terjadi serangan**, tetapi **belum membuktikan seluruh logika deteksi awal**.

## 2. Pergerakan asli: fisika terpisah dari keputusan taktis

`Lx7` merupakan flight/motion controller kapal `w7.l`:

- `x7.a(J)V` (baris ~46–949): memproses transform/heading dan posisi dari komponen `Lu6`, fungsi trigonometrik dan `atan`; bidang koordinat original **2D**. Tujuan/orientasi tidak boleh diterjemahkan langsung ke yaw/pitch/roll 3D tanpa rancangan.
- `x7.b()V` (~1375–1619): akselerasi/kecepatan dengan parameter `x7.d` (limit dasar) dan `x7.b` (pertambahan). Ketika `w7.h0>0`, sebuah batas dikalikan **0.5**; ketika `w7.q0>0`, dikalikan **0.33**; cabang `w7.T` dan `x7.i` juga menggunakan pengali **1.3 / 1.2** pada situasi tertentu. **Nama kondisi sebenarnya belum dipastikan** dan harus dibedakan dari nama game upgrade; ini efek dari source, bukan stats baru yang bebas diadopsi.
- `x7.j()V` (~2411–2490): satu jalur perlambatan mengurangi `x7.e` sebesar `0.6 × x7.b`, dengan guard nilai negatif `-0.25 × x7.d`; **bukan bukti semua NPC otomatis brake**.
- `x7.d()`/`h()`: ada mode temporer dengan `f=true/false`, modifikasi komponen thruster `Lu6`. Nama sebenarnya tak boleh ditebak dari kelas obfuscated.
- Setter target/orientasi/kecepatan tidak hanya berada dalam `x7`; referensi penulis `x7.k/l/e` ditemukan di `u2,z2,d8,o,t,v,y,b3`, dll. Karena itu perlu inspeksi pengambil keputusan yang mengatur controller, bukan hanya mem-port `x7`.

### Yang *belum* terbukti dari source
Tidak ada hasil audit ini yang membuktikan bahwa NPC **selalu orbit**, **keep at range secara pintar**, **retreat ketika HP < 30%**, atau **memiliki sensor radius tertentu**. Seluruh perilaku tersebut **masih desain Godot** hingga ditemukan jalur sumber atau diuji.

## 3. Diagram keputusan usulan untuk remake Godot 3D

Satu `NpcShipBrain3D` menerima target kandidat dari tiga sumber: **sensor**, **event terkena serangan**, dan **perintah/script misi**. Semua masuk lewat validasi faksi, mission permissions, status kapal, PvP/safe-zone dan kemampuan deteksi. Data target terpisah dari kendali kamera.

```text
IDLE / PATROL
  ├─ event damage yang valid ─→ ALERT / RETALIATE
  └─ sensor candidate valid ───→ ALERT
ALERT
  ├─ valid hostile + within sensor / engagement budget → SELECT_TARGET
  └─ no valid target → PATROL
SELECT_TARGET
  ├─ target available → ENGAGE
  └─ target lost → SEARCH / RETURN
ENGAGE
  ├─ within viable weapon range & fire arc → FIRE ELIGIBLE MOUNTS
  ├─ outside range → APPROACH (if nav permitted)
  ├─ arcs blocked by hull → REPOSITION
  ├─ target destroyed/lost → RETARGET / RETURN
  └─ scripted command or safe zone → DISENGAGE
```

**Semua state di atas adalah desain baru**. Efek UI, AI decision dan fisika harus dipisah, supaya kontrol joystick pemain tidak berubah karena NPC AI.

## 4. Weapon-aware Auto Engage: bukan tombol akurasi 100%

Untuk setiap mount aktif:
1. Pastikan slot sesuai jenis senjata dan target legal menurut faksi/misi/PvP.
2. Hitung `effective_range` dari senjata + modifier yang berlaku (Targeting AI **tidak** menambah range; Range Extension hanya kategori yang memenuhi syarat).
3. Hitung `lead_point` proyektil dari pergerakan relatif dan waktu terbang; turret punya batas yaw/pitch serta tracking speed.
4. Raycast dari muzzle ke `lead_point`, kecualikan projectile/target yang tepat tapi **uji occlusion oleh hull sendiri dan objek luar**.
5. Periksa cooldown, energi dan torpedo-ammo, lalu tembak. Tiap proyektil/homing bisa meleset meski sistem AUTO aktif.

**Pemilihan jarak Auto Engage:** jangan memilih jangkauan senjata terbesar begitu saja (misalnya hanya missile jauh jika 80% DPS berada di cannon pendek). Hitung himpunan rentang yang layak dan preferensi _combat profile_ kelas kapal; `KEEP_AT_RANGE`/ `ORBIT` adalah perintah pengguna, dengan koreksi ketika arc/line of sight tidak terpenuhi. Bila tidak ada senjata yang mampu mengenai target, kapal bisa mendekat, mengganti target, atau berhenti sesuai perintah; **tidak menembak saat out-of-range**.

**Override/cancel:** `CANCEL AUTO` atau input joystick disengaja menghentikan navigasi otomatis segera, tapi kecepatan/inertia tidak di-reset. `CAMERA LOCK` tidak dipengaruhi `TARGET LOCK` atau `AUTO ENGAGE`. Docking orbital menghentikan combat autopilot.

## 5. Hasil audit terstruktur dan test matrix

| ID | Jenis tes berikutnya | Ekspektasi |
|---|---|---|
| T8-A | Hit projectile → `y7.d` | Terbentuk ancaman saat pemeriksaan state/faksi lolos |
| T8-B | Beberapa serangan beruntun | Jalur response counters/timer tidak menyebabkan reaksi kelompok terus-menerus |
| T8-C | Target mati/hilang | Threat set dan pilihan target diperbarui secara aman |
| T8-D | F.E.R. vs Pirate & Independent | Hubungan faksi dinamis, tidak friendly fire tanpa izin |
| T8-E | Sensor vs event serangan | Passive detection berbeda dari retaliation event |
| T8-F | NPC mendekat dari luar range | Tidak menembak out-of-range; gunakan physical velocity |
| T8-G | Turret kiri vs target kanan | Tidak menembus lambung/arc restricted |
| T8-H | Joystick override AUTO | Auto navigation batal, inertia terus |
| T8-I | Fighter keluar dari Carrier | Parent & threat propagation valid |
| T8-J | Mission/script target random | Periksa seleksi `y7.g()` ketika hasil random = 0 |

**Status:** Tahap 8 memperdalam event-driven NPC, _branch counters_, motion modifiers dan batas klaim perilaku original. **Belum ada kepastian sensor range/flee/orbit original.** Tahap 9 disarankan: telusuri caller `x7` (class-script NPC/pilot), parameter sensor dan AI combat weapon preference; kemudian buat satu _test harness_ NPC+turret di Godot. Tidak perlu menyalin source Smali lengkap ke repo publik.

Dataset tambahan: [Crosswalk Method Tahap 8](../data/npc_decision_source_crosswalk_stage8.csv) dan [Test Matrix JSON](../data/npc_combat_test_matrix_stage8.json).
