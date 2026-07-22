# template-ai-blogspot

# VEMS AI — Cloudflare Worker AI + Blogger Frontend

VEMS AI adalah platform asisten cerdas berbasis AI yang berjalan di Cloudflare Workers dengan integrasi Cloudflare KV, R2 Storage, Whisper STT, dan ElevenLabs TTS. Dilengkapi dengan tampilan frontend siap pakai untuk Blogger (Blogspot) sebagai tema interaktif.

Proyek ini memungkinkan Anda memiliki chatbot AI yang bisa diajak ngobrol, melakukan pencarian web (Tavily), mengunggah & menganalisis dokumen, serta berbicara dengan AI melalui input suara dan output suara — semuanya dalam satu paket gratis (dengan batasan kuota tertentu).

---

## Fitur Utama

- Chat AI berbasis Qwen (qwen3-30b-a3b-fp8 atau qwen2.5-coder-32b-instruct) untuk percakapan, coding, UI, dan penjelasan konsep.
- Autentikasi GitHub OAuth untuk menyimpan riwayat chat di Cloudflare KV dan mengakses penyimpanan R2.
- Penyimpanan file Cloudflare R2 (unggah, kelola, unduh gambar, dokumen, audio, video).
- Speech-to-Text (Whisper) — rekam suara dari browser, kirim ke endpoint /api/audio/stt, dapatkan transkripsi teks.
- Text-to-Speech (ElevenLabs) — jawaban AI otomatis diubah menjadi suara natural dan diputar di modal interaktif.
- Visualizer audio realtime saat merekam maupun saat AI berbicara.
- Multi-session chat — riwayat percakapan disimpan per pengguna dan bisa diakses kembali.
- Markdown & syntax highlighting untuk respons AI (tabel, daftar, blok kode).
- Integrasi pencarian web (Tavily) untuk informasi terkini.

---

## Arsitektur Sistem

Frontend (Blogger XML) <-> Backend (Cloudflare Worker)
Backend menangani:
- AI (Qwen)
- Auth GitHub OAuth
- KV Session & History
- R2 File Storage
- Whisper STT
- ElevenLabs TTS
- Tavily Search

---

## Prasyarat

Sebelum memulai, pastikan Anda memiliki:

- Akun Cloudflare dengan Workers diaktifkan.
- (Opsional) Akun GitHub OAuth App untuk login.
- (Opsional) API Key ElevenLabs untuk TTS (bisa pakai voice ID gratis).
- (Opsional) API Key Tavily untuk pencarian web.
- Akun Blogger jika ingin menggunakan frontend sebagai tema.

---

## Setup & Konfigurasi

### 1. Backend (Cloudflare Worker)

1. Clone atau buat file `vems_ai.ts` dari kode yang disediakan.
2. Buat Worker di dashboard Cloudflare dan tempelkan kode tersebut.
3. Siapkan Resources:
   - KV Namespace dengan nama `CHAT_SESSION_KV` (atau sesuai keinginan).
   - R2 Bucket dengan nama `MEDIA_R2` (atau sesuai keinginan).
