# 🎬 KissKH Universal Extractor API

A self-hosted FastAPI + Playwright server that extracts drama info, stream links, and subtitles from KissKH — with Cloudflare bypass, ad blocking, and proxy support.

---

## 🚀 Deploy to Render (Free Tier)

1. Fork or clone this repo
2. Go to [render.com](https://render.com) → **New Web Service**
3. Connect your GitHub repo
4. Set the following:
```bash
|     Setting       |   Value |
|-------------------|---------|
| **Environment**   | `Docker`|
| **Instance Type** | `Free`  |
| **Port** | `8000` |
```
5. Add environment variables:
```bash
|      Key   |  Value |
|------------|--------|
| `HEADLESS` | `true` |
| `PORT`     | `8000` |
```
6. Click **Deploy**

> ⚡ The built-in keep-alive pinger runs every **5 minutes** to prevent Render's free tier from sleeping.

---

## 🐳 Docker (Self-hosted)

**Dockerfile:**
```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.44.0-jammy

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
RUN playwright install chromium

COPY . .

EXPOSE 8000
CMD ["python", "main.py"]
```

**Build & run:**
```bash
docker build -t kisskh-api .
docker run -p 8000:8000 -e HEADLESS=true kisskh-api
```

---

## 💻 Run Locally

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/kisskh-api.git
   cd kisskh-api
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Install Playwright browser:**
   ```bash
   playwright install chromium
   ```

4. **Start the server:**
   ```bash
   python main.py
   ```

**requirements.txt:**
```
fastapi
uvicorn
playwright
httpx
```

---

## ⚙️ Environment Variables
```bash
| Variable | Default |     Description        |
|------|---|---------|------------------------|
| HEADLESS |   true  | Run browser headlessly |
| PORT     |   8000  |     Server port        |
```
---

## 📂 Project Structure

```
kisskh-api/
├── main.py                  # FastAPI app + KissKHExtractor
├── requirements.txt         # Python dependencies
├── Dockerfile               # Container definition
├── kisskh_browser_data/     # Persistent browser session (auto-created)
└── README.md
```

---

## 📡 API Endpoints

Base URL: `http://localhost:8000`

---

### `GET /`
Health check — confirms server is running and Cloudflare is bypassed.

**Response:**
```json
{
  "message": "KissKH Active. Engine operating cleanly via Main IO Pool!"
}
```

---

### `GET /search?q={query}`
Search for dramas by title.



**Example:** `/search?q=queen+of+tears`

**Response:**
```json
[
  {
    "id": 4401,
    "title": "Queen of Tears",
    "thumbnail": "https://kisskh.do/img/drama/queen-of-tears.jpg",
    "episodesCount": 16,
    "status": "Completed",
    "type": "Korean Drama",
    "sub": "Sub"
  }
]
```

---

### `GET /info/{drama_id}`
Get full drama details and episode list.


**Example:** `/info/4401`

**Response:**
```json
{
  "id": 4401,
  "title": "Queen of Tears",
  "description": "A top conglomerate heiress and her husband...",
  "thumbnail": "https://kisskh.do/img/drama/queen-of-tears.jpg",
  "episodesCount": 16,
  "status": "Completed",
  "type": "Korean Drama",
  "episodes": [
    { "id": 98201, "number": 1, "sub": 1 },
    { "id": 98202, "number": 2, "sub": 1 },
    { "id": 98203, "number": 3, "sub": 1 }
  ]
}
```

---

### `GET /resolve/{episode_id}`
Resolve stream URL and subtitles for a specific episode.


**Example:** `/resolve/98201`

> ⚠️ Stream URLs are time-limited — always resolve fresh before playback.

**Response:**
```json
{
  "episode_id": "98201",
  "stream": {
    "Video": "https://cdn.kisskh.do/hls/ep98201/index.m3u8",
    "BackupVideo": "https://backup.kisskh.do/hls/ep98201/index.m3u8",
    "ThirdVideo": null
  },
  "subtitles": [
    {
      "file": "https://cdn.kisskh.do/sub/98201.en.vtt",
      "label": "English",
      "kind": "captions"
    },
    {
      "file": "https://cdn.kisskh.do/sub/98201.zh.vtt",
      "label": "Chinese",
      "kind": "captions"
    }
  ]
}
```

**Error response (after 3 failed attempts):**
```json
{
  "error": "Failed after 3 attempts",
  "status": 403
}
```

---

### `GET /home/latest`
Recently updated dramas.


**Example:** `/home/latest?page=1&page_size=20`

**Response:**
```json
{
  "data": [
    {
      "id": 4401,
      "title": "Queen of Tears",
      "thumbnail": "https://kisskh.do/img/...",
      "status": "Ongoing",
      "episodesCount": 16,
      "type": "Korean Drama"
    }
  ],
  "total": 200,
  "page": 1,
  "pageSize": 20
}
```

---

### `GET /home/popular`
Most popular dramas.


**Example:** `/home/popular?page=1`

---

### `GET /home/new`
Newly added dramas.


**Example:** `/home/new?page=1`

---

### `GET /home/ongoing`
Currently airing dramas.


**Example:** `/home/ongoing?country=1&page=1`

---

### `GET /home/completed`
Fully aired dramas.


**Example:** `/home/completed?country=2&page=2`

---

### `GET /browse`
Full browse with all filters combined.

**Example:** `/browse?status=1&country=1&order=2&page_size=10`

---

## 📝 Notes

- First startup takes **30–60 seconds** to complete Cloudflare bypass
- Stream URLs expire — always call `/resolve` fresh per playback session
- Subtitle extraction uses 3 fallback methods: network sniff → page fetch → embedded JSON
- Keep-alive pings `GET /` every 5 minutes to prevent Render free tier sleep


