# Draw Together — Setup Guide

Two repos, two deploys. Takes about 15 minutes total.

---

## Folder Structure

```
drawing-server/   → Deploy to Railway (WebSocket relay)
drawing-app/      → Deploy to Vercel (PWA)
```

---

## Step 1 — Deploy the Server to Railway

1. Push `drawing-server/` to a new GitHub repo (e.g. `drawing-server`)
2. Go to [railway.app](https://railway.app) → New Project → Deploy from GitHub
3. Select your `drawing-server` repo
4. Railway auto-detects Node.js and runs `npm start`
5. Go to **Settings → Networking → Generate Domain**
6. Copy your domain — it'll look like `drawing-server-production.up.railway.app`

Your WebSocket URL will be:
```
wss://drawing-server-production.up.railway.app
```

---

## Step 2 — Deploy the PWA to Vercel

1. Push `drawing-app/` to a new GitHub repo (e.g. `drawing-app`)
2. Go to [vercel.com](https://vercel.com) → New Project → Import from GitHub
3. Select your `drawing-app` repo
4. Set **Root Directory** to `public`
5. Deploy — Vercel gives you a URL like `drawing-app.vercel.app`

---

## Step 3 — Add Icons (Optional but needed for PWA install)

Generate icons at [realfavicongenerator.net](https://realfavicongenerator.net) and save:
- `public/icon-192.png`
- `public/icon-512.png`

---

## Step 4 — Using the App

Open `drawing-app.vercel.app` on both phones.

**Your phone:**
- Server URL: `wss://drawing-server-production.up.railway.app`
- Room ID: anything (e.g. `us`)
- Role: `app` (hardcoded — this is your drawing side)

**Her phone / ESP32:**
- Same server URL and room ID
- Role: `display`

Both sides draw → see each other's strokes in real time.

**To install as PWA:**
- iOS: Safari → Share → Add to Home Screen
- Android: Chrome → menu → Add to Home Screen

---

## Step 5 — Connect the ESP32

The ESP32 connects as `role=display` to the same WebSocket server.

Incoming message format:
```json
{ "type": "draw", "x0": 0.5, "y0": 0.3, "x1": 0.51, "y1": 0.31, "color": "#e8ff47", "size": 6 }
{ "type": "clear" }
```

Coordinates are **normalized 0.0–1.0** — multiply by display width/height to get pixels.
Color is hex string — parse to RGB for TFT_eSPI.

---

## How It Works

```
Your phone (role=app)
      │  draw packets (normalized coords)
      ▼
Railway WebSocket Server
      │  relays to other role in same room
      ▼
Her phone / ESP32 (role=display)
      │  draw packets back
      ▼
Railway WebSocket Server
      │
      ▼
Your phone (sees her strokes on the "theirs" canvas layer)
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Can't connect | Make sure URL starts with `wss://` not `https://` |
| Strokes don't appear on other side | Check both are using the same room ID |
| Railway keeps sleeping | Free tier stays awake while connections are active |
| PWA won't install | Icons must exist at icon-192.png and icon-512.png |
