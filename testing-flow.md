# Testing Bootstrap — [NAMA_PROJECT]

> **Satu file perintah.** Berikan ke AI agent mana pun (Hermes / Codex / Claude / dll).
> Agent menjalankan screening, lalu menyiapkan & mengisi folder kerja `.testapp`.
> Setelah itu kamu yang evaluasi hasil & memutuskan (sidang).

## Konfigurasi
- **Project path:** `[PROJECT_PATH]`
- **Testing folder (hasil):** `[PROJECT_PATH]/.testapp` (folder tersembunyi, tidak ikut deploy/publik)
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
> Uji validitas skenario — jawab tiga pertanyaan ini sebelum skenario dikirim ke agent:
> 1. Apakah ini bisa terjadi tanpa rekayasa teknis — secara alami?
> 2. Apakah orang yang tidak tahu kodenya bisa melakukan ini?
> 3. Adakah kondisi nyata (fisik / teknis / manusiawi) yang memicu ini?
>
> Kalau salah satu jawabannya tidak → skenario dibuat dari kode, bukan dari realita. Jangan lanjutkan.

### Tiga Jenis Skenario

| Jenis | Definisi | Contoh nyata |
|-------|----------|--------------|
| `HAPPY` | Alur valid berjalan normal | Petugas input data dengan benar, koneksi stabil |
| `FAILED` | Aksi/input yang tidak valid atau tidak diizinkan | Petugas coba akses data yang bukan miliknya |
| `CRAZY` | Perilaku manusia nyata yang ekstrem tapi mungkin terjadi | Petugas klik submit dua kali karena loading lambat, sinyal putus di tengah input, buka dua tab sekaligus |

> **CRAZY bukan berarti mustahil.** Ini adalah hal yang *wajar* dilakukan orang sungguhan di kondisi nyata — bukan skenario rekayasa teknis.
>
> **Panduan menulis CRAZY** — mulai dari pertanyaan:
> *"Apa yang dilakukan aktor ini ketika sistem tidak merespons sesuai harapan mereka dalam 3 detik pertama?"*
>
> Pola CRAZY yang sering terlewat:
> - **Reload** → form kosong, data hilang, state reset
> - **Klik berulang** → submit dobel, data duplikat, state conflict
> - **Buka tab baru** → session conflict, data tidak sinkron
> - **Koneksi putus di tengah aksi** → data setengah tersimpan, state menggantung
> - **Ganti perangkat** → session mati, progres hilang
> - **Dua orang akses data yang sama bersamaan** → race condition, data tertimpa
> - **Layar kecil / HP lama** → elemen tidak terjangkau, button tidak berfungsi
> - **Klik cepat berurutan** → popup conflict, state tidak terisolasi

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
   - Perhatikan khusus skenario berlabel ⚠️ di actors.md — prioritaskan untuk gap analysis.
6. Perhatikan titik persinggungan antar aktor (Bagian C di actors.md) — ini lokasi bug paling rawan
   yang tidak terdeteksi karena tiap aktor ditest secara terpisah.
7. JANGAN buat daftar aktor baru dari kode — aktor sudah didefinisikan user di actors.md.
8. JANGAN mulai test. Laporkan gap analysis + usulan alur, tunggu evaluasi user.
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

---

### B2 — isi `actors.md`

> **`actors.md` ditulis oleh user, bukan di-generate agent dari kode.**
> Agent boleh mengisi base-app.md (teknis), tapi actors.md harus dimulai dari brief user.
> Kalau belum ada, agent boleh mengusulkan draft — tapi user harus review & koreksi sebelum test dimulai.
>
> **Aturan keras:** Skenario tidak boleh ditulis sebelum Bagian A (Verifikasi Aktor) selesai diisi.
> Skenario yang dibuat sebelum verifikasi selesai = skenario dari asumsi = kemungkinan besar tidak akan menemukan bug nyata.

