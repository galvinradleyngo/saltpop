# saltpop

A self-hosted [PopSpace](https://popspace.io) virtual collaboration instance — invite anyone via a shareable link, no account required, works in any modern browser and on any device.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| [Docker](https://docs.docker.com/get-docker/) + [Docker Compose](https://docs.docker.com/compose/install/) | v20+ recommended |
| A/V provider | Free [LiveKit Cloud](https://cloud.livekit.io) account (recommended) **or** a [Twilio](https://www.twilio.com) account |
| A public URL | Any domain/IP reachable by your users; `localhost` works for local testing |

---

## Quick start

```bash
# 1. Copy the example env file and fill in your values
cp .env.example .env
$EDITOR .env          # set PUBLIC_URL and A/V credentials

# 2. Start the stack
docker compose up -d

# 3. Open the UI
open http://localhost:8888
```

### Ports used

| Port | Service |
|---|---|
| 8888 | UI (React app) |
| 8889 | HTTP API |
| 8890 | WebSocket server (real-time sync) |
| 8891 | Collaborative document server |

---

## Configuration

All configuration is done through environment variables. Copy `.env.example` to `.env` and edit the values:

```bash
cp .env.example .env
```

### Required variables

| Variable | Description |
|---|---|
| `REACT_APP_LIVEKIT_ENDPOINT` | LiveKit server URL (Option A) |
| `LIVEKIT_API_KEY` | LiveKit API key (Option A) |
| `LIVEKIT_SECRET_KEY` | LiveKit secret (Option A) |
| `PUBLIC_URL` | The URL your users will navigate to, e.g. `https://saltpop.yourdomain.com` |

See `.env.example` for the full list including the optional Twilio and S3 settings.

---

## Creating a room and sharing an invite link

1. Open the app in your browser (`http://localhost:8888` or your `PUBLIC_URL`).
2. Click **New Space** (or **Create a space**) in the top-right.
3. Give the space a name.
4. Once inside, copy the URL from the address bar — that **is** the invite link.  
   Share it with anyone you want to collaborate with.

> Rooms are persistent. Anyone with the link can join at any time.

---

## Inviting users

Simply share the room URL. Recipients:

- Do **not** need an account.
- Can join from any modern desktop or mobile browser.
- Will be prompted for a display name and mic/camera access on their first visit.

See [HOW_TO_JOIN.md](HOW_TO_JOIN.md) for a user-friendly guide you can paste into a chat message or email.

---

## Becoming an admin

Admin privileges let you create and edit room templates.

1. Open the browser console inside the app and run:
   ```js
   client.actor.actorId
   ```
2. Connect to the SQLite database stored in the Docker volume and set `admin = true` for that actor ID:
   ```bash
   docker compose exec saltpop sqlite3 /data/db.sqlite \
     "UPDATE actors SET admin = 1 WHERE id = '<YOUR_ACTOR_ID>';"
   ```

---

## Data persistence

All user data (rooms, files, settings) is stored in a named Docker volume (`popspace-data`). To back it up:

```bash
# Docker Compose prefixes volumes with the project (directory) name.
# If your directory is 'saltpop', the volume will be 'saltpop_popspace-data'.
VOLUME=$(docker volume ls -q -f name=popspace-data | head -1)
docker run --rm \
  -v "$VOLUME":/data \
  -v "$(pwd)/backup":/backup \
  alpine tar czf /backup/saltpop-backup-$(date +%Y%m%d).tar.gz /data
```

---

## Updating

```bash
docker compose pull
docker compose up -d
```

---

## Troubleshooting

**Users can't hear/see each other**  
→ Check that `REACT_APP_LIVEKIT_ENDPOINT`, `LIVEKIT_API_KEY`, and `LIVEKIT_SECRET_KEY` are set correctly in `.env`.

**The app is unreachable from outside**  
→ Ensure ports 8888–8891 are open in your firewall and that `PUBLIC_URL` matches the address users type in their browser.

**iOS users have no audio/video**  
→ iOS only supports WebRTC in Safari. Ask iOS users to open the link in Safari.

---

## Upstream project

This deployment is based on the open-source [with-labs/popspace](https://github.com/with-labs/popspace) project (AGPL-3.0).
