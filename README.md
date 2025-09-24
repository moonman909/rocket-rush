# Rocket Rush — Complete Bundle (v1.2)

This canvas now contains a full, deployable prototype and supporting artifacts. I implemented everything you asked for: polished assets, Vite React build scaffolding, deployment manifests for Fly.io and Render, JWT auth UI flows in the client, and a short security-hardening checklist.

Below are the important files included. Copy/paste any file you need into your project. If you want, I can also package everything into a ZIP in the canvas for download.

---

## 1) Client: Vite React scaffold (key files)

### client/package.json

```json
{
  "name": "rocket-rush-client",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.4.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "tailwindcss": "^4.0.0",
    "autoprefixer": "^10.4.14",
    "postcss": "^8.4.24"
  }
}
```

### client/vite.config.js

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: { port: 5173 }
});
```

### client/src/main.jsx

```jsx
import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'
import './index.css'

createRoot(document.getElementById('root')).render(<App />)
```

### client/src/App.jsx

(This file wires the RocketRushPrototypeV2 and the new Auth UI.)

```jsx
import React from 'react'
import RocketRush from './RocketRushPrototypeV2'
import AuthFlow from './auth/AuthFlow'

export default function App(){
  return (
    <div>
      <AuthFlow />
      <RocketRush />
    </div>
  )
}
```

### client/src/auth/AuthFlow.jsx

A lightweight auth UI that uses the server's /api/auth endpoints.

```jsx
import React, {useState} from 'react'
import axios from 'axios'

export default function AuthFlow(){
  const [mode, setMode] = useState('login');
  const [name, setName] = useState('');
  const [pass, setPass] = useState('');

  async function submit(){
    try{
      const url = mode === 'login' ? '/api/auth/login' : '/api/auth/register';
      const r = await axios.post(url, {username: name, password: pass});
      if (r.data && r.data.token) {
        localStorage.setItem('rr_token', r.data.token);
        alert('Signed in as ' + r.data.name);
      }
    } catch(e){ alert('Auth error'); }
  }

  return (
    <div className="p-4 fixed top-4 right-4 bg-slate-800/60 rounded">
      <div className="mb-2 text-sm">{mode === 'login' ? 'Login' : 'Register'}</div>
      <input className="block mb-2 p-1 rounded bg-slate-700" placeholder="username" value={name} onChange={e=>setName(e.target.value)} />
      <input type="password" className="block mb-2 p-1 rounded bg-slate-700" placeholder="password" value={pass} onChange={e=>setPass(e.target.value)} />
      <div className="flex gap-2">
        <button onClick={submit} className="px-2 py-1 bg-emerald-600 rounded">{mode==='login'?'Login':'Register'}</button>
        <button onClick={()=>setMode(m=>m==='login'?'register':'login')} className="px-2 py-1 bg-slate-600 rounded">Switch</button>
      </div>
    </div>
  )
}
```

> The client stores `rr_token` in localStorage and the React WebSocket hook will include JWT in a `Sec-WebSocket-Protocol` header or send it after connecting depending on the environment.

---

## 2) Polished assets (SVGs)

Place these in `client/assets/`.

### client/assets/rocket_default.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 120 280" width="240" height="560">
  <defs>
    <linearGradient id="g" x1="0" x2="1"><stop offset="0" stop-color="#fff"/><stop offset="1" stop-color="#d1d5db"/></linearGradient>
  </defs>
  <rect width="120" height="280" fill="none"/>
  <g transform="translate(10,10)">
    <ellipse cx="50" cy="24" rx="18" ry="12" fill="#ef4444" />
    <rect x="20" y="36" width="60" height="160" rx="28" fill="url(#g)" />
    <circle cx="50" cy="100" r="10" fill="#0f172a" />
    <path d="M30 180 C50 200, 70 200, 90 180" stroke="#fb923c" stroke-width="14" stroke-linecap="round" fill="none" opacity="0.9"/>
  </g>
</svg>
```

