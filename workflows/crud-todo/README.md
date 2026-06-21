# 📋 Workflow: CRUD Todo List — JSONPlaceholder

**Kelas Otomesyen — Agentic AI Automation | Modul 3: HTTP Requests**

---

## Informasi Workflow

| Field | Detail |
|---|---|
| **Nama Workflow** | CRUD Todo List |
| **Modul** | Modul 3 — HTTP Requests: Fondasi Semua Node di N8N |
| **Tanggal Dibuat** | 20 Juni 2026 |
| **Platform** | n8n self-hosted |
| **Instance** | `https://wflow.shifttab.my.id` |
| **GitHub** | `github.com/th3ddy/n8n-idejongkok` |

---

## Deskripsi

Workflow ini mendemonstrasikan semua HTTP Methods (GET, POST, PUT, DELETE) menggunakan **JSONPlaceholder** — sebuah fake REST API gratis yang dirancang khusus untuk keperluan belajar dan testing.

Tujuan utama: memahami perbedaan fungsi setiap HTTP method dalam satu workflow yang berurutan.

---

## API yang Digunakan

| Field | Detail |
|---|---|
| **Nama API** | JSONPlaceholder |
| **URL Dokumentasi** | https://jsonplaceholder.typicode.com |
| **Base URL** | `https://jsonplaceholder.typicode.com` |
| **Authentication** | None (public API) |
| **Format Response** | JSON |

---

## Arsitektur Workflow

```
[Manual Trigger]
      ↓
[GET - Ambil Todos]        → GET /todos?_limit=3
      ↓
[POST - Buat Todo Baru]    → POST /todos
      ↓
[PUT - Replace Todo]       → PUT /todos/1
      ↓
[DELETE - Hapus Todo]      → DELETE /todos/1
      ↓
[Edit Fields - Rangkuman]  → Kumpulkan semua hasil
```

---

## Detail Node

### 1. Manual Trigger
| Parameter | Value |
|---|---|
| **Type** | Manual Trigger |
| **Fungsi** | Memulai workflow secara manual |

---

### 2. GET - Ambil Todos
| Parameter | Value |
|---|---|
| **Method** | `GET` |
| **URL** | `https://jsonplaceholder.typicode.com/todos?_limit=3` |
| **Authentication** | None |
| **Fungsi** | Mengambil 3 data todo pertama |

**Contoh Response:**
```json
[
  { "userId": 1, "id": 1, "title": "delectus aut autem", "completed": false },
  { "userId": 1, "id": 2, "title": "quis ut nam facilis", "completed": false },
  { "userId": 1, "id": 3, "title": "fugiat veniam minus", "completed": false }
]
```

---

### 3. POST - Buat Todo Baru
| Parameter | Value |
|---|---|
| **Method** | `POST` |
| **URL** | `https://jsonplaceholder.typicode.com/todos` |
| **Body Type** | JSON |
| **Execute Once** | ✅ ON (mencegah eksekusi 3x mengikuti jumlah item dari GET) |

**Request Body:**
```json
{
  "userId": 1,
  "title": "Belajar n8n Modul 3",
  "completed": false
}
```

**Contoh Response:**
```json
{
  "userId": 1,
  "id": 201,
  "title": "Belajar n8n Modul 3",
  "completed": false
}
```

> `id: 201` menandakan server "berhasil menyimpan" dan memberi ID baru.

---

### 4. PUT - Replace Todo
| Parameter | Value |
|---|---|
| **Method** | `PUT` |
| **URL** | `https://jsonplaceholder.typicode.com/todos/1` |
| **Body Type** | JSON |
| **Fungsi** | Mengganti **seluruh** data todo dengan ID 1 |

**Request Body:**
```json
{
  "userId": 1,
  "id": 1,
  "title": "Todo sudah diganti seluruhnya",
  "completed": true
}
```

> **PUT = replace semua field.** Semua field wajib disertakan, kalau tidak field tersebut akan hilang/null.

---

### 5. DELETE - Hapus Todo
| Parameter | Value |
|---|---|
| **Method** | `DELETE` |
| **URL** | `https://jsonplaceholder.typicode.com/todos/1` |
| **Body** | Tidak ada |
| **Fungsi** | Menghapus todo dengan ID 1 |

**Response:** `{}` (object kosong) dengan status code **200 OK**

---

### 6. Edit Fields - Rangkuman
| Parameter | Value |
|---|---|
| **Type** | Set Node |
| **Fungsi** | Mengumpulkan hasil dari semua node sebelumnya |

**Output:**
```json
{
  "get_result": "delectus aut autem",
  "post_result": "Belajar n8n Modul 3",
  "put_result": "Todo sudah diganti seluruhnya",
  "delete_result": "Berhasil dihapus (response kosong)"
}
```

---

## Perbandingan HTTP Methods

| Method | Fungsi | Analogi | Yang Dikirim |
|---|---|---|---|
| **GET** | Ambil data | Buka menu restoran | Tidak ada body |
| **POST** | Buat data baru | Pesan makanan baru | Semua field data baru |
| **PUT** | Replace seluruh data | Ganti seluruh pesanan | Semua field (wajib lengkap) |
| **PATCH** | Update sebagian | Tambah topping saja | Hanya field yang berubah |
| **DELETE** | Hapus data | Batalin pesanan | Tidak ada body |

### PUT vs PATCH — Perbedaan Penting

**Data awal:**
```json
{ "id": 1, "title": "Todo lama", "completed": false }
```

**Setelah PUT** dengan body `{ "completed": true }`:
```json
{ "id": 1, "title": null, "completed": true }
```
→ `title` **hilang** karena tidak disertakan!

**Setelah PATCH** dengan body `{ "completed": true }`:
```json
{ "id": 1, "title": "Todo lama", "completed": true }
```
→ `title` **tetap ada**, hanya `completed` yang berubah.

---

## Konsep yang Dipelajari

### Execute Once
Node POST menggunakan opsi **Execute Once** untuk mencegah eksekusi berulang. Tanpa ini, POST akan dijalankan 3 kali karena node sebelumnya (GET) menghasilkan 3 items.

### Referencing Data Antar Node
Di Set node, kita bisa mengambil data dari node manapun menggunakan syntax:
```
{{ $('Nama Node').first().json.field }}
```

### Status Code
- `200 OK` — request berhasil
- `201 Created` — data berhasil dibuat (setelah POST)
- `{}` response kosong setelah DELETE — bukan error, memang begitu standarnya

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| POST dieksekusi 3x | Node GET menghasilkan 3 items | Aktifkan **Execute Once** di Settings node POST |
| FALSE branch "Node was not executed" | Bukan error — workflow ambil jalur lain | Normal, test branch lain dengan ubah kondisi IF sementara |

---

## Referensi

| Resource | URL |
|---|---|
| JSONPlaceholder Docs | https://jsonplaceholder.typicode.com/guide |
| n8n HTTP Request Node | https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest |
| n8n Set Node | https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.set |

---

*Dibuat oleh: Theddy Wijaya | Kelas Agentic AI Automation — Kelas Otomesyen*
