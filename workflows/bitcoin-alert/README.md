# 📊 Laporan Workflow: Bitcoin Price Alert
**Kelas Otomesyen — Agentic AI Automation | Modul 3: HTTP Requests**

---

## Informasi Workflow

| Field | Detail |
|---|---|
| **Nama Workflow** | Bitcoin Price Alert |
| **Modul** | Modul 3 — HTTP Requests: Fondasi Semua Node di N8N |
| **Tanggal Dibuat** | 20 Juni 2026 |
| **Platform** | n8n self-hosted |
| **GitHub** | `github.com/th3ddy/n8n-idejongkok` |

---

## Deskripsi

Workflow ini memantau harga Bitcoin (BTC/USDT) secara otomatis setiap 30 menit dan mengirim notifikasi ke Telegram berdasarkan kondisi harga:

- Jika harga BTC **lebih dari $60,000** → kirim pesan 🚀 BTC MOON!
- Jika harga BTC **kurang dari atau sama dengan $60,000** → kirim pesan 📉 BTC tur nih.

---

## Arsitektur Workflow

```
[Schedule Trigger: setiap 30 menit]
        ↓
[HTTP Request: GET harga BTC dari Binance API]
        ↓
[Code (JavaScript): konversi price string → number]
        ↓
[IF: price > 60000]
        ↓                        ↓
   TRUE branch               FALSE branch
[Telegram: 🚀 BTC MOON!]  [Telegram: 📉 BTC tur nih.]
```

---

## Detail Node

### 1. Schedule Trigger
| Parameter | Value |
|---|---|
| **Node Type** | `n8n-nodes-base.scheduleTrigger` |
| **Interval** | Every `30` minutes |
| **Fungsi** | Memicu workflow secara otomatis setiap 30 menit |

---

### 2. HTTP Request
| Parameter | Value |
|---|---|
| **Node Type** | `n8n-nodes-base.httpRequest` |
| **Method** | `GET` |
| **URL** | `https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT` |
| **Authentication** | None (public API) |
| **Fungsi** | Mengambil harga BTC/USDT real-time dari Binance |

**Contoh Response:**
```json
{
  "symbol": "BTCUSDT",
  "price": "105432.12"
}
```

> **Catatan:** API Coindesk yang disebutkan di modul sudah tidak aktif sejak 2024. Diganti dengan Binance Public API yang lebih reliable dan real-time.

---

### 3. Code (JavaScript)
| Parameter | Value |
|---|---|
| **Node Type** | `n8n-nodes-base.code` |
| **Language** | JavaScript |
| **Fungsi** | Konversi `price` dari String ke Number + format tampilan |

**Code:**
```javascript
const priceString = $input.first().json.price;
const priceNumber = Number(priceString);

return [{
  json: {
    symbol: $input.first().json.symbol,
    price: priceNumber,
    price_formatted: `$${priceNumber.toLocaleString('en-US', {
      minimumFractionDigits: 2,
      maximumFractionDigits: 2
    })}`
  }
}];
```

**Kenapa perlu node ini?**
Binance API mengembalikan `price` sebagai **String** (`"105432.12"`), bukan Number. IF node tidak bisa membandingkan string dengan angka secara akurat, sehingga perlu dikonversi terlebih dahulu dengan `Number()`.

**Output:**
```json
{
  "symbol": "BTCUSDT",
  "price": 105432.12,
  "price_formatted": "$105,432.12"
}
```

---

### 4. IF Node
| Parameter | Value |
|---|---|
| **Node Type** | `n8n-nodes-base.if` |
| **Condition** | `{{ $json.price }}` **greater than** `60000` |
| **Type Validation** | Strict (Number) |
| **Fungsi** | Branching berdasarkan kondisi harga BTC |

---

### 5. Telegram — TRUE Branch (BTC MOON)
| Parameter | Value |
|---|---|
| **Node Type** | `n8n-nodes-base.telegram` |
| **Chat ID** | `276441270` |
| **Trigger Condition** | Harga BTC > $60,000 |
| **Message** | `🚀 BTC MOON! Harga sekarang: {{ $json.price_formatted }} Update setiap 30 menit.` |

---

### 6. Telegram — FALSE Branch (BTC tur)
| Parameter | Value |
|---|---|
| **Node Type** | `n8n-nodes-base.telegram` |
| **Chat ID** | `276441270` |
| **Trigger Condition** | Harga BTC ≤ $60,000 |
| **Message** | `📉 BTC tur nih. Harga sekarang: {{ $json.price_formatted }} Update setiap 30 menit.` |

---

## Konsep yang Dipelajari

### HTTP Methods
Workflow ini menggunakan method **GET** — dipakai untuk membaca/mengambil data tanpa mengubah apapun di server. Cocok untuk API publik seperti price ticker.

### JSON Parsing
API Binance mengembalikan data JSON:
```json
{ "symbol": "BTCUSDT", "price": "105432.12" }
```
Untuk akses field `price` di node berikutnya, kita pakai expression n8n:
```
{{ $json.price }}
```

### Type Coercion Issue
Pelajaran penting dari workflow ini: **selalu perhatikan tipe data**. Field `price` dari Binance adalah String, bukan Number. Perbandingan `"105432" > 60000` bisa menghasilkan hasil yang tidak terduga jika tidak dikonversi terlebih dahulu.

### Conditional Branching
Node IF memungkinkan workflow berjalan di dua jalur berbeda berdasarkan kondisi. Ini adalah dasar dari logika automation — mirip `if/else` di programming.

### Credential Management
Telegram Bot Token disimpan sebagai **Credential** di n8n (tidak di-hardcode langsung di node). Ini best practice keamanan yang wajib diterapkan.

---

## Troubleshooting yang Ditemui

| Masalah | Penyebab | Solusi |
|---|---|---|
| API Coindesk tidak bisa diakses | Domain api.coindesk.com sudah mati sejak 2024 | Ganti ke Binance API |
| FALSE branch: "Node was not executed" | Bukan error — harga BTC memang > $60K sehingga workflow ambil jalur TRUE | Ubah threshold sementara ke `999999999` untuk test FALSE branch |

---

## Referensi

| Resource | URL |
|---|---|
| Binance API Docs | https://binance-docs.github.io/apidocs/spot/en/#symbol-price-ticker |
| n8n HTTP Request Node | https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/ |
| n8n IF Node | https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/ |
| n8n Code Node | https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/ |

---

*Dibuat oleh: TW | Kelas Agentic AI Automation — Kelas Otomesyen*