```markdown
# Daftar Aktor — [NAMA_PROJECT]

> Aktor didefinisikan dari perspektif nyata, bukan dari role di kode.
> Isi berdasarkan siapa mereka di dunia nyata dan bagaimana mereka berperilaku.

---

## [NAMA_AKTOR] — [PERAN SINGKAT DI DUNIA NYATA]

> Contoh yang benar: "Petugas lapangan yang input data di lokasi sambil pegang dokumen fisik"
> Contoh yang salah: "User dengan role Admin yang bisa akses menu X"

---

### Bagian A — Verifikasi Aktor (wajib selesai sebelum tulis skenario)

> Jawab semua pertanyaan ini dari sumber nyata — bukan dari asumsi.
> Kalau tidak tahu jawabannya, tandai ⚠️ dan cari tahu dulu sebelum lanjut.

#### A1. Siapa mereka di dunia nyata?
- **Jabatan / peran sehari-hari:** ___
- **Di mana mereka biasanya bekerja saat pakai sistem ini:** ___
  *(contoh: di kantor dengan meja, di lapangan sambil berdiri, di perjalanan)*
- **Seberapa sering mereka pakai sistem ini:** ___
  *(contoh: setiap hari, seminggu sekali, hanya saat ada kasus tertentu)*
- **Tingkat kenyamanan dengan teknologi:** ___
  *(contoh: tidak terbiasa, cukup terbiasa, sangat terbiasa)*

#### A2. Kondisi fisik & teknis nyata
- **Perangkat yang mereka pakai:** ___
  *(HP / laptop / keduanya — sebutkan ukuran layar atau spesifikasi kalau tahu)*
- **Koneksi internet di lokasi kerja mereka:** ___
  *(stabil / sering putus / tergantung lokasi / tergantung jam)*
- **Apakah mereka satu-satunya yang akses data ini?** ___
  *(ya / tidak — kalau tidak, siapa lagi dan apakah bisa bersamaan?)*
- **Apakah sistem ini dipakai di browser tertentu?** ___

#### A3. Kondisi manusiawi nyata
- **Tekanan waktu saat pakai sistem ini:** ___
  *(santai / sedang / terburu-buru — jelaskan konteksnya)*
- **Apakah mereka sering terganggu saat mengerjakan ini?** ___
  *(contoh: sambil terima telepon, sambil layani orang lain, harus cepat karena antrean)*
- **Kebiasaan yang mungkin terbawa:** ___
  *(contoh: klik cepat tanpa baca pesan, refresh kalau loading lama, buka banyak tab, jarang logout)*
- **Apa yang mereka lakukan kalau sistem tidak merespons dalam 3 detik?** ___
  *(ini pertanyaan paling penting untuk menemukan skenario CRAZY)*

#### A4. Konteks sebelum & sesudah pakai sistem
- **Apa yang mereka kerjakan sebelum buka sistem ini?** ___
  *(contoh: terima dokumen fisik, dapat instruksi dari atasan, baru dari lapangan)*
- **Apa yang mereka lakukan setelah selesai di sistem ini?** ___
  *(contoh: cetak dokumen, lapor ke atasan, serahkan ke petugas lain)*
- **Apakah ada petugas lain yang terlibat dalam alur yang sama?** ___
  *(kalau iya, siapa — dan di titik mana alur mereka bersinggungan?)*

#### A5. Sumber pengetahuan tentang aktor ini

Tandai semua yang berlaku:
- [ ] ✅ Observasi langsung — melihat mereka bekerja
- [ ] ✅ Wawancara — bertanya langsung ke orangnya
- [ ] 🔶 Feedback / complaint dari sistem lama atau sistem serupa
- [ ] 🔶 Cerita dari orang yang dekat dengan aktor ini
- [ ] ⚠️ Asumsi pembuat — belum diverifikasi ke orang nyata

> **Penting:** Skenario yang bersumber dari ⚠️ harus diuji lebih dulu dari yang lain —
> bukan karena lebih penting, tapi karena paling mungkin meleset dari realita.

---

### Bagian B — Profil & Skenario

> Tulis bagian ini hanya setelah Bagian A selesai.
> Semua yang ada di sini harus bisa dilacak ke jawaban di Bagian A.

**Tujuan utama saat buka sistem ini:**
[Apa yang ingin mereka capai — biasanya dalam kondisi seperti apa]

**Cakupan di aplikasi:**
- [Halaman atau fitur yang mereka akses dalam pekerjaan sehari-hari]

**Titik risiko nyata:**
- [Di mana alur ini paling mungkin putus untuk aktor ini — berdasarkan kondisi di Bagian A]

#### Skenario HAPPY — Alur valid berjalan normal
> Kondisi: aktor dalam keadaan normal, koneksi stabil, tidak terburu-buru, perangkat berfungsi baik.

| # | Kondisi awal | Aksi aktor | Ekspektasi nyata | Sumber |
|---|---|---|---|---|
| H1 | ___ | ___ | ___ | ✅ / 🔶 / ⚠️ |
| H2 | ___ | ___ | ___ | ✅ / 🔶 / ⚠️ |

#### Skenario FAILED — Aksi atau input tidak valid / tidak diizinkan
> Kondisi: aktor melakukan sesuatu yang salah — bukan karena jahat, tapi karena salah paham,
> terburu-buru, atau sistem sebelumnya mengizinkan ini.

| # | Kondisi awal | Aksi aktor | Ekspektasi nyata | Sumber |
|---|---|---|---|---|
| F1 | ___ | ___ | ___ | ✅ / 🔶 / ⚠️ |
| F2 | ___ | ___ | ___ | ✅ / 🔶 / ⚠️ |

#### Skenario CRAZY — Perilaku manusia nyata yang ekstrem tapi wajar terjadi
> Kondisi: aktor dalam tekanan, koneksi buruk, perangkat lambat, atau melakukan sesuatu yang
> wajar dilakukan manusia saat sistem tidak merespons sesuai harapan.
> Mulai dari: "Apa yang dilakukan aktor ini ketika sistem tidak merespons dalam 3 detik pertama?"

| # | Kondisi nyata pemicu | Aksi aktor | Ekspektasi nyata | Sumber |
|---|---|---|---|---|
| C1 | ___ | ___ | ___ | ✅ / 🔶 / ⚠️ |
| C2 | ___ | ___ | ___ | ✅ / 🔶 / ⚠️ |

---

### Bagian C — Catatan Lintas Aktor

> Isi ini kalau aktor ini bersinggungan dengan aktor lain dalam satu alur.
> Titik persinggungan adalah lokasi paling rawan bug yang tidak terdeteksi
> karena masing-masing aktor ditest secara terpisah.

| Aktor lain | Titik persinggungan | Risiko nyata |
|---|---|---|
| ___ | ___ | ___ |

---
[Ulangi seluruh struktur di atas untuk setiap aktor]

---

## Catatan pembaruan actors.md

> Dokumen ini hidup — diperbarui setiap ada informasi baru tentang aktor.
> Skenario lama yang terbukti meleset dari realita harus dikoreksi, bukan dihapus.

| Tanggal | Aktor | Perubahan | Sumber pembaruan |
|---|---|---|---|
| ___ | ___ | ___ | ___ |
```

