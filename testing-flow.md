# Testing Bootstrap — [NAMA_PROJECT]

> **Satu file perintah.** Berikan ke AI agent mana pun (Hermes / Codex / Claude / dll).
> Agent menjalankan screening, lalu menyiapkan & mengisi folder kerja `.testapp`.
> Setelah itu kamu yang evaluasi hasil & memutuskan (sidang).

## Konfigurasi
- **Project path:** `[PROJECT_PATH]`
- **Testing folder (hasil):** `[PROJECT_PATH]/.testapp`  (folder tersembunyi, tidak ikut deploy/publik)
- **Aturan keras:** JANGAN ubah kode produksi.

---

## BATAS AGENT (apa yang boleh dilakukan agent)
1. **Screening** — baca kode, profil project. Tanpa ubah kode.
2. **Identifikasi aktor** — tentukan siapa saja yang menggunakan sistem di dunia nyata.
3. **Membuat test case** — tulis test sesuai cakupan yang disepakati.
4. **Menjalankan test** — jalankan runner jika bisa.
5. **Mencatat hasil** — tulis ke file laporan di dalam `.testapp/`.

> Di luar kelima itu (ubah produksi, hapus data) = dilarang.
> Keputusan "bug / behavior / bukan masalah" tetap di tangan user.

---

## PRINSIP UTAMA — SKENARIO DARI REALITA, BUKAN DARI KODE

> **Skenario dibuat dari perspektif perilaku nyata aktor, bukan dari fitur yang terlihat di aplikasi.**

Urutan yang benar:
1. Siapa aktor ini di dunia nyata?
2. Apa yang mereka lakukan sebelum, saat, dan setelah menggunakan sistem?
3. Apa kebiasaan, keterbatasan, dan kondisi nyata mereka? (sinyal jelek, terburu-buru, pakai HP lama, dll)
4. Tulis ekspektasi: apa yang *seharusnya* terjadi menurut logika dunia nyata — sebelum melihat kode.
5. Baru petakan ke alur aplikasi. Bandingkan ekspektasi vs realita kode.

**Dilarang:** membuat skenario karena melihat tombol, field, atau endpoint di kode.
> Uji: bisakah skenario ini ditulis oleh orang yang belum pernah melihat kodenya? Kalau tidak bisa, skenarionya salah arah.

### Tiga Jenis Skenario

| Jenis | Definisi | Contoh nyata |
|-------|----------|--------------|
| `HAPPY` | Alur valid berjalan normal | Wasit input skor dengan benar, koneksi stabil |
| `FAILED` | Aksi/input yang tidak valid atau tidak diizinkan | Wasit coba akses match yang bukan miliknya |
| `CRAZY` | Perilaku manusia nyata yang ekstrem tapi mungkin terjadi | Wasit klik submit dua kali karena loading lambat, sinyal putus di tengah input, buka dua tab sekaligus |

> **CRAZY bukan berarti mustahil.** Ini adalah hal yang *wajar* dilakukan orang sungguhan di kondisi nyata — bukan skenario rekayasa teknis.

---

## BAGIAN A — PERINTAH SCREENING (agent jalankan pertama)

> **Penting:** `actors.md` dan daftar skenario adalah milik user — bukan hasil generate dari kode.
> Agent screening hanya boleh membaca kode untuk membandingkan ekspektasi yang sudah ada vs realita implementasi.

```
SCREENING (read-only, no edits, no test run yet):
1. Baca actors.md yang sudah ada — pahami ekspektasi nyata per aktor.
2. Baca struktur project di [PROJECT_PATH]: framework, folder inti, logika inti.
3. Profil BASE APP: nama, fungsi, alur utama, pemakai, aturan keras — catat ke base-app.md.
4. Temukan cara menjalankan test (runner) — catat perintah pastinya.
5. Lakukan GAP ANALYSIS: untuk setiap skenario nyata di actors.md, apakah kode menanganinya?
   - Catat skenario yang tidak ada handler-nya (ini calon bug paling berharga).
   - Catat skenario yang ada handler-nya tapi perlu diverifikasi lewat test.
6. JANGAN buat daftar aktor baru dari kode — aktor sudah didefinisikan user di actors.md.
7. JANGAN mulai test. Laporkan gap analysis + usulan alur, tunggu evaluasi user.
```

---

## BAGIAN B — STRUKTUR FOLDER & ISI (agent yang buat)

```
.testapp/
  base-app.md        ← hasil screening (B1)
  actors.md          ← daftar aktor & perilaku nyata (B2)
  prompt.md          ← template brief test (B3)
  laporan/
    template.md      ← template laporan (B4)
```

### B1 — isi `base-app.md`
```
# BASE APP — [NAMA_PROJECT]
- Nama:
- Fungsi:
- Stack:
- Path produksi:
- Alur utama:
- Pemakai:
- Aturan keras:
- Cara jalanin test (runner):
- Alur prioritas test (usulan agent):
```

### B2 — isi `actors.md`

> **`actors.md` ditulis oleh user, bukan di-generate agent dari kode.**
> Agent boleh mengisi base-app.md (teknis), tapi actors.md harus dimulai dari brief user.
> Kalau belum ada, agent boleh mengusulkan draft — tapi user harus review & koreksi sebelum test dimulai.

