# 🔄 Upgrade Guide: Fix Network Error & Incomplete Data

## ⚠️ Masalah yang Diperbaiki

1. **URL terlalu panjang**: Method JSONP GET sebelumnya mengirim semua data dalam URL, yang memiliki batas ~2048 karakter. Ketika data besar (banyak login, alamat panjang), data terpotong.
2. **Callback JSONP tidak dipanggil**: Response format yang salah menyebabkan error message meski data terkirim.
3. **Data tidak lengkap**: Karena URL terpotong, sebagian field tidak terkirim ke Google Sheets.

## ✅ Solusi

**add-v2.html** telah diupdate untuk menggunakan **POST request dengan fetch()** yang:
- ✅ Mengirim data di request body (tidak ada batas panjang URL)
- ✅ Response handling yang lebih baik
- ✅ Error messages yang lebih jelas
- ✅ Timeout handling yang proper

## 📝 Yang Harus Diupdate di Google Apps Script

Google Apps Script Anda perlu diupdate untuk menerima **POST request** dan mengembalikan **JSON response** yang proper.

### Option 1: Update Existing Script (Recommended)

Jika Anda sudah punya Google Apps Script yang menggunakan `doGet(e)`, tambahkan function `doPost(e)` berikut:

```javascript
/**
 * Handle POST request dari add-v2.html (NEW METHOD)
 */
function doPost(e) {
  try {
    console.log('📥 POST request received');
    console.log('📦 Raw postData:', e.postData);

    // Parse POST data
    let data;
    let action;

    if (e.postData && e.postData.contents) {
      // Parse FormData
      const params = parseFormData(e.postData.contents);
      action = params.action;
      data = JSON.parse(params.data);
    } else if (e.parameter && e.parameter.data) {
      // Fallback: data di parameter
      action = e.parameter.action;
      data = JSON.parse(e.parameter.data);
    } else {
      throw new Error('No data received in POST request');
    }

    console.log('🎯 Action:', action);
    console.log('📊 Parsed data:', JSON.stringify(data, null, 2));

    // Route to appropriate handler
    if (action === 'submitClientData') {
      return handleClientDataSubmission(data);
    } else {
      throw new Error('Unknown action: ' + action);
    }

  } catch (error) {
    console.error('❌ Error in doPost:', error);
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString(),
      stack: error.stack
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

/**
 * Parse FormData from POST request
 */
function parseFormData(contents) {
  const params = {};
  const pairs = contents.split('&');

  for (let i = 0; i < pairs.length; i++) {
    const pair = pairs[i].split('=');
    const key = decodeURIComponent(pair[0]);
    const value = decodeURIComponent(pair[1] || '');
    params[key] = value;
  }

  return params;
}

/**
 * Handle client data submission
 */
function handleClientDataSubmission(data) {
  try {
    console.log('📝 Processing client data submission...');

    // Validation
    if (!data.nama || !data.accNo || !data.emailClient) {
      throw new Error('Required fields missing: nama, accNo, or emailClient');
    }

    // Get target sheet
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const targetSheet = ss.getSheetByName(data.targetSheet || 'Client Aktif');

    if (!targetSheet) {
      throw new Error('Target sheet not found: ' + (data.targetSheet || 'Client Aktif'));
    }

    // Get current timestamp
    const timestamp = new Timestamp();

    // Prepare row data - ADJUST THIS ACCORDING TO YOUR SHEET COLUMNS!
    const rowData = [
      timestamp,                    // Column A: Timestamp
      data.targetSheet || '',       // Column B: Target Sheet
      data.nama || '',              // Column C: Nama Client
      data.accNo || '',             // Column D: Account Number
      data.emailClient || '',       // Column E: Email Client
      data.nomorCS || '',           // Column F: Nomor CS
      data.loginGmail || '',        // Column G: Login Gmail
      data.loginStarlink || '',     // Column H: Login Starlink
      data.loginAlternatif || '',   // Column I: Login Alternatif
      data.kitNumber || '',         // Column J: KIT Number
      data.serialNumber || '',      // Column K: Serial Number
      data.alamat || '',            // Column L: Alamat
      data.tanggalJatuhTempo || '', // Column M: Tanggal Jatuh Tempo
      data.payment || '',           // Column N: Payment Method
      data.status || '',            // Column O: Status
      data.paket || '',             // Column P: Paket
      data.last4Digit || '',        // Column Q: Last 4 Digit
      data.idTransaksi || '',       // Column R: ID Transaksi
      data.kode || '',              // Column S: Kode
      data.noRegister || ''         // Column T: No Register
    ];

    // Append to sheet
    targetSheet.appendRow(rowData);
    const lastRow = targetSheet.getLastRow();

    console.log('✅ Data saved to row:', lastRow);

    // Return success response
    return ContentService.createTextOutput(JSON.stringify({
      status: 'success',
      message: 'Data berhasil disimpan',
      targetSheet: data.targetSheet || 'Client Aktif',
      rowNumber: lastRow,
      timestamp: timestamp
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    console.error('❌ Error saving data:', error);
    return ContentService.createTextOutput(JSON.stringify({
      status: 'error',
      message: error.toString(),
      stack: error.stack
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

/**
 * Get formatted timestamp
 */
function getTimestamp() {
  const now = new Date();
  const options = {
    timeZone: 'Asia/Jakarta',
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  };

  return now.toLocaleString('id-ID', options);
}
```

### Option 2: Keep doGet for Backward Compatibility

Jika Anda ingin tetap support method lama (JSONP GET) sambil menambah support POST:

```javascript
/**
 * Handle GET request (OLD METHOD - for backward compatibility)
 */
function doGet(e) {
  try {
    const action = e.parameter.action;
    const callback = e.parameter.callback;

    if (action === 'submitClientData') {
      const data = JSON.parse(decodeURIComponent(e.parameter.data));
      const result = handleClientDataSubmission(data);

      // Return JSONP response for GET requests
      if (callback) {
        return ContentService.createTextOutput(
          callback + '(' + result.getContent() + ')'
        ).setMimeType(ContentService.MimeType.JAVASCRIPT);
      } else {
        return result;
      }
    }

    throw new Error('Unknown action: ' + action);

  } catch (error) {
    console.error('❌ Error in doGet:', error);
    const result = JSON.stringify({
      status: 'error',
      message: error.toString()
    });

    if (e.parameter.callback) {
      return ContentService.createTextOutput(
        e.parameter.callback + '(' + result + ')'
      ).setMimeType(ContentService.MimeType.JAVASCRIPT);
    } else {
      return ContentService.createTextOutput(result)
        .setMimeType(ContentService.MimeType.JSON);
    }
  }
}

/**
 * Handle POST request (NEW METHOD - recommended)
 */
function doPost(e) {
  // ... (sama seperti Option 1 di atas)
}
```

## 🚀 Deployment Steps

1. Buka Google Apps Script editor
2. Copy-paste code di atas
3. **PENTING**: Adjust array `rowData` di function `handleClientDataSubmission()` sesuai dengan urutan kolom di Google Sheet Anda
4. Save script (Ctrl+S atau Cmd+S)
5. Deploy:
   - Klik **Deploy** → **New deployment**
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone** (atau sesuai kebutuhan)
   - Klik **Deploy**
6. Copy **Web app URL** yang baru
7. Update `config.js` dengan URL baru tersebut

## 🧪 Testing

Setelah deploy, test dengan:

1. Buka add-v2.html di browser
2. Isi semua required fields
3. Buka browser console (F12)
4. Submit form
5. Check console logs:
   - Harus muncul: `🚀 Starting POST request to: ...`
   - Harus muncul: `✅ Response received: 200 OK`
   - Harus muncul: `✅ Parsed result: {status: "success", ...}`
6. Check Google Sheet - data harus lengkap dan tidak ada yang missing

## 📊 Expected Console Logs (Success)

```
🚀 Starting POST request to: https://script.google.com/...
📦 Data yang akan dikirim: {...}
📊 Data size: 1234 bytes
📤 Sending POST request...
✅ Response received: 200 OK
📄 Content-Type: application/json
✅ Parsed result: {status: "success", targetSheet: "Client Aktif", rowNumber: 123}
```

## ❌ Troubleshooting

### Error: "Failed to fetch"
- Check internet connection
- Check SCRIPT_URL in config.js
- Pastikan Web App sudah di-deploy dengan "Who has access: Anyone"

### Error: "No data received in POST request"
- Check Google Apps Script code
- Pastikan ada function `doPost(e)`
- Check console.log di Apps Script execution log

### Data masih tidak lengkap
- Check urutan kolom di array `rowData`
- Check console logs untuk melihat data yang dikirim
- Pastikan semua field ada di `rowData` array

## 🔍 Debugging

1. **Browser Console**: F12 → Console tab
2. **Google Apps Script Logs**:
   - Apps Script Editor → Executions
   - Atau: View → Logs (setelah test execution)

## 📞 Support

Jika masih ada masalah:
1. Check browser console logs
2. Check Google Apps Script execution logs
3. Verify Web App deployment settings
4. Test dengan data minimal dulu (isi field required aja)