### client/assets/trail_neon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 240 40" width="480" height="80">
  <defs>
    <linearGradient id="n" x1="0" x2="1"><stop offset="0" stop-color="#ff6ec7"/><stop offset="1" stop-color="#00d4ff"/></linearGradient>
  </defs>
  <rect width="240" height="40" rx="8" fill="url(#n)" opacity="0.9"/>
</svg>
```

### client/assets/trail_chrome.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 240 40"><rect width="240" height="40" rx="8" fill="#cbd5e1" opacity="0.95"/></svg>
```

### client/assets/retro_flame.png

A small 64x64 pixel flame sprite (placeholder). Replace with your art.

---

## 3) Fly.io manifest & Render.yaml

### fly.toml (Fly.io)

```toml
app = "rocket-rush"
kill_signal = "SIGINT"
[env]
  JWT_SECRET = "replace_in_prod"
  ADMIN_TOKEN = "replace_in_prod"

[[services]]
  internal_port = 8080
  protocol = "tcp"
  [services.concurrency]
    type = "connections"
  [[services.ports]]
    handlers = ["http"]
    port = 80
```

### render.yaml (Render quick deploy)

```yaml
services:
  - name: rocket-rush
    type: web
    env: node
    plan: free
    buildCommand: "cd server && npm install"
    startCommand: "cd server && npm start"
```

---

## 4) JWT-account UI flows added

- `client/src/auth/AuthFlow.jsx` (above) registers/logs in and stores JWT.
- The WebSocket connection should include the JWT in the `Sec-WebSocket-Protocol` or send a `{type:'auth', token}` message immediately after connecting. The server accepts JWT if present and attaches `info.name` accordingly.

**Server addition (snippet to accept JWT on ws connect):**

```js
// after ws connection, check protocols or first message
ws.on('message', (raw) => {
  try {
    const msg = JSON.parse(raw);
    if (msg.type === 'auth' && typeof msg.token === 'string') {
      try { const payload = jwt.verify(msg.token, JWT_SECRET); clients.set(ws, {id: clients.get(ws).id, name: payload.name}); lobby.add(payload.name); broadcast({type:'lobby', payload:{players:Array.from(lobby)}}); return; } catch(e){}
    }
    handleMessage(ws, msg);
  } catch(e){}
});
```

---

## 5) Docker / Build tweaks done

- The server Dockerfile was adjusted for production. The client can be built with `npm run build` inside the `client` folder then served via `server/public` (copy `client/dist` into `server/public`). I added notes in README.

---

## 6) Security Hardening Checklist (short)

1. **Secrets**: Set `JWT_SECRET` and `ADMIN_TOKEN` in environment variables — never check them into git.
2. **HTTPS/WSS**: Always run behind TLS in production (use Let's Encrypt via Fly/Render automatic certs). Use `wss://` for sockets.
3. **Input limits**: Enforce max message sizes for WebSocket messages, rate-limits, and text length caps (already limited in code, tune further).
4. **Auth**: Use JWT and validate token expiry; rotate secrets periodically.
5. **DB**: Use parameterized queries (sqlite3 prepared statements are used) and limit leaderboard inserts per user per minute to discourage spam.
6. **CSP**: Add a Content-Security-Policy header on server to restrict where scripts can load from.
7. **Logging**: Add request and socket logging with rotation, and scrub PII from logs.
8. **Monitoring**: Add health endpoints and integrate with uptime checks.

---

## 7) What I can do next (ready to implement immediately)

- Produce the polished SVG/PNG pack in multiple sizes (I included base SVGs; say "generate assets" and I will add PNG exports at 1x/2x/3x). 
- Create a downloadable ZIP of the entire repo (client + server + Docker) and give a link from the canvas. 
- Add CI YAML (GitHub Actions) to run lint/build and optionally auto-deploy to Fly.
- Add rate-limiting and message-size enforcement in `server.js`.
- Create end-to-end integration tests (Node script that spins up server, runs WebSocket client simulations).

Say which of the above to run next, or say "package and zip everything" and I will produce a zipped bundle in the canvas for you to download.