4. Set Environment Variables di Worker:

   | Variable               | Deskripsi                                                    |
   |------------------------|--------------------------------------------------------------|
   | `AI`                   | Binding AI dari Cloudflare (auto).                           |
   | `CHAT_SESSION_KV`      | Binding KV namespace.                                        |
   | `MEDIA_R2`             | Binding R2 bucket.                                           |
   | `GITHUB_CLIENT_ID`     | Client ID dari GitHub OAuth App.                             |
   | `GITHUB_CLIENT_SECRET` | Client Secret dari GitHub OAuth App.                         |
   | `FRONTEND_URL`         | URL frontend Anda (misal https://blog-anda.blogspot.com).    |
   | `TAVILY_API_KEY`       | API Key dari Tavily (untuk pencarian web).                   |
   | `ELEVENLABS_API_KEY`   | API Key ElevenLabs (opsional, jika ingin TTS).               |
   | `ELEVENLABS_VOICE_ID`  | Voice ID ElevenLabs (default 21m00Tcm4TlvDq8ikWAM untuk Rachel, atau JBFqnCBsd6RMkjVDRZzb untuk gratis). |

5. Deploy Worker. Catat URL Worker (misal https://vems-ai.workers.dev).

### 2. Frontend (Blogger Template)

1. Buka Blogger -> Tema -> Edit HTML.
2. Salin seluruh kode `vems-ai.xml` yang telah disediakan dan tempelkan di dalam editor, ganti semua konten yang ada.
3. Di dalam file XML, cari dan ganti `const workerApiUrl = "https://vems-ai.vildaesa.workers.dev";` dengan URL Worker Anda.
4. Simpan tema.

Atau, jika Anda menggunakan mode pengembangan lokal, cukup buka file HTML di browser (dengan CORS diaktifkan).

---

## Penggunaan

1. Buka blog Anda yang sudah dipasangi template VEMS AI.
2. Login dengan GitHub (klik tombol "Gunakan Sinkronisasi GitHub" di sidebar) untuk menyimpan riwayat dan mengakses R2.
3. Mulai Chat:
   - Ketik pesan di kotak input.
   - Lampirkan file (dokumen, gambar, dll.) dengan tombol +.
   - Gunakan tombol Mic untuk merekam suara (transkripsi otomatis).
4. Jawaban AI akan muncul dengan format Markdown dan otomatis diputar suaranya (jika ElevenLabs dikonfigurasi).
5. Galeri R2: Buka menu "Galeri Penyimpanan R2" untuk melihat, mengunggah, atau menghapus file.
6. Riwayat Sesi: Semua percakapan tersimpan di sidebar dan bisa diakses kembali.

---

## API Endpoints (Backend)

| Endpoint                     | Method | Deskripsi                                                                 |
|------------------------------|--------|---------------------------------------------------------------------------|
| /api/auth/github             | GET    | Redirect ke halaman otorisasi GitHub.                                     |
| /api/auth/github/callback    | GET    | Callback OAuth, menukar code dengan token dan menyimpan sesi.             |
| /api/auth/me                 | GET    | Mengembalikan profil user yang sedang login.                              |
| /api/auth/logout             | POST   | Menghapus sesi token dari KV.                                             |
| /api/chat/history            | GET    | Mengambil riwayat chat user dari KV.                                      |
| /api/chat/history            | POST   | Menyimpan riwayat chat ke KV.                                             |
| /api/chat/history            | DELETE | Menghapus seluruh riwayat chat user.                                      |
| /api/media                   | GET    | Mendaftar semua file milik user di R2.                                    |
| /api/media/upload            | POST   | Mengunggah file ke R2 (multipart form-data).                              |
| /api/media/download          | GET    | Mengunduh file dari R2 (dengan parameter key).                            |
| /api/media                   | DELETE | Menghapus file dari R2 (dengan parameter key).                            |
| /api/audio/stt               | POST   | Menerima audio (WebM) dan mengembalikan teks transkripsi (Whisper).       |
| /api/audio/tts               | POST   | Menerima teks dan mengembalikan audio MP3 dari ElevenLabs (voiceId opsional). |
| / (root)                     | POST   | Endpoint utama chat: menerima message, fileContext, model, history.       |

---

## Kustomisasi & Pengembangan

- Ganti Model AI: Ubah nilai model di frontend atau backend sesuai model yang didukung Cloudflare AI.
- Tambahkan Voice ID: Kirim parameter voiceId di body /api/audio/tts untuk mengganti suara TTS.
- Ganti Tema: Sesuaikan warna dan gaya di file XML (Tailwind CSS sudah terintegrasi).
- Tambahkan Fitur: Anda bisa memperluas Worker dengan menambahkan endpoint baru atau mengintegrasikan API lain.

---

## Catatan Penting

- Kuota Gratis:
  - Cloudflare Workers: 100.000 request/hari untuk free plan.
  - ElevenLabs gratis: 10.000 token per bulan (voice ID JBFqnCBsd6RMkjVDRZzb adalah voice gratis).
  - Tavily free tier: batas pencarian terbatas.
- Keamanan: Pastikan GITHUB_CLIENT_SECRET dan ELEVENLABS_API_KEY tidak terekspos di frontend.
- CORS: Worker sudah mengatur header CORS secara otomatis.
- Browser Support: Gunakan browser modern (Chrome, Firefox, Edge) untuk dukungan Web Audio API dan MediaRecorder.

---

## Kontribusi

Jika Anda ingin mengembangkan proyek ini lebih lanjut, silakan fork repository dan ajukan pull request. Untuk pertanyaan atau saran, buka issue di repository.

---

## Lisensi

Proyek ini menggunakan lisensi MIT – Anda bebas menggunakan, memodifikasi, dan mendistribusikan ulang dengan tetap mencantumkan atribusi.

---

Selamat mencoba dan semoga bermanfaat!
