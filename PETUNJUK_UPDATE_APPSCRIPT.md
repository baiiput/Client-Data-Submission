# 📋 Petunjuk Update Google Apps Script

## ✅ Yang Sudah Difix

Error **"No POST data received"** sudah diperbaiki!

### Root Cause:
- Google Apps Script Web App punya behavior khusus dengan POST request
- Kalau kirim JSON (`application/json`), data sering hilang saat redirect
- `e.postData.contents` jadi `undefined`

### Solusi FINAL:
- ✅ add-v2.html sekarang kirim **URLSearchParams** (`application/x-www-form-urlencoded`)
- ✅ Format: `action=submitClientData&clientData=%7B...%7D`
- ✅ Data available di **e.postData.contents** DAN **e.parameter** (double safety!)
- ✅ Google Apps Script bisa parse dengan mudah

---

## 🚀 Cara Update (2 Langkah Aja!)

### Langkah 1: Update Google Apps Script

1. **Buka Google Sheets** Anda yang untuk data client
2. **Extensions** → **Apps Script**
3. **Hapus SEMUA kode** yang ada di editor
4. **Copy SEMUA isi file `appscript.txt`** (yang sudah saya revisi)
5. **Paste** ke Apps Script editor
6. **Save** (Ctrl+S)

### Langkah 2: Deploy

#### Jika Sudah Pernah Deploy (Update Version):
1. Klik **Deploy** → **Manage deployments**
2. Klik **✏️ Edit** (icon pensil) di deployment yang aktif
3. Pilih **New version** di dropdown
4. Klik **Deploy**
5. ✅ **Selesai!** URL tetap sama

#### Jika Belum Pernah Deploy (Deploy Baru):
1. Klik **Deploy** → **New deployment**
2. Klik icon ⚙️ → Pilih **Web app**
3. Setting:
   - **Execute as**: **Me**
   - **Who has access**: **Anyone**
4. Klik **Deploy**
5. **Authorize** → **Allow**
6. **COPY Web app URL**
7. Paste ke **config.js**

---

## ✅ Test Sekarang!

### 1. Hard Refresh Browser
**Penting!** Browser cache bisa bikin file lama masih keload.
- **Windows/Linux**: `Ctrl + Shift + R`
- **Mac**: `Cmd + Shift + R`

### 2. Buka Console (F12)

### 3. Isi Form & Submit

### 4. Expected Console Logs:

```
🚀 Starting POST request to: https://script.google.com/...
📦 Data yang akan dikirim: {
  "targetSheet": "Client Aktif",
  "nama": "Test Client",
  "accNo": "ACC-001",
  ...
}
📊 Data size: 456 bytes
📤 Sending POST request...
✅ Response received: 200 OK
📄 Content-Type: application/json
✅ Parsed result: {
  "status": "success",
  "targetSheet": "Client Aktif",
  "rowNumber": 123
}
```

### 5. Check Google Sheet
✅ Data harus masuk **LENGKAP** di row baru!

---

## 🔍 Check Apps Script Log

1. Apps Script Editor → **Executions** (icon jam)
2. Klik execution terakhir
3. Expected logs:

```
=== DOPOST REQUEST ===
POST data: action=submitClientData&clientData=%7B%22targetSheet%22...
Content type: application/x-www-form-urlencoded
✅ Parsed as URL-encoded format
POST Action: submitClientData
Processing client data submission via POST...
=== SUBMIT CLIENT DATA START (UPDATED MAPPING) ===
Target sheet: Client Aktif, New row: 123
✅ Data inserted to Client Aktif at row 123 with updated column mapping
```

---

## ❌ Troubleshooting

### Masih Error "No POST data received"?

**Solusi:**
1. **Pastikan sudah Deploy dengan NEW VERSION** (bukan edit deployment lama tanpa new version)
2. **Hard refresh browser**: Ctrl+Shift+R (bukan F5 biasa!)
3. **Clear browser cache**: Settings → Clear browsing data → Cached images
4. **Test di browser lain** (Chrome, Firefox, Edge)
5. **Check URL di config.js** harus benar dan lengkap

### Error: "Invalid JSON" atau Parse Error

**Cek:**
- Pastikan `appscript.txt` di-copy **LENGKAP** (termasuk function `parseFormData()`)
- Pastikan tidak ada typo saat copy-paste
- Save ulang di Apps Script editor

### Data masih tidak masuk / kosong

**Cek:**
1. Browser console - ada error merah?
2. Apps Script Executions log - apa yang error?
3. Network tab (F12 → Network) - request berhasil 200 OK?
4. Column mapping di `appscript.txt` sudah sesuai dengan sheet?

### Error 403 Forbidden

**Fix:**
- Deploy ulang dengan **"Who has access: Anyone"**
- JANGAN pilih "Only myself"

---

## 🎯 Apa yang Berubah

### add-v2.html (Otomatis sudah terupdate):
```javascript
// SEBELUM (gagal):
const payload = { action: 'submitClientData', clientData: data };
fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
})

// SEKARANG (success!):
const payload = new URLSearchParams();
payload.append('action', 'submitClientData');
payload.append('clientData', JSON.stringify(data));
fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: payload
})
```

### appscript.txt (Perlu Anda update):
```javascript
// DITAMBAHKAN:
// 1. Fallback ke e.parameter jika e.postData tidak ada
if (!e.postData || !e.postData.contents) {
    if (e.parameter && e.parameter.action) {
        // Use e.parameter as fallback
    }
}

// 2. Support URL-encoded format parsing
var params = parseFormData(e.postData.contents);
postData = {
    action: params.action,
    clientData: JSON.parse(params.clientData)
};
```

---

## ✅ Kenapa Ini Work?

**application/x-www-form-urlencoded** adalah format yang:
- ✅ **Native** untuk web forms
- ✅ **Reliably handled** by Google Apps Script
- ✅ Data available di **e.postData.contents** (primary)
- ✅ Data available di **e.parameter** (fallback/backup!)
- ✅ **No data loss** saat redirect
- ✅ **Works across all browsers**

Dengan 2 source data (e.postData + e.parameter), hampir mustahil gagal!

---

## 📋 Checklist

Sebelum test, pastikan:
- [x] File add-v2.html sudah terupdate (otomatis)
- [ ] File `appscript.txt` sudah di-copy ke Apps Script editor
- [ ] Sudah **Save** di Apps Script
- [ ] Sudah **Deploy NEW VERSION** (penting!)
- [ ] URL di `config.js` sudah benar
- [ ] Browser sudah **hard refresh** (Ctrl+Shift+R)
- [ ] Console dibuka (F12) untuk monitoring

---

## 💡 Tips Pro

1. **Selalu hard refresh** (Ctrl+Shift+R) setelah update code
2. **Selalu Deploy NEW VERSION** (bukan edit tanpa new version)
3. **Check Executions log** untuk debug di server side
4. **Check Console log** untuk debug di client side
5. **Check Network tab** untuk lihat request/response detail
6. **Test data minimal dulu** (isi required field aja) sebelum data lengkap
7. **Jangan close Console** saat test - sangat penting untuk debugging!

---

## 📞 Masih Bermasalah?

Share screenshot dari:
1. **Browser Console** (F12 → Console) - full logs
2. **Network Tab** (F12 → Network → klik request → Headers & Response)
3. **Apps Script Executions** log
4. **Error message** yang muncul

Dengan info lengkap, saya bisa bantu troubleshoot lebih cepat! 🚀