---

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

---

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
| ID | Aktor | Jenis | Kasus | Ekspektasi (nyata) | Hasil (app) | Status | Lokasi (file:baris) | Sumber | Catatan |
|----|-------|-------|-------|--------------------|-------------|--------|---------------------|--------|---------|
| F1 |       | HAPPY |       |                    |             | LOLOS  |                     | ✅/🔶/⚠️ |        |
| F2 |       | FAILED|       |                    |             | GAGAL  |                     | ✅/🔶/⚠️ |        |
| F3 |       | CRAZY |       |                    |             | GAGAL  |                     | ✅/🔶/⚠️ |        |

> **Ekspektasi (nyata):** apa yang seharusnya terjadi menurut logika dunia nyata — ditulis SEBELUM melihat kode.
> **Hasil (app):** apa yang benar-benar terjadi saat test dijalankan.
> **Sumber:** ✅ terverifikasi langsung | 🔶 dari sistem lama/cerita | ⚠️ asumsi pembuat
> Kalau Ekspektasi = Hasil selalu → tandanya skenario dibuat dari kode, bukan dari realita.

**Jenis:** `HAPPY` = alur valid normal | `FAILED` = input/aksi invalid | `CRAZY` = perilaku ekstrem nyata

### Gap Analysis
Skenario nyata yang dibutuhkan aktor tapi tidak ada handler-nya di kode:
- [ ] [deskripsikan gap — ini lebih berharga dari test yang gagal]

Skenario berlabel ⚠️ yang terbukti meleset dari asumsi:
- [ ] [catat di sini — jadi bahan koreksi actors.md]

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
2. **Sidang daftar aktor** — tambah, hapus, atau koreksi di `actors.md`.
   > `actors.md` adalah dokumen milikmu — agent hanya mengusulkan, kamu yang memutuskan.
   > Prioritaskan koreksi pada skenario berlabel ⚠️ — paling mungkin meleset dari realita.
3. Pilih aktor + alur dari gap analysis → isi `prompt.md`, kirim ke agent.
   > Prioritaskan gap (skenario tanpa handler) di atas skenario yang sudah punya kode — gap lebih mungkin temukan bug nyata.
4. Agent: **tulis ekspektasi dulu → buat test case → jalankan runner → catat ke `laporan/`** (copy template → `laporan/YYYY-MM-DD-[aktor]-[alur].md`).
5. Kamu sidang via checkbox keputusan — perhatikan kolom Ekspektasi vs Hasil.
   > Kalau Ekspektasi = Hasil selalu → evaluasi ulang skenario, kemungkinan dibuat dari kode bukan realita.
6. Setelah sidang, perbarui `actors.md` — koreksi skenario yang terbukti meleset, tambah yang baru dari temuan laporan.
7. Ulangi untuk aktor/alur lain — folder `.testapp/` terus bertambah, tidak ikut rilis.