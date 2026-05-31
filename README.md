# NeuroCare AI — Sistem Prediksi Risiko Alzheimer

Aplikasi web untuk skrining risiko penyakit Alzheimer menggunakan model **XGBoost** (22 fitur klinis). Sistem terdiri dari:

| Komponen | Teknologi | Port default |
|----------|-----------|--------------|
| Antarmuka web | Node.js + Express + EJS | `3000` |
| API prediksi ML | Python + Flask | `5000` |
| Penyimpanan riwayat | MySQL (XAMPP) | `3306` |

```
Browser → Express (3000) → Flask API (5000) → model_alzheimer_final (2).pkl
                ↓
            MySQL (neurocare_ai)
```

---

## Persyaratan

Pastikan sudah terpasang di komputer Anda:

- **Node.js** v14 atau lebih baru — [https://nodejs.org](https://nodejs.org)
- **Python** 3.8 atau lebih baru — [https://python.org](https://python.org)
- **XAMPP** (MySQL) — [https://www.apachefriends.org](https://www.apachefriends.org)  
  *(Wajib jika ingin fitur riwayat prediksi & data pasien)*
- File model: `model_alzheimer_final (2).pkl` (harus ada di folder root proyek)

Cek instalasi:

```cmd
node -v
npm -v
python --version
pip --version
```

---

## 1. Instalasi (sekali saja)

Buka **Command Prompt** atau **PowerShell**, masuk ke folder proyek:

```cmd
cd "C:\path\ke\folder\APM"
```

### 1.1 Dependensi Node.js

```cmd
npm install
```

### 1.2 Dependensi Python

```cmd
pip install -r requirements.txt
```

### 1.3 Database MySQL (opsional, untuk riwayat prediksi)

1. Buka **XAMPP Control Panel** → klik **Start** pada **MySQL** (dan **Apache** jika ingin phpMyAdmin).
2. Jalankan setup database:

```cmd
npm run db:setup
```

Atau double-click file `database\setup_db.bat`.

Script ini akan:
- Membuat database `neurocare_ai` dan tabel
- Mengimpor data awal RS Hermina dari CSV

> **Catatan:** Setup memakai user MySQL `root` tanpa password (default XAMPP). Jika MySQL Anda memakai password, buat file `.env` (lihat bagian Konfigurasi).

---

## 2. Konfigurasi (opsional)

Buat file `.env` di folder root jika port atau database berbeda dari default:

```env
# Server web Node.js
PORT=3000
PYTHON_API_URL=http://localhost:5000

# MySQL (XAMPP)
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_NAME=neurocare_ai
```

---

## 3. Cara menjalankan sistem

**Urutan penting:** MySQL (jika dipakai) → **Python API** → **Server web**

### Opsi A — Cepat (Windows, disarankan)

Double-click **`start.bat`** di folder proyek.

Atau dari terminal:

```cmd
start.bat
```

Script akan:
1. Membuka jendela terminal untuk Flask API (port 5000)
2. Menunggu 3 detik
3. Membuka jendela terminal untuk Express (port 3000)
4. Membuka browser ke `http://localhost:3000`

Untuk menghentikan: tutup kedua jendela terminal (Python & Node.js).

---

### Opsi B — Manual (dua terminal)

**Terminal 1 — API Python (jalankan dulu, biarkan terbuka):**

```cmd
cd "C:\path\ke\folder\APM"
python predict_api.py
```

Output yang diharapkan:

```
✓ Model berhasil dimuat!
🚀 Starting Flask API on port 5000...
```

**Terminal 2 — Server web:**

```cmd
cd "C:\path\ke\folder\APM"
npm start
```

Output yang diharapkan:

```
Server is running on port 3000
MySQL connected: neurocare_ai
```

*(Baris MySQL bisa berupa peringatan jika database belum di-setup — prediksi tetap bisa jalan, riwayat tidak tersimpan.)*

---

### Opsi C — Mode pengembangan

Terminal 1:

```cmd
python predict_api.py
```

Terminal 2 (auto-restart saat file diubah):

```cmd
npm run dev
```

Jika mengubah CSS Tailwind:

```cmd
npm run watch:css
```

---

## 4. Mengakses aplikasi

| Halaman | URL | Fungsi |
|---------|-----|--------|
| Beranda | http://localhost:3000 | Landing page |
| Prediksi | http://localhost:3000/predict | Form skrining (wizard) |
| Riwayat | http://localhost:3000/database | Daftar prediksi tersimpan |
| Cek API ML | http://localhost:5000/health | Status model Python |
| phpMyAdmin | http://localhost/phpmyadmin | Kelola database (XAMPP) |

### Alur penggunaan

1. Pastikan **Terminal Python** dan **Terminal Node.js** masih berjalan.
2. Buka http://localhost:3000 → menu **Prediksi** / **Screening**.
3. Isi form (22 variabel klinis) → **Submit**.
4. Lihat hasil risiko di halaman hasil.
5. Buka **Riwayat** (`/database`) untuk melihat prediksi yang tersimpan *(butuh MySQL)*.

---

## 5. Verifikasi sistem berjalan

| Cek | Cara | Hasil OK |
|-----|------|----------|
| Model ML | Buka http://localhost:5000/health | `"model_loaded": true` |
| Web | Buka http://localhost:3000 | Halaman beranda tampil |
| Database | Lihat log terminal Node.js | `MySQL connected: neurocare_ai` |

---

## 6. Pemecahan masalah

### Python API tidak jalan

| Gejala | Solusi |
|--------|--------|
| `ModuleNotFoundError: flask` | `pip install -r requirements.txt` |
| `Error loading model` / file tidak ditemukan | Pastikan `model_alzheimer_final (2).pkl` ada di folder root |
| Port 5000 sudah dipakai | Tutup proses lain di port 5000, atau set `PORT=5001` lalu sesuaikan `PYTHON_API_URL` di `.env` |

### Server Node.js error

| Gejala | Solusi |
|--------|--------|
| `Cannot find module 'axios'` | `npm install` |
| `ECONNREFUSED localhost:5000` | Jalankan `python predict_api.py` **sebelum** `npm start` |
| Port 3000 sudah dipakai | Ubah `PORT=3001` di `.env` |

### Database / riwayat tidak muncul

| Gejala | Solusi |
|--------|--------|
| `MySQL not connected` | Start MySQL di XAMPP |
| Database belum ada | `npm run db:setup` |
| Password MySQL tidak kosong | Isi `DB_PASSWORD` di file `.env` |

### Prediksi gagal di browser

1. Cek http://localhost:5000/health — harus `healthy`.
2. Pastikan kedua server (Python + Node) masih aktif.
3. Lihat pesan error di halaman atau di terminal.

---

## 7. Perintah npm berguna

```cmd
npm start          # Jalankan server web (production)
npm run dev        # Server web + auto-reload (nodemon)
npm run db:setup   # Setup database MySQL + seed data
npm run db:seed    # Import ulang data Hermina saja
npm run build:css  # Compile Tailwind CSS sekali
npm run watch:css  # Compile Tailwind otomatis saat edit
```

---

## 8. Struktur proyek

```
APM/
├── config/                 # Koneksi MySQL & repository
├── database/
│   ├── neurocare_ai.sql    # Skema database
│   ├── setup_db.bat        # Setup otomatis (XAMPP)
│   └── seed_hermina.js     # Import data CSV
├── public/                 # CSS, JS, gambar
├── views/                  # Template EJS (halaman web)
├── server.js               # Server Express
├── predict_api.py          # API Flask + model ML
├── model_alzheimer_final (2).pkl   # Model XGBoost (wajib)
├── requirements.txt        # Paket Python
├── package.json            # Paket Node.js
├── start.bat               # Jalankan semua server (Windows)
└── README.md
```

---

## 9. Model — 22 fitur input

Model membutuhkan 22 variabel klinis (urutan tetap): Age, Gender, BMI, Smoking, AlcoholConsumption, PhysicalActivity, DietQuality, SleepQuality, FamilyHistoryAlzheimers, CardiovascularDisease, Diabetes, Depression, HeadInjury, Hypertension, SystolicBP, DiastolicBP, CholesterolTotal, MMSE, FunctionalAssessment, MemoryComplaints, BehavioralProblems, ADL.

Detail API: `GET /features` dan `POST /predict` di http://localhost:5000

---

## Kredit

- **Institusi:** Universitas Andalas  
- **Fakultas:** Teknologi Informasi  
- **Tim:** Kelompok APM  
- **Tahun:** 2026  

Proyek akademik — bukan untuk diagnosis medis definitif.
