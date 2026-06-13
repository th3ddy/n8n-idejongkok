# 🤖 N8N Workflows — Kelas Agentic AI Automation

Koleksi workflow n8n yang dibuat selama mengikuti kelas **Agentic AI Automation** di Kelas Otomesyen.

## 🛠️ Setup

### Prasyarat
- VPS dengan Docker & Docker Compose
- N8N self-hosted (lihat setup di bawah)
- Telegram Bot Token

### Install N8N
```bash
mkdir ~/n8n && cd ~/n8n
```

Buat `docker-compose.yml`:
```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your_password
      - N8N_HOST=your-domain.com
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://your-domain.com/
      - N8N_SECURE_COOKIE=false
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
```

```bash
docker compose up -d
```

---

## 📋 Daftar Workflow

### 1. 🌤️ Cuaca Jakarta → Telegram

**File:** `workflows/weather-telegram.json`

**Deskripsi:**  
Workflow yang mengambil data cuaca Jakarta dari API publik (Open-Meteo) dan mengirimkan notifikasi suhu ke Telegram.

**Flow:**
[Manual Trigger] → [HTTP Request: Open-Meteo API] → [Set: Format Pesan] → [Telegram: Kirim Pesan]
**Step by Step Membuat Workflow:**

#### Step 1: Manual Trigger
- Di canvas n8n, klik **"+"**
- Search **"Manual Trigger"** → tambahkan

#### Step 2: HTTP Request — Ambil Data Cuaca
- Klik **"+"** → search **"HTTP Request"** → tambahkan
- **Method:** `GET`
- **URL:**
https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&current_weather=true

- Klik **Execute** → lihat output JSON

#### Step 3: Set — Format Pesan
- Klik **"+"** → search **"Set"** → tambahkan
- Klik **"Add field"**
  - **Name:** `message`
  - **Value:** `Suhu Jakarta sekarang: {{ $json.current_weather.temperature }}°C`

#### Step 4: Telegram — Kirim Pesan
- Klik **"+"** → search **"Telegram"** → tambahkan
- **Credential:** buat baru dengan Bot Token dari @BotFather
- **Operation:** `Send Message`
- **Chat ID:** ID Telegram kamu (dapat dari @userinfobot)
- **Text:** `{{ $json.message }}`

#### Step 5: Test
- Klik **"Execute Workflow"**
- Cek Telegram → harusnya masuk pesan suhu Jakarta ✅

**Output contoh di Telegram:**
Suhu Jakarta sekarang: 33.3°C

---

## 📚 Resource
- [N8N Documentation](https://docs.n8n.io/)
- [Open-Meteo API](https://open-meteo.com/)
- [Kelas Otomesyen](https://kelasotomesyen.com)

## 👤 Author
- GitHub: [@th3ddy](https://github.com/th3ddy)
