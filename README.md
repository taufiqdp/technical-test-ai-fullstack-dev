# Technical Test: AI Fullstack Developer Intern

## Case

Kamu diminta untuk membangun sistem **ATS (Applicant Tracking System)** sederhana.

Tim HR sering kewalahan menyaring ratusan CV untuk satu lowongan. Fitur yang kamu buat membantu HR menilai kecocokan kandidat dengan requirement lowongan menggunakan AI.

## Waktu

- Batas pengumpulan: **2 × 24 jam** sejak brief ini diterima.

## Stack

| Bagian   | Ketentuan                                                                                                                                            |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend | Next.js (App Router), TypeScript                                                                                                                     |
| Backend  | Hono, TypeScript. Berjalan sebagai service terpisah dari Next.js (bukan Route Handler Next.js). Runtime bebas: Node.js, Bun, atau Cloudflare Workers |
| Database | Postgres/SQLite                                                                                                                                      |
| ORM      | Drizzle ORM                                                                                                                                          |
| AI / LLM | Gratis: Google AI Studio (Gemini) atau model `:free` di OpenRouter                                                                                   |
| Lainnya  | Bebas memilih library UI, package manager, dan struktur repo (monorepo atau dua folder)                                                              |

## Fitur wajib

### 1. Lowongan

- HR bisa membuat lowongan berisi judul, deskripsi, dan daftar requirement. Setiap requirement bertipe `must` (wajib) atau `nice` (nilai tambah).
- HR bisa melihat daftar lowongan beserta detailnya.

> Kamu boleh melakukan seed data dari `dataset/jobs.json`, tetapi form untuk membuat lowongan tetap harus ada.

### 2. Kandidat dan analisis AI

- Di halaman detail lowongan, HR bisa menambah kandidat dengan **mengunggah CV dalam format PDF**. Hanya file PDF yang diterima.
- Backend mengekstrak teks dari PDF, lalu menganalisis CV terhadap setiap requirement lowongan menggunakan LLM.
- Hasil analisis minimal berisi data berikut. Nama field boleh berbeda dan field boleh ditambah.

  ```json
  {
    "profile": {
      "name": "Raka Pratama",
      "years_experience": 3,
      "skills": ["React", "Next.js", "TypeScript"],
      "education": "S1 Informatika"
    },
    "requirements": [
      {
        "requirement": "Menguasai TypeScript",
        "type": "must",
        "verdict": "met",
        "evidence": "Built and maintained an internal analytics dashboard using Next.js (App Router) and TypeScript"
      }
    ],
    "summary": "Ringkasan singkat kecocokan kandidat untuk dibaca HR.",
    "score": 87
  }
  ```

  - `verdict` berisi salah satu dari `met`, `partial`, atau `not_met`.
  - `evidence` adalah kutipan dari CV yang mendukung verdict.
  - `score` bernilai 0–100.

- Kalau file bukan PDF atau teksnya tidak bisa dibaca (misalnya PDF hasil scan atau rusak), tolak dengan pesan error yang jelas.
- Kalau analisis gagal (output AI tidak valid, timeout, kena rate limit, dan sebagainya), kandidat tetap tersimpan dengan status gagal, dan HR bisa menjalankan ulang analisisnya.

### 3. Dashboard kandidat

- Daftar kandidat per lowongan, diurutkan berdasarkan skor, dengan filter status.
- Detail kandidat berisi ringkasan, profil, dan tabel penilaian per requirement beserta buktinya.
- HR bisa mengubah status kandidat dari `new` menjadi `shortlisted` atau `rejected`.
- Ada tampilan untuk kondisi loading, error, dan data kosong.

### 4. Backend dan kualitas

- Semua panggilan LLM dilakukan di backend Hono. API key tidak boleh sampai ke browser dan tidak boleh di-commit. Sediakan `.env.example`.
- Validasi input request dan output LLM. Kami menyarankan zod + `@hono/zod-validator`.
- Backend tidak boleh crash ketika LLM mengembalikan output yang aneh, merespons lambat, atau menolak request.
- Sertakan beberapa automated test untuk logika yang penting. LLM boleh (dan disarankan) di-mock.

Referensi endpoint (boleh disesuaikan):

```
POST  /jobs
GET   /jobs
GET   /jobs/:id
POST  /jobs/:id/candidates
GET   /jobs/:id/candidates?status=&sort=score
PATCH /candidates/:id
POST  /candidates/:id/reanalyze
```

## Bonus (opsional, maksimal 1–2)

- Docker: sediakan `Dockerfile` untuk frontend dan backend, serta `docker-compose.yml`, sehingga `docker compose up --build` menjalankan seluruh aplikasi. Data harus tetap ada setelah container di-restart, dan API key tidak boleh ikut tersimpan di dalam image.
- Hono RPC client (`hc`) agar tipe data frontend dan backend tersambung
- Generator pertanyaan interview berdasarkan gap kandidat
- Streaming ringkasan analisis
- Script evaluasi: beri label beberapa CV secara manual, lalu ukur seberapa sering verdict AI cocok dengan label kamu
- Deploy ke layanan apa pun (misalnya Vercel, Cloudflare Workers, Railway, atau VPS)

## Dataset

Isi folder `dataset/`:

- `jobs.json` berisi 2 lowongan: Frontend Developer dan HR Generalist.
- `cvs/` berisi 8 CV dummy dalam format PDF. File berawalan `fe-` melamar posisi Frontend Developer, sedangkan yang berawalan `hr-` melamar posisi HR Generalist.

Semua data di atas fiktif. Kamu boleh menambah data sendiri, tetapi **jangan memakai CV orang sungguhan**, karena provider AI di free tier bisa memakai data yang dikirim untuk meningkatkan layanannya.

## Tentang API KEY AI gratis

- **Google AI Studio** (aistudio.google.com): buat API key gratis, lalu gunakan model Gemini Flash. Mendukung structured output (JSON schema).
- **OpenRouter** (openrouter.ai): pilih model yang namanya berakhiran `:free`. API-nya kompatibel dengan format OpenAI. Tidak semua model gratis mendukung structured output.
- Keduanya punya rate limit yang cukup ketat. Menanganinya adalah bagian dari tantangan, bukan kesalahanmu.

## Pengumpulan

Kirim link repository GitHub. Kalau repository-nya private, undang user `taufiqdp` dan README.md minimal berisi:

1. Cara menjalankan project dari nol, termasuk env yang dibutuhkan (serta cara menjalankannya dengan Docker, kalau kamu mengambil bonus ini)
2. Gambaran singkat arsitektur dan alur data analisis CV
3. Keputusan teknis yang penting beserta trade-off-nya
4. Bagian yang belum selesai atau yang akan diperbaiki kalau ada waktu lebih

## Penilaian

| Aspek                                                                              | Bobot |
| ---------------------------------------------------------------------------------- | ----- |
| Integrasi LLM: validasi output, retry, penanganan error dan rate limit             | 25%   |
| Aplikasi berjalan end-to-end dan mudah dijalankan                                  | 20%   |
| Judgement produk: apakah hasil AI bisa dipercaya dan dipertanggungjawabkan oleh HR | 15%   |
| Desain API dan data                                                                | 15%   |
| Kualitas kode frontend dan backend                                                 | 15%   |
| UX dasar dan dokumentasi                                                           | 10%   |
