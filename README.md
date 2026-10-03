# NEXVO

NEXVO is a mobile-first Arabic messaging interface with a same-origin Node.js API and a SQLite database.

## Requirements

- Node.js 22.13 or newer (the built-in SQLite module is used).
- Copy `.env.example` to `.env` and replace `ADMIN_EMAIL` and `ADMIN_PASSWORD` with the credentials for this installation. Use a unique administrator password of at least 14 characters; the example intentionally contains no real password.
- `SESSION_SECRET` is optional: when not set, the server generates one and persists it in `NEXVO_DATA_DIR`. Keep that directory on persistent storage in production.
- The password previously shared in chat must not be reused. It was exposed; choose a new unique password.

`npm start` loads the local `.env` file. Render does not load `.env.example`; it supplies `ADMIN_EMAIL`, `ADMIN_PASSWORD`, and the other settings from its service environment. Never put real passwords in `.env.example` or commit secrets.

For production, bind to `0.0.0.0`, use the hosting platform's `PORT`, enable secure cookies, and store `NEXVO_DATA_DIR` on persistent storage.

### Deploying to Render

The repository includes a `render.yaml` Blueprint. In Render, choose **New + → Blueprint**, connect the Git repository containing this project, and provide the requested `ADMIN_EMAIL` and a new `ADMIN_PASSWORD` when prompted. The Blueprint installs the committed Yarn lockfile, starts with `npm start`, checks `/healthz`, binds on `0.0.0.0`, and mounts a persistent disk at `/var/data` for SQLite, uploaded status media, and the automatically generated session secret.

If an existing manually created service crashes with `Missing or unsafe ADMIN_PASSWORD`, open **Render Dashboard → your web service → Environment** and add:

- `ADMIN_EMAIL`: your administrator email.
- `ADMIN_PASSWORD`: a new unique password of at least 14 characters; do not reuse a password exposed in chat.
- `NODE_ENV=production`, `HOST=0.0.0.0`, and `SESSION_COOKIE_SECURE=true`.
- Attach a persistent disk mounted at `/var/data` and set `NEXVO_DATA_DIR=/var/data`. This keeps the generated session secret and user data stable across restarts.

Save the environment changes and redeploy. Do not upload `1.env` or `.env` as Render environment settings; Render does not read those files as service secrets. For an existing service, set **Settings → Build & Deploy → Build Command** to `yarn install --frozen-lockfile`, then commit and push this repository's latest changes before deploying them.

This configuration uses a paid Starter web service because Render persistent disks are not available on the Free plan. Keep a single service instance: SQLite and the attached disk are local to one instance. Push the project and `render.yaml` to your Git provider before creating the Blueprint; Render deploys from that repository.

The project uses **Yarn Classic** and commits `yarn.lock`; `render.yaml` uses `yarn install --frozen-lockfile`. The conflicting npm lockfile has been removed. Existing manually configured Render services must also use **Settings → Build & Deploy → Build Command: `yarn install --frozen-lockfile`**. Updating repository files alone does not change an existing service's manual settings.

For local development, copy `.env.example` to `.env`, set a unique `ADMIN_PASSWORD`, and use `HOST=127.0.0.1`, `PORT=3000`, `NODE_ENV=development`, `SESSION_COOKIE_SECURE=false`, and `NEXVO_DATA_DIR=.data`. Install with `yarn install --frozen-lockfile`, start with `npm start`, and open <http://127.0.0.1:3000>. The administrator credentials come from the environment and are never embedded in browser code.

For deployment, terminate HTTPS at a trusted reverse proxy that forwards WebSocket upgrades and keeps realtime connections open. Keep `NODE_ENV=production` and `SESSION_COOKIE_SECURE=true`, store real secrets in the hosting provider's secret manager, and back up the database and uploaded media together.

## Current server-backed features

- Profile registration with a phone number, username, password, and optional profile image.
- Passwords hashed with Node's scrypt; opaque, HttpOnly, SameSite session cookies backed by SQLite.
- Environment-provisioned administrator sign-in, user directory and account status/verification controls.
- Persistent brand settings and image assets for the app logo, app icon, favicon, default avatar, and Open Graph image.
- Administrative audit log.
- Private one-to-one conversations backed by SQLite, with messages delivered in real time over authenticated WebSockets.
- User lookup by exact username or phone number, without exposing phone numbers in search results.
- Text and supported image/video statuses shared with conversation peers, served from the server, and removed after 24 hours.

Video status media supports byte-range requests for browser playback. The status upload limit is 50 MB per item; supported formats are PNG, JPEG, WEBP, GIF, MP4, and WebM.

Real phone verification, media transcoding/resumable uploads, WebRTC, groups, and end-to-end encryption are not implemented. Messages are stored as text in SQLite and are not end-to-end encrypted. Place the app behind HTTPS in production; administrators do not have an API or dashboard feature to read private conversation content.
