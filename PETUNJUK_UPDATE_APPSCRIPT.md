# 📋 Petunjuk Update Google Apps Script

## ✅ Yang Sudah Direvisi di `appscript.txt`

File **appscript.txt** Anda sudah saya revisi agar bisa menerima data dari **add-v2.html** yang baru.

### Perubahan yang Dilakukan:

1. **Function `parseFormData()` (BARU)** - Line 75-89
   - Menambahkan helper function untuk parse FormData format
   - Dibutuhkan untuk menerima data dari add-v2.html yang menggunakan POST FormData

2. **Function `doPost()` (DIREVISI)** - Line 1581-1622
   - Sekarang bisa handle **2 format**:
     - **Format lama**: JSON dengan `clientData`
     - **Format baru**: FormData dengan `data` (dari add-v2.html)
   - Auto-detect format dan parse sesuai

## 🚀 Cara Update ke Google Apps Script

### Langkah Simple:

1. **Buka Google Sheets** → **Extensions** → **Apps Script**

2. **Hapus semua kode yang ada** di editor

3. **Copy SEMUA isi file `appscript.txt`** (Ctrl+A → Ctrl+C)

4. **Paste ke Apps Script editor** (Ctrl+V)

5. **Save** (Ctrl+S)

6. **Deploy**:
   - Klik **Deploy** → **Manage deployments**
   - Klik **✏️ Edit** (icon pensil) di deployment yang aktif
   - Pilih **New version**
   - Klik **Deploy**
   - Copy URL (kalau berubah, update di config.js)

   ATAU jika belum pernah deploy:
   - Klik **Deploy** → **New deployment**
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Klik **Deploy**
   - **Authorize** dan **Allow**
   - Copy **Web app URL**
   - Paste ke **config.js**

## ✅ Test

1. Buka **add-v2.html** di browser
2. Isi form lengkap
3. Buka **Console** (F12)
4. Submit
5. Check console log dan Google Sheet

### Log yang Benar:

```
🚀 Starting POST request to: https://...
📦 Data yang akan dikirim: {...}
📊 Data size: XXX bytes
📤 Sending POST request...
✅ Response received: 200 OK
✅ Parsed result: {status: "success", targetSheet: "...", rowNumber: ...}
```

### Di Google Apps Script Log:

```
=== DOPOST REQUEST ===
POST data: action=submitClientData&data=%7B...
Content type: application/x-www-form-urlencoded
⚠️ Not JSON, trying FormData format...
✅ Parsed as FormData (new format from add-v2.html)
POST Action: submitClientData
Processing client data submission via POST...
=== SUBMIT CLIENT DATA START (UPDATED MAPPING) ===
✅ Data inserted to Client Aktif at row XX with updated column mapping
```

## 🎯 Kenapa Perlu Update?

**Masalah sebelumnya:**
- ❌ add-v2.html mengirim FormData (format baru)
- ❌ appscript.txt hanya bisa terima JSON (format lama)
- ❌ Data tidak cocok → error

**Solusi sekarang:**
- ✅ appscript.txt bisa terima FormData DAN JSON
- ✅ Backward compatible (format lama tetap jalan)
- ✅ Support add-v2.html yang baru
- ✅ Data lengkap, tidak ada yang hilang

## ❓ FAQ

**Q: Apakah semua form lain tetap jalan?**
A: Ya! Revisi ini backward compatible. Form lama yang kirim JSON tetap jalan seperti biasa.

**Q: Apakah perlu deploy ulang?**
A: Ya, karena kode berubah. Tapi bisa update deployment yang ada (tidak perlu buat baru).

**Q: URL berubah tidak?**
A: Kalau update deployment yang sama, URL tetap sama. Kalau buat deployment baru, URL akan berubah.

**Q: Data lama hilang?**
A: TIDAK. Data di Google Sheet tetap aman. Ini hanya update kode server.

## 🐛 Troubleshooting

**Error: "Invalid request format"**
- Pastikan copy-paste appscript.txt dengan lengkap
- Pastikan ada function `parseFormData()`

**Data tetap tidak masuk**
- Check Google Apps Script execution log
- Pastikan deployment sudah update ke version baru
- Test dengan console log terbuka (F12)
