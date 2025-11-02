# RetroTV+ — Product Requirements Document

## Overview

RetroTV+ recreates the classic TV experience for modern media libraries.  
Users upload or point to folders of videos; the system schedules continuous, non-interactive playback across multiple themed “channels.”  
Viewers can tune in on Android TV, Xbox, or the web — always seeing whatever’s “on now.”

---

## Goals

| Goal | Description |
|------|--------------|
| 🕓 Continuous playback | Deterministic 24/7 schedules built from local media |
| 📺 Multi-channel lineup | Multiple channels, each with its own schedule policy |
| ⏰ Wall-clock sync | Every client stays in step with server UTC |
| 🧭 No controls | No seek, pause, or rewind; pure linear flow |
| ⚙️ Simple management | Lightweight web UI to upload, tag, and schedule media |
| 💤 Off-air mode | Configurable test pattern between set hours |
| 🧃 Vintage ads | Optional ad breaks with public-domain 80s/90s commercials |
| 🌍 Cross-platform | Android TV + Xbox UWP clients; shared API |
| ✅ Deterministic testing | Same seed ⇒ same schedule ⇒ reproducible tests |

---

## Non-Goals (V1)

- Multi-user accounts or cloud sync  
- Live transcoding or adaptive bitrate  
- Monetization, analytics, or DRM

---

## Core Features

- **Library scanner** – ffprobe metadata extraction  
- **Scheduler** – deterministic 24 h EPG generation  
- **Channel policies** – folder weights, ordering, ad cadence  
- **Off-air windows** – SMPTE test pattern playback  
- **Ad ingestion** – import public-domain ads from Archive.org  
- **REST API** – `/channels/:slug/now`, `/channels/:slug/epg`, `/library/*`  
- **Web admin** – Plex-lite interface for setup & previews  
- **Android TV / Xbox apps** – display current programming  
- **Shared Zod contracts** – type safety across stack  
- **Automated tests** – schedule determinism, offset math, ad insertion

---

## User Stories

1. **Setup** – Admin adds a folder → server scans → auto-creates a channel.  
2. **Scheduling** – Admin defines policy: shuffle, block size, blackout times.  
3. **Ad breaks** – Admin toggles vintage-ad support (frequency & duration).  
4. **Viewing** – User opens app → plays whatever’s live, no control.  
5. **Off-air** – Between 02:00 – 06:00, a test pattern plays automatically.  

---

## Success Metrics

- < 200 ms response time on `/now`  
- ≥ 99.9 % deterministic EPG generation under identical seeds  
- Zero playback gaps across file transitions  
- CI runs all critical schedule tests in < 2 min

---

## Future Enhancements

- Server-side stitched HLS  
- AI-generated bumpers or dynamic slates  
- Remote shared viewing (“watch together”)  
- Channel discovery & search  
- Per-channel analytics