```
# Daftar Aktor — [NAMA_PROJECT]

> Aktor didefinisikan dari perspektif nyata, bukan dari role di kode.
> Isi berdasarkan siapa mereka di dunia nyata dan bagaimana mereka berperilaku.

---

## [NAMA_AKTOR] — [PERAN SINGKAT]

**Siapa di dunia nyata:**
[Deskripsi singkat — jabatan, konteks kerja, kondisi fisik saat pakai sistem]

**Tujuan utama saat buka aplikasi ini:**
[Apa yang ingin mereka capai, biasanya dalam kondisi seperti apa]

**Kondisi nyata:**
- Perangkat: [HP/laptop, spesifikasi rendah/tinggi]
- Koneksi: [stabil/sering putus/tergantung lokasi]
- Tekanan waktu: [santai/terburu-buru/di tengah acara]
- Kebiasaan: [klik cepat, sering salah input, jarang baca instruksi, dll]

**Cakupan di aplikasi:**
- [Fitur/halaman yang mereka akses]

**Titik risiko nyata:**
- [Skenario nyata yang bisa menyebabkan masalah bagi aktor ini]

**Skenario yang mungkin terjadi:**
- HAPPY: [contoh alur normal mereka]
- FAILED: [contoh kegagalan yang mungkin mereka alami]
- CRAZY: [contoh perilaku ekstrem yang wajar mereka lakukan]

---
[Ulangi untuk setiap aktor]
```

### B3 — isi `prompt.md`
```
# Prompt Testing (template brief — isi & kirim ke agent tiap mau test)

Project: [NAMA_PROJECT], path [PROJECT_PATH]
Aktor: [nama aktor — tulis deskripsi singkat perilaku nyatanya dari actors.md]
Tugas: Testing alur: [NAMA_ALUR]
Jenis skenario: [HAPPY / FAILED / CRAZY — boleh lebih dari satu]
Konteks nyata: [jelaskan kondisi nyata aktor saat alur ini terjadi, bukan sekadar "user klik tombol X"]
Cakupan:
1. [HAPPY — kondisi normal → hasil yang benar]
2. [FAILED — kondisi gagal → sistem menolak/menangani dengan benar]
3. [CRAZY — perilaku ekstrem nyata → sistem tidak crash/korup/kehilangan data]
Kriteria lolos: semua ter-cover, output jelas (hijau/merah).
Jangan ubah kode produksi. Pakai runner: [PERINTAH_RUNNER].
Lapor ke laporan/, pakai template.md — sebut file:baris tiap temuan.
```

### B4 — isi `laporan/template.md`
```
## Laporan Testing — [ALUR]
Tanggal:
Aktor: [nama aktor yang diuji]
Cakupan:

### Ringkasan
- Test ditulis:
- Lolos (hijau):
- Gagal (merah):
- Crazy (ekstrem — perlu dicatat meski tidak crash):

### Temuan
| ID | Aktor | Jenis | Kasus | Ekspektasi (nyata) | Hasil (app) | Status | Lokasi (file:baris) | Catatan |
|----|-------|-------|-------|--------------------|-------------|--------|---------------------|---------|
| F1 |       | HAPPY |       |                    |             | LOLOS  |                     |         |
| F2 |       | FAILED|       |                    |             | GAGAL  |                     |         |
| F3 |       | CRAZY |       |                    |             | GAGAL  |                     |         |

> **Ekspektasi (nyata):** apa yang seharusnya terjadi menurut logika dunia nyata — ditulis SEBELUM melihat kode.
> **Hasil (app):** apa yang benar-benar terjadi saat test dijalankan.
> Kalau Ekspektasi = Hasil selalu → tandanya skenario dibuat dari kode, bukan dari realita.

**Jenis:** `HAPPY` = alur valid normal | `FAILED` = input/aksi invalid | `CRAZY` = perilaku ekstrem nyata

### Gap Analysis
Skenario nyata yang dibutuhkan aktor tapi tidak ada handler-nya di kode:
- [ ] [deskripsikan gap — ini lebih berharga dari test yang gagal]

### Keputusan
- [ ] F2: bug sungguhan? → perlu perbaikan kode
- [ ] F3: crash karena CRAZY? → perlu di-guard | atau memang behavior yang bisa diterima?

### Rekomendasi
-
```

---

## BAGIAN C — RUNNER (tetap pakai yang ada)
- Jalankan dari `[PROJECT_PATH]` dengan perintah: `[PERINTAH_RUNNER]`
- Jangan ganti runner — pakai yang sudah disediakan project.

---

## BAGIAN D — LANJUTAN (kamu yang kerjakan setelah screening)
1. Evaluasi BASE APP dari agent → benarkan kalau meleset.
2. **Sidang daftar aktor** — tambah, hapus, atau koreksi deskripsi perilaku nyata di `actors.md`.
   > `actors.md` adalah dokumen milikmu — agent hanya mengusulkan, kamu yang memutuskan.
3. Pilih aktor + alur dari gap analysis → isi `prompt.md`, kirim ke agent.
   > Prioritaskan gap (skenario tanpa handler) di atas skenario yang sudah punya kode — gap lebih mungkin temukan bug nyata.
4. Agent: **tulis ekspektasi dulu → buat test case → jalankan runner → catat ke `laporan/`** (copy template → `laporan/YYYY-MM-DD-[aktor]-[alur].md`).
5. Kamu sidang via checkbox keputusan — perhatikan kolom Ekspektasi vs Hasil.
6. Ulangi untuk aktor/alur lain — folder `.testapp/` terus bertambah, tidak ikut rilis.