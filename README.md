# ✂️ SnipLink — URL Shortener

A full-stack URL shortener built with **React + Vite** on the frontend and **Express + MongoDB** on the backend. Shorten long URLs into clean, Base62-encoded short links with click tracking, idempotent creation, and collision-safe code generation.

![Node.js](https://img.shields.io/badge/Node.js-22+-339933?logo=node.js&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?logo=mongodb&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?logo=express&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-6-646CFF?logo=vite&logoColor=white)

---

## ✨ Features

- **Base62 Short Codes** — 6-character codes using `A-Z a-z 0-9` (~56.8 billion combinations)
- **Idempotent Shortening** — same URL always returns the same short link
- **URL Normalization** — `google.com` and `google.com/` resolve to the same entry (trailing slashes, default ports, query param order)
- **Collision Handling** — automatic retry (up to 5 attempts) on short code collision
- **Click Tracking** — tracks total click count per shortened URL
- **Rate Limiting** — 100 requests per 15 minutes per IP
- **Animated UI** — dark-themed glassmorphism design with particle background
- **Recent Links** — displays the 10 most recently shortened URLs

---

## 🏗️ Tech Stack

| Layer    | Technology                          |
| -------- | ----------------------------------- |
| Frontend | React 19, Vite 6, Vanilla CSS      |
| Backend  | Express 5, Mongoose 9, nanoid 3    |
| Database | MongoDB Atlas (cloud)               |
| Extras   | express-rate-limit, CORS, dotenv    |

---

## 📁 Project Structure

```
bluetick-consultants/
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   │   └── urlController.js    # Core business logic (shorten, redirect, stats)
│   │   ├── models/
│   │   │   └── Url.js              # Mongoose schema (longUrl, shortCode, clicks)
│   │   ├── routes/
│   │   │   └── urlRoutes.js        # API route definitions
│   │   ├── middleware/             # Custom middleware (if any)
│   │   └── server.js              # Express app setup, DB connection
│   ├── .env                       # Environment variables
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Navbar.jsx          # Top navigation bar
│   │   │   ├── ParticleBackground.jsx  # Animated canvas particle effect
│   │   │   ├── RecentLinks.jsx     # Recent shortened URLs list
│   │   │   └── UrlShortener.jsx    # Main URL input + shorten form
│   │   ├── App.jsx                 # Root application component
│   │   ├── main.jsx                # React entry point
│   │   └── index.css               # Global styles (glassmorphism theme)
│   ├── index.html
│   └── package.json
│
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** v18+ (v22 recommended)
- **npm** v9+
- A **MongoDB Atlas** connection string (or a local MongoDB instance)

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/bluetick-consultants.git
cd bluetick-consultants
```

### 2. Backend Setup

```bash
cd backend
npm install
```

Create a `.env` file in the `backend/` directory:

```env
PORT=5001
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>
BASE_URL=http://localhost:5001
```

> **⚠️ macOS Note:** Port `5000` is often occupied by AirPlay Receiver. Use `5001` or disable AirPlay in **System Settings → General → AirDrop & Handoff**.

Start the backend:

```bash
# Development (auto-restart on file changes)
npm run dev

# Production
npm start
```

You should see:

```
✅ Connected to MongoDB Atlas
🚀 Server running on http://localhost:5001
```

### 3. Frontend Setup

```bash
cd ../frontend
npm install
```

Create a `.env` file in the `frontend/` directory:

```env
VITE_API_BASE=http://localhost:5001
VITE_SHORT_DOMAIN=snipl.ink
```

Start the dev server:

```bash
npm run dev
```

The frontend will be available at **http://localhost:5173**.

---

## 📡 API Reference

All API endpoints are prefixed with the backend base URL (e.g. `http://localhost:5001`).

### Shorten a URL

```
POST /api/shorten
```

**Request Body:**

```json
{
  "longUrl": "https://www.example.com/some/very/long/path"
}
```

**Response** `201 Created` (new) or `200 OK` (existing):

```json
{
  "success": true,
  "data": {
    "longUrl": "https://www.example.com/some/very/long/path",
    "shortCode": "2metoq",
    "shortUrl": "http://localhost:5001/2metoq",
    "clicks": 0,
    "createdAt": "2026-04-28T15:00:00.000Z",
    "isExisting": false
  }
}
```

---

### Redirect to Original URL

```
GET /:shortCode
```

Redirects (302) to the original long URL and increments the click counter.

---

### Get URL Stats

```
GET /api/stats/:shortCode
```

**Response** `200 OK`:

```json
{
  "success": true,
  "data": {
    "longUrl": "https://www.example.com/some/very/long/path",
    "shortCode": "2metoq",
    "shortUrl": "http://localhost:5001/2metoq",
    "clicks": 42,
    "createdAt": "2026-04-28T15:00:00.000Z",
    "updatedAt": "2026-04-28T16:30:00.000Z"
  }
}
```

---

### Get Recent URLs

```
GET /api/urls/recent
```

Returns the 10 most recently shortened URLs, sorted by creation date (newest first).

---

### Health Check

```
GET /api/health
```

Returns `200 OK` with server status and timestamp.

---

## 🔧 URL Normalization

To ensure idempotency, URLs are normalized before storage:

| Input                                    | Stored As                          |
| ---------------------------------------- | ---------------------------------- |
| `https://www.Google.COM/`                | `https://www.google.com/`          |
| `https://example.com/path/`             | `https://example.com/path`         |
| `https://example.com:443/`              | `https://example.com/`             |
| `https://example.com?b=2&a=1`           | `https://example.com/?a=1&b=2`     |

This means `google.com` and `google.com/` will always return the **same** short link.

---

### Backend (`/backend/.env`)

| Variable      | Description                      | Default                  |
| ------------- | -------------------------------- | ------------------------ |
| `PORT`        | Backend server port              | `5001`                   |
| `MONGODB_URI` | MongoDB connection string        | *(required)*             |
| `BASE_URL`    | Base URL for generated short links | `http://localhost:5001` |

### Frontend (`/frontend/.env`)

| Variable             | Description                                  | Default                   |
| -------------------- | -------------------------------------------- | ------------------------- |
| `VITE_API_BASE`      | Backend API base URL                         | `http://localhost:5001`  |
| `VITE_SHORT_DOMAIN`  | Domain name to display in shortened links    | `snipl.ink`               |

---

## 🛡️ Rate Limiting

The API enforces rate limiting on all `/api/*` routes:

- **Window:** 15 minutes
- **Max requests:** 100 per IP
- Returns `429 Too Many Requests` with a descriptive error when exceeded

---

## 📜 Available Scripts

### Backend (`/backend`)

| Command         | Description                              |
| --------------- | ---------------------------------------- |
| `npm start`     | Start the server with Node.js            |
| `npm run dev`   | Start with `--watch` (auto-restart)      |

### Frontend (`/frontend`)

| Command            | Description                           |
| ------------------ | ------------------------------------- |
| `npm run dev`      | Start Vite dev server (HMR)           |
| `npm run build`    | Build for production                  |
| `npm run preview`  | Preview production build locally      |

--

<p align="center">
  Built with ⚡ by <a href="https://github.com/isharoy8777"><strong>isharoy8777</strong></a>
</p>
