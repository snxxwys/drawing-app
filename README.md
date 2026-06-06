# Draw Together — Setup Guide

Two repos, two deploys. Takes about 15 minutes total.

---

## Folder Structure

```
drawing-server/   → Deploy to Render (WebSocket relay, free forever)
drawing-app/      → Deploy to Vercel (PWA)
```

---

## Step 1 — Deploy the Server to Render

1. Push `drawing-server/` to a new GitHub repo (e.g. `drawing-server`)
2. Go to [render.com](https://render.com) → New → Web Service
3. Connect your `drawing-server` GitHub repo
4. Render auto-detects the `render.yaml` config — just hit Deploy
5. Once deployed, copy your service URL — looks like:
   `https://drawing-relay-server.onrender.com`

Your WebSocket URL will be:
```
wss://drawing-relay-server.onrender.com
```

> **Note:** Render free tier sleeps after 15 minutes of inactivity.
> The app shows a "Waking up server..." overlay on first connect.
> Usually takes 20–50 seconds. After that it's instant.

---

## Step 2 — Deploy the PWA to Vercel

1. Push `drawing-app/` to a new GitHub repo (e.g. `drawing-app`)
2. Go to [vercel.com](https://vercel.com) → New Project → Import from GitHub
3. Select your `drawing-app` repo
4. Set **Root Directory** to `public`
5. Deploy — Vercel gives you a URL like `drawing-app.vercel.app`

---

## Step 3 — Add Icons (needed for PWA install prompt)

Generate icons at [realfavicongenerator.net](https://realfavicongenerator.net) and save:
- `public/icon-192.png`
- `public/icon-512.png`

---

## Step 4 — Using the App

Open `drawing-app.vercel.app` on both phones.

- Server URL: `wss://drawing-relay-server.onrender.com`
- Room ID: anything shared between both of you (e.g. `us`)

Both phones connect to the same room and see each other's strokes live.

**To install as PWA:**
- iOS: Safari → Share → Add to Home Screen
- Android: Chrome → menu → Add to Home Screen

---

## Step 5 — Connect the ESP32 (later)

The ESP32 connects as `role=display` to the same WebSocket server.

Incoming message format:
```json
{ "type": "draw", "x0": 0.5, "y0": 0.3, "x1": 0.51, "y1": 0.31, "color": "#e8ff47", "size": 6 }
{ "type": "clear" }
```

Coordinates are **normalized 0.0–1.0** — multiply by display width/height to get pixels.

---

## How It Works

```
Your phone  ──draw packets──▶  Render WebSocket Server  ──▶  Her phone
Her phone   ──draw packets──▶  Render WebSocket Server  ──▶  Your phone
```

Both connect as `role=app` to the same room. The server relays everything
between the two. The ESP32 later connects as `role=display`.
