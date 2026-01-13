# 🚀 Cara Update Google Apps Script

## ⚡ 3 Langkah Mudah

### 1️⃣ Buka Apps Script Editor
1. Buka Google Sheets Anda (yang untuk menyimpan data client)
2. Klik menu **Extensions** → **Apps Script**
3. Editor Apps Script akan terbuka di tab baru

### 2️⃣ Copy-Paste Kode
1. Buka file **`appscript.txt`** (ada di folder ini)
2. **Copy SEMUA isi file tersebut** (Ctrl+A → Ctrl+C)
3. Kembali ke Apps Script Editor
4. **Hapus semua kode yang ada di editor**
5. **Paste kode baru** (Ctrl+V)
6. **PENTING**: Edit bagian **KONFIGURASI** (baris 25-40):
   - Cek nama sheet: `Client Aktif`, `Client Non Aktif`, `Client Lepas`
   - Cek urutan kolom sesuai dengan Google Sheet Anda
7. **Save** (Ctrl+S atau klik ikon Save)

### 3️⃣ Deploy Web App
1. Di Apps Script Editor, klik **Deploy** → **New deployment**
2. Klik ikon ⚙️ (gear) di samping "Select type"
3. Pilih **Web app**
4. Isi setting:
   - **Description**: "Client Data Submission API" (atau apa saja)
   - **Execute as**: **Me** (pilih email Anda)
   - **Who has access**: **Anyone**
5. Klik **Deploy**
6. Akan muncul dialog "Authorize access" → klik **Authorize access**
7. Pilih akun Google Anda
8. Klik **Advanced** → **Go to [Your Project Name] (unsafe)** → **Allow**
9. **COPY "Web app URL"** yang muncul (contoh: `https://script.google.com/macros/s/AKfycbz.../exec`)

### 4️⃣ Update config.js
1. Buka file **config.js** di folder project Anda
2. Paste URL yang baru dicopy ke bagian:
   ```javascript
   CONFIG = {
       API_URL: "PASTE_URL_DISINI",
       CUSTOM_APIS: {
           client: "PASTE_URL_DISINI"
       }
   }
   ```
3. Save file config.js

## ✅ Selesai! Test Form

1. Buka **add-v2.html** di browser
2. Isi form lengkap (semua field WAJIB)
3. Buka **Console** (tekan F12 → tab Console)
4. Klik **Submit Data**
5. Lihat console, harus muncul:
   ```
   🚀 Starting POST request to: https://...
   📦 Data yang akan dikirim: {...}
   📊 Data size: XXX bytes
   📤 Sending POST request...
   ✅ Response received: 200 OK
   ✅ Parsed result: {status: "success", ...}
   ```
6. Cek Google Sheet → data harus masuk LENGKAP

## 🐛 Jika Ada Error

### Error: "SCRIPT_URL belum dikonfigurasi"
- Cek file **config.js** sudah benar atau belum
- URL harus diawali dengan `https://script.google.com/macros/s/`

### Error: "Failed to fetch"
- Pastikan deploy dengan **"Who has access: Anyone"**
- Coba deploy ulang (Deploy → Manage deployments → New deployment)

### Error: "No data received in POST request"
- Pastikan sudah copy-paste **appscript.txt** dengan benar
- Jangan ada typo atau kode yang kurang

### Data masih tidak lengkap
- Cek bagian **COLUMN_ORDER** di appscript.txt
- Urutan harus sesuai dengan kolom di Google Sheet
- Contoh: Jika kolom A = Timestamp, B = Target Sheet, C = Nama, dst

## 📞 Debugging

### Lihat Log di Apps Script
1. Buka Apps Script Editor
2. Klik **Executions** di menu kiri
3. Klik execution yang terakhir
4. Lihat log untuk cek error

### Lihat Log di Browser
1. Buka add-v2.html
2. Tekan **F12** → tab **Console**
3. Submit form
4. Lihat semua log yang muncul

## ❓ FAQ

**Q: Apakah kode lama akan hilang?**
A: Ya, tapi tidak masalah. Kode baru ini lebih baik dan fix masalah data tidak lengkap.

**Q: Apakah harus deploy baru?**
A: Ya, karena code berubah. URL-nya akan sama atau berubah, tergantung cara deploy.

**Q: Apakah data lama di sheet akan hilang?**
A: TIDAK. Data lama tetap aman. Kode ini hanya menambah data baru.

**Q: Berapa lama prosesnya?**
A: Sekitar 5-10 menit jika lancar.
