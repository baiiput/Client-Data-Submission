# 📋 Petunjuk Update Google Apps Script

## ✅ Yang Sudah Difix

Error **"Cannot read properties of undefined (reading 'contents')"** sudah diperbaiki!

### Root Cause:
- add-v2.html awalnya kirim **FormData** (multipart/form-data)
- Google Apps Script **tidak populate** `e.postData.contents` untuk multipart
- Error: `e.postData` jadi `undefined` → crash!

### Solusi:
- ✅ add-v2.html sekarang kirim **JSON** dengan `Content-Type: application/json`
- ✅ Google Apps Script bisa handle JSON dengan baik
- ✅ `e.postData.contents` akan terisi dengan data JSON

---

## 🚀 Cara Update (SIMPLE!)

### Langkah 1: Update Google Apps Script

1. **Buka Google Sheets** Anda yang untuk data client
2. **Extensions** → **Apps Script**
3. **Hapus SEMUA kode** yang ada di editor
4. **Copy SEMUA isi file `appscript.txt`** (Ctrl+A → Ctrl+C)
5. **Paste** ke Apps Script editor (Ctrl+V)
6. **Save** (Ctrl+S atau klik ikon Save)

### Langkah 2: Update/Deploy Web App

#### Jika Sudah Pernah Deploy (Update Version):
1. Klik **Deploy** → **Manage deployments**
2. Klik **✏️ Edit** (icon pensil) di deployment yang aktif
3. Pilih **New version** di dropdown
4. Klik **Deploy**
5. Selesai! URL tetap sama, tidak perlu update config.js

#### Jika Belum Pernah Deploy (Deploy Baru):
1. Klik **Deploy** → **New deployment**
2. Klik icon ⚙️ di samping "Select type"
3. Pilih **Web app**
4. Setting:
   - **Description**: "Client Data API"
   - **Execute as**: **Me**
   - **Who has access**: **Anyone**
5. Klik **Deploy**
6. Klik **Authorize access**
7. Pilih akun Google Anda
8. Klik **Advanced** → **Go to ... (unsafe)** → **Allow**
9. **COPY Web app URL** yang muncul
10. Paste URL tersebut ke **config.js** Anda

---

## ✅ Test

### 1. Buka add-v2.html di browser

### 2. Buka Console (F12)

### 3. Isi form lengkap dan Submit

### 4. Check Console Log - Harus muncul:

```
🚀 Starting POST request to: https://script.google.com/...
📦 Data yang akan dikirim: {
  "targetSheet": "Client Aktif",
  "nama": "...",
  "accNo": "...",
  ...
}
📊 Data size: XXX bytes
📤 Sending POST request with JSON payload...
✅ Response received: 200 OK
📄 Content-Type: application/json
✅ Parsed result: {
  "status": "success",
  "targetSheet": "Client Aktif",
  "rowNumber": 123
}
```

### 5. Check Google Sheet

Data harus masuk dengan **LENGKAP**, tidak ada field yang kosong!

---

## 🔍 Check Apps Script Log (Optional)

1. Buka Apps Script Editor
2. Klik **Executions** di menu kiri (icon jam)
3. Klik execution yang terakhir
4. Lihat log, harus muncul:

```
=== DOPOST REQUEST ===
POST data: {"action":"submitClientData","clientData":{...}}
Content type: application/json
✅ Parsed as JSON
Action: submitClientData
Processing client data submission via POST...
=== SUBMIT CLIENT DATA START (UPDATED MAPPING) ===
Client data received: {...}
Target sheet: Client Aktif, New row: 123
✅ Data inserted to Client Aktif at row 123 with updated column mapping
```

---

## ❌ Troubleshooting

### Error: "No POST data received"
- Pastikan deploy dengan **New version**
- Hard refresh browser: **Ctrl+Shift+R** atau **Cmd+Shift+R**
- Clear browser cache

### Error: "Invalid JSON"
- Check browser console untuk lihat data yang dikirim
- Pastikan add-v2.html sudah di-save dan di-refresh

### Data masih tidak masuk
- Check **URL di config.js** sudah benar atau belum
- URL harus diawali: `https://script.google.com/macros/s/`
- Coba copy-paste URL lagi dari deployment

### Error 403 atau 401
- Pastikan deploy dengan **"Who has access: Anyone"**
- Jangan pilih "Only myself"

---

## 🎯 Apa yang Berubah

### File: `add-v2.html`
**Sebelum:**
```javascript
// Kirim FormData (multipart/form-data)
const formData = new FormData();
formData.append('action', 'submitClientData');
formData.append('data', JSON.stringify(data));

fetch(url, { method: 'POST', body: formData })
```

**Sekarang:**
```javascript
// Kirim JSON langsung (application/json)
const payload = {
    action: 'submitClientData',
    clientData: data
};

fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
})
```

### File: `appscript.txt`
**Yang Ditambah:**
```javascript
// Check e.postData existence
if (!e.postData || !e.postData.contents) {
    return error response
}

// Parse JSON
postData = JSON.parse(e.postData.contents);
```

---

## ✅ Checklist

Sebelum test, pastikan:
- [ ] File `appscript.txt` sudah di-copy ke Apps Script editor
- [ ] Sudah Save di Apps Script
- [ ] Sudah Deploy (New version atau New deployment)
- [ ] URL di `config.js` sudah benar
- [ ] Browser sudah di-refresh (Ctrl+Shift+R)
- [ ] Console dibuka (F12) untuk monitoring

---

## 💡 Tips

1. **Selalu cek Console browser** saat submit
2. **Selalu cek Executions** di Apps Script untuk debug
3. **Jangan lupa Deploy dengan New version** setiap kali ubah code
4. **Test dengan data minimal** dulu (isi field required aja)
5. **Kalau sukses**, baru isi data lengkap

---

## 📞 Masih Error?

Kirim screenshot dari:
1. Browser Console (F12 → Console tab)
2. Apps Script Executions log
3. Network tab (F12 → Network → klik request → Headers & Response)

Saya akan bantu troubleshoot! 🚀
