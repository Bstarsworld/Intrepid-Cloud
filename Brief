# Intrepid Cloud — Master Manual

**Owner:** Bstar (Bernard Mathis) · Intrepid Media
**Last reconciled:** October 2, 2026
**Domain:** intrepidcloud.net

This single document replaces every earlier manual, recap and handoff. It merges the May 5 session recap, both May 14 ZimaBlade manuals, the May 24 Gemma manual, the June 15 Immich handoff, the August 13 project summary and the September 27 Alienware roadmap. Where those disagreed, the newest source wins, and anything nobody has confirmed on the live server is marked **(verify)**.

---

## Contents

1. What this is
2. Hardware
3. Network and remote access
4. Storage layout
5. Services
6. Workflows
7. War stories: what broke and how it got fixed
8. Command cheat sheet
9. Maintenance schedule
10. Open items
11. Roadmap: Intrepid client tools
12. Rules for AI assistants

---

## 1. What this is

Intrepid Cloud is a self-hosted media production stack for a solo filmmaker. It handles footage ingest, storage, proxy generation, multi-device editing, client delivery, family photo backup and AI tools, running on a low-power ZimaBlade at home instead of Google Drive, Dropbox or iCloud subscriptions.

**Philosophy:** own the hardware, prefer free and open-source tools, and avoid recurring SaaS fees wherever a self-hosted option is good enough. The only fixed recurring cost is the $12/year domain.

**What it is not:** a software project. There is no custom application codebase. It is infrastructure administration: Docker containers, bash scripts, systemd services and configuration on Debian Linux.

---

## 2. Hardware

### ZimaBlade 3760 (the server)

| Part | Spec |
| --- | --- |
| CPU | Intel Celeron N3350, dual-core, **no AVX** |
| RAM | 8GB (~7.8GB usable), shared by every container |
| OS | Debian 12 with CasaOS on top |
| Root disk | 27GB eMMC, **keep under 80%** |
| Network | 2.5GbE PCIe card (add-on) |
| Internet | T-Mobile cellular, roughly 3 to 33 Mbps upload |
| Power cable | Replacement cable, $20 from Target |
| BstarDrive1 | 458GB SSD, part of the mergerfs pool (already owned) |
| BstarDrive2 | 458GB SSD, part of the mergerfs pool (already owned) |
| WD 3TB "Vault" | 2.7TB HDD, Nextcloud file library, **not** in the pool (already owned) |

The three drives were already owned before this build, so they weren't a cost of the project. Their mounts and rules are in section 4, Storage layout.

What the limits mean in practice:

- **No local AI on the server.** No AVX means no Ollama or ML inference worth running. AI lives on the Mac Mini and, later, the Alienware.
- **No hardware decode of GH6 10-bit 4:2:2.** VAAPI was tested and rejected.
- **Load average over 2.0 means overloaded.** Avoid running heavy jobs at the same time.
- **Slow upload.** Anything remote works from small proxies, never originals.
- **LAN is fast.** On the home network, full-resolution footage streams fine.

### Other machines

| Device | Role | Tailscale IP |
| --- | --- | --- |
| Mac Mini M4 (16GB) | Main edit machine, Ollama/Gemma host, Parsec target | <TAILSCALE-IP> |
| MacBook Air M1 | Daily laptop, secondary editing | <TAILSCALE-IP> (verify) |
| iPad A16 (cellular) | Mobile/break editing | <TAILSCALE-IP> (verify) |
| iPhone 12 | Shortcuts and Siri triggers | <TAILSCALE-IP> (verify) |
| Apple Watch Series 9 | Secure ShellFish server widget (setup in progress) | — |
| Alienware 15 R3 | **Planned** CUDA workstation for AI media jobs | — |

Mac Mini MagicDNS name: `<MAGICDNS-NAME>`

### Alienware 15 R3 (planned)

i7-7700HQ, GTX 1060 6GB (Pascal), 16GB RAM going to 32GB, 512GB NVMe plus 1TB HDD, moving to Windows 11 Home. It will be an always-on job box reachable through Tailscale and Parsec. The full build plan lives in `Alienware_15_R3_AI_Workstation_Roadmap.md`.

The one rule that matters most: **only PyTorch builds marked `cu126`.** Newer builds dropped Pascal, start fine, then fail with "no kernel image is available."

---

## 3. Network and remote access

### Tailscale (private access)

Installed **natively** on the ZimaBlade as a systemd service, not as a Docker container.

| Item | Value |
| --- | --- |
| ZimaBlade Tailscale IP | `<TAILSCALE-IP>` |
| ZimaBlade LAN IP | `<LAN-IP>` |

**Use Tailscale for:** large file transfers, the Resolve database, Parsec, SSH and anything admin.

### Cloudflare Tunnel (public access)

No router ports are open. Public HTTPS goes through a Cloudflare Tunnel named **My Cloud1** (SSL mode Full, Always Use HTTPS on). Domain registered at Cloudflare for $12/year.

Cloudflare tuning on the free plan: Browser Cache TTL 1 month, Always Online on, and all three free Page Rules used to cache Nextcloud's `.css`, `.js` and `.png` for a month.

**Cloudflare limit to remember:** uploads through the tunnel are capped at 100MB per request, so big uploads go over Tailscale.

### Parsec

Remote desktop into the Mac Mini: 50 Mbps, 10-bit color, H.265, hardware decode, VSync off.

### Mountain Duck connections (macOS)

| Connection | Protocol | Address | Use |
| --- | --- | --- | --- |
| ZimaBlade raw filesystem | SFTP, port 22, user `casaos` | `<TAILSCALE-IP>` | **Editing** |
| SFTPGo | SFTP, port 2022, user `Bstar` | `/srv/videos` | **Client delivery shares** |
| Nextcloud (fast) | WebDAV over HTTP, port 10081 | `<TAILSCALE-IP>`, path `/remote.php/dav/files/bstar/` | Large transfers on Tailscale |
| Nextcloud (anywhere) | WebDAV over HTTPS, port 443 | `nextcloud.intrepidcloud.net`, same path | 100MB upload cap |
| Seafile | WebDAV, path `/seafdav` | port 8080 via Tailscale or 443 public | Legacy |
| pCloud | Native | pCloud login | 500GB lifetime plan |

The two SFTP connections are easy to mix up. Port 22 is for editing; port 2022 is for delivery.

---

## 4. Storage layout

| Drive | Device | Mount | Size | Holds |
| --- | --- | --- | --- | --- |
| eMMC | `mmcblk0p2` | `/` | 27GB | OS, CasaOS, **Docker images** |
| BstarDrive1 | `sdb1` | `/mnt/BstarDrive1` | 458GB SSD | Databases (direct path), part of pool |
| BstarDrive2 | `sda1` | `/mnt/BstarDrive2` | 458GB SSD | General storage, part of pool |
| WD 3TB "Vault" | `sdc2` | `/media/devmon/sdc2-ata-WDC_WD30EZRZ-00Z` | 2.7TB HDD | Nextcloud file library |

**mergerfs pool:** `/mnt/BstarDrive1:/mnt/BstarDrive2` → `/DATA` (about 915GB).

The WD drive is **not** in the pool and is mounted by `devmon` at an auto-generated path. That path already moved once after a restart and broke Immich (section 7). Nextcloud and SFTPGo both depend on it.

Known UUIDs (confirm with `sudo blkid` before using):

- BstarDrive2: `<UUID>`
- WD 3TB: `<UUID>`

Device letters (`sda`, `sdb`, `sdc`) can swap between boots. Always mount by UUID.

### Storage rules

1. **Databases never go on mergerfs.** MariaDB and Postgres crash-loop on `/DATA`. Use direct paths like `/mnt/BstarDrive1/AppData/<app>/`.
2. **Docker's data-root also can't go on mergerfs.** Docker's overlay2 driver doesn't work on a FUSE filesystem, which is the likely reason both earlier migrations to `/DATA` failed. A future migration should target a direct path such as `/mnt/BstarDrive1/docker`.
3. **Every non-essential drive in `/etc/fstab` gets `nofail`**, so a missing drive can't drop the server into emergency mode.
4. **Check `df -h /` before pulling any image or upgrading.** The eMMC has filled up twice.

### Key paths

| What | Path |
| --- | --- |
| Nextcloud config | `/DATA/AppData/nextcloud/var/www/html/config/config.php` |
| Nextcloud user files | `.../var/www/html/data/Bstar/files/` (on the WD drive; verify exact root) |
| Immich upload root (host) | `/DATA/Gallery/immich` → container `/usr/src/app/upload` |
| Immich Postgres data | `/mnt/BstarDrive1/AppData/immich/pgdata` |
| Immich DB backups | `/DATA/Gallery/immich/backups/` |
| Seafile database | `/mnt/BstarDrive1/AppData/seafile/mysql` |
| Seafile compose file | `/var/lib/casaos/apps/big-bear-seafile/docker-compose.yml` |
| resolve-postgres data | `/mnt/BstarDrive1/AppData/resolve-postgres` |
| Docker data-root | `/var/lib/docker` (still on eMMC) |

---

## 5. Services

| Service | Version | Port | How it runs | Status |
| --- | --- | --- | --- | --- |
| Nextcloud | 33.0.6 | 10081 | Docker | Primary file storage |
| Redis | `redis:alpine` | 6379 | Docker, custom network | Nextcloud cache and locking |
| Immich | v3.0.1 | 2283 | Docker (server, ML, Postgres, Valkey 9) | Photo and video library |
| SFTPGo | v2.7 | 2022 + web | Docker | Client delivery |
| Seafile | 11.0.13 | 8080 | Docker (CasaOS) | Legacy, under-used |
| resolve-postgres | postgres:14 | 5432 | Docker | Retiring (see 6.3) |
| n8n | 1.123.0 | 5678 | Docker | Idle; planned MCP host |
| cloudflared | — | — | systemd or Docker (verify) | Public tunnel |
| Tailscale | — | — | systemd, native | Private VPN |
| webhook | — | 9000 | systemd `webhook.service` | iPhone triggers |
| SD watcher | — | — | systemd `watch-lumix.service` | Auto-trigger currently off (verify) |
| Twelve Labs UI | — | — | systemd, Flask `twelvelabs-ui.py` | AI video search submission |
| Ollama + Gemma 4 | — | 11434 | **Mac Mini**, not the server | Local AI |

### Nextcloud

- Database is SQLite (legacy filename `owncloud.db`). Fine for two users; migration deferred until it's actually needed.
- Web server is Apache. Nginx migration deferred as too risky.
- Redis is reached by the **hostname `redis`** over a custom Docker network. This is the permanent fix for the old IP-shuffle bug, with one catch covered in section 7.
- Tuning in place: APCu local cache, Redis distributed and locking cache, cron every 5 minutes (root crontab), OPcache (128MB, 32MB interned strings, 10,000 files), 16GB PHP upload limit, chunked uploads.
- Preview Generator 5.13.0: squareSizes `32 256`, widthSizes `256 384 1024`, heightSizes `256 1024`, max 2048px, JPEG quality 60.
- Bloated apps disabled or removed (Talk, Deck, Forms, Memories, Recognize and others). `collectives` removed entirely; `threedviewer` reinstalled clean.

### Immich

- On the Postgres/VectorChord image Immich expects, which made the v2 → v3 upgrade low-risk.
- The machine-learning container is the biggest resource hog on the box. Disabling it is under consideration, since Twelve Labs covers video understanding better.
- 1,632 assets as of June.

### Seafile

Kept as a secondary sync option. It survived a symlink crash (section 7) and is rarely used now that SFTPGo handles delivery. A candidate for retirement.

---

## 6. Workflows

### 6.1 Ingest: SD card to edit-ready

The backbone of the whole system: `/usr/local/bin/ingest.sh`.

1. Shoot on the GH6 (or any camera).
2. Insert the SD card into the ZimaBlade.
3. Run ingest manually, or by iPhone Shortcut / "Hey Siri, start ingest."
4. The script identifies the camera from the volume label or DCIM folder prefix: `PANA*` Lumix, `CANON*`/`EOS*` Canon, `NIKON*` Nikon, `SONY*`/`MSDCF*` Sony, `FUJI*` Fuji, otherwise `SD_Card`.
5. Footage is copied into Nextcloud's storage under `YYYY-MM-DD/CameraName/`.
6. FFmpeg makes 1080p H.264 proxies (CRF 23, AAC audio) in `.mov` containers in `YYYY-MM-DD/Proxies/`, **with timecode copied from the source via ffprobe.** Matching filename, matching timecode and a `.mov` container are the three things Resolve needs to link proxies automatically.
7. Files are `chown`ed to `www-data` and Nextcloud rescans so they appear in the web UI.

Details that matter:

- **Atomic writes.** Proxies are written as `file.part.mov` and renamed on success, so an interrupted run never leaves a half-written proxy.
- **Lock file** at `/tmp/ingest.lock` stops two runs overlapping.
- **Formats:** MOV, MP4, MTS, M2TS, AVI, MKV, MXF.
- **Whisper transcription was removed** from the pipeline.
- **Auto-trigger on card insert is currently off.** Whether to turn it back on is an open decision.
- **GH6 advice:** shoot H.265 for daily work. ProRes HQ runs about 14GB per minute, and 6K proxies take roughly 5 to 10 minutes per minute of footage on the Celeron.
- One older note says proxy generation was offloaded to the Mac Mini's VideoToolbox; the August summary says `ingest.sh` encodes with libx264 on the server. **(verify which is live)**

Manual run:

```bash
sudo nohup /usr/local/bin/ingest.sh > /tmp/ingest.log 2>&1 &
tail -f /tmp/ingest.log
```

Webhook triggers (iPhone Shortcuts, configured in `/etc/webhook.conf`):

| Hook | Runs | Shortcut |
| --- | --- | --- |
| `ingest` | `/usr/local/bin/ingest.sh` | Start Ingest / "Hey Siri, start ingest" |
| `refresh` | `/usr/local/bin/nextcloud-refresh.sh` (full file rescan) | Refresh Nextcloud |
| `recover` | `/usr/local/bin/recover-deleted.sh` (pulls files back from the SD card's `.Trashes`) | Recover Deleted |

### 6.2 Client delivery: SFTPGo

Nextcloud's sharing was too slow for big video deliveries, so SFTPGo serves the same files (read-only bind mount) over lean HTTP and SFTP.

- Shares are created **per project folder only**. Never share the root; it contains personal files.
- Shares are anonymous by choice, favoring easy downloads over access control.
- **Not yet tested end-to-end with a real client.**

### 6.3 Editing across devices: DaVinci Resolve

**Today:** a `resolve-postgres` container acts as a free shared project database over Tailscale.

**Planned replacement (not started):**

1. Blackmagic Cloud, about $5/month, for one project library, set to **No Sync** for media so there are no cloud storage fees.
2. Proxies link automatically thanks to the timecode fix in ingest.
3. For the iPad, mark the project's `Proxies/` folder **Available Offline** in the Nextcloud iOS app so proxies download over home Wi-Fi ahead of time. Resolve on iPad can't hold a network file connection once it's backgrounded; that's an iOS limit, not an app choice.
4. At home, switch from proxies to originals with Resolve's one-click toggle.

Once that works, retire `resolve-postgres`.

### 6.4 AI tools

**Gemma on the Mac Mini ("Intrepid AI").** Ollama runs Gemma 4 on the Mac Mini (16GB), reachable over Tailscale at port 11434 and on iPhone/iPad through the Enchanted app. Section 12 holds the rules it follows when helping maintain the server.

**Twelve Labs (cloud API).** Search footage by meaning, not filename.

- Two indexes: an older Marengo-only one (search only) and **Intrepid Main** with Marengo plus Pegasus (search and written descriptions).
- Free tier is 600 minutes total, permanently counted, even after deleting videos.
- A small password-protected Flask UI (`twelvelabs-ui.py`) on the server submits videos from a browser.
- **Gap:** results aren't stored anywhere permanent. A Qdrant vector database was planned but never deployed.

**MCP server (planned, nothing built).** An n8n MCP Server Trigger exposing exactly two tools, Twelve Labs search and Immich search, so Claude can search the footage library. Nextcloud write access is deliberately excluded as too risky. Needs an Immich API key and a new tunnel subdomain.

**Alienware (planned).** Heavy jobs: Video2X upscaling, Flowframes/RIFE interpolation, faster-whisper and WhisperX transcription, Applio voice models, ComfyUI, SAM 2. Jobs will be fed through Nextcloud `Alienware/Inbox` and `Alienware/Outbox` folders. Full plan in the Alienware roadmap.

---

## 7. War stories: what broke and how it got fixed

| What happened | Real cause | Fix | Status |
| --- | --- | --- | --- |
| Nextcloud "Internal Server Error" after every container restart | `config.php` pointed at Redis by IP, and Docker reassigns IPs on the default network | Put Nextcloud and Redis on a custom Docker network and use the hostname `redis` | Fixed, but if the Nextcloud container is ever recreated, it must be reconnected to that network (already forgotten once) |
| "No space left on device," twice | Docker images live on the 27GB eMMC; upgrades filled it | Emergency `docker image prune -a`; the real fix (moving Docker's data-root) is still open | **Unresolved** |
| Nextcloud 32 → 33 upgrade died halfway | Same eMMC problem during `occ upgrade` | Freed space, re-ran `occ upgrade`, it resumed cleanly | Fixed |
| All 1,632 Immich photos showed "Error loading image" | After a restart, `devmon` remounted the WD drive at a different path, so the container's folder was empty | Copied originals, thumbs and encoded video into the path Immich expects, then regenerated thumbnails | Recovered (verify all load); permanent mount fix still open |
| FFmpeg: "unable to find suitable output format" | Temp file named `file.mov.part`; FFmpeg picks the format from the last extension | Renamed the pattern to `file.part.mov` | Fixed |
| Seafile crash loop after a power cut | An interrupted startup script left a symlink pointing at itself | Deleted the bad symlink; the container rebuilt it | Fixed |
| Server stuck in emergency mode after a move | A drive in `/etc/fstab` didn't mount and had no `nofail` | Console, Ctrl+D, add `nofail`, reseat the loose SSD | Fixed |
| Immich Postgres restart loop | Image version didn't match what the database had been upgraded to | Moved to the matching VectorChord image | Fixed |
| Nextcloud app wouldn't update ("Failed to open directory") | Half-corrupted app folder in `custom_apps` | Delete the folder, disable, reinstall | Fixed |
| Nextcloud 33.0.6 image "missing" from Docker Hub | Docker images lag GitHub releases by days | Waited about 4 days | Normal behavior |
| Cloudflare Error 1033, Tailscale offline, "the server crashed" | A four-year-old unplugged the router | Plugged it back in | Not a bug |

### Redis reconnect (after any Nextcloud recreate)

```bash
# Find the network Redis is on
sudo docker inspect redis --format '{{range $k,$v := .NetworkSettings.Networks}}{{$k}} {{end}}'

# Attach Nextcloud to it, then restart
sudo docker network connect <network-name> nextcloud
sudo docker restart nextcloud
```

### Emergency mode recovery

1. Plug in an HDMI monitor and USB keyboard.
2. Press Ctrl+D, log in as `casaos`.
3. Add `,nofail` to any non-essential line in `/etc/fstab`.
4. `sudo reboot`, and physically reseat any drive that didn't mount.

### Restart order after a catastrophic failure

Redis → Nextcloud → Immich Postgres → Immich server → cloudflared. Non-critical services (SFTPGo, previews, n8n) last.

---

## 8. Command cheat sheet

```bash
# Health at a glance
uptime
df -h / /DATA
free -h
sudo docker ps --format "table {{.Names}}\t{{.Status}}"
docker info | grep "Docker Root Dir"

# Shut down safely (never just pull the power)
sudo poweroff

# Nextcloud
sudo docker exec -u www-data nextcloud php occ status
sudo docker exec -u www-data nextcloud php occ files:scan --all
sudo docker exec nextcloud tail -50 /var/www/html/data/nextcloud.log

# Immich
sudo docker logs immich-server --tail 30
sudo docker logs immich-postgres --tail 30
curl -s -o /dev/null -w "%{http_code}" http://localhost:2283/api/server/ping

# Services
sudo systemctl status webhook.service watch-lumix.service
sudo journalctl -u webhook.service -f
sudo tailscale status

# Free eMMC space
sudo docker image prune -a
sudo apt clean
sudo journalctl --vacuum-time=7d
sudo du -h --max-depth=1 / 2>/dev/null | sort -hr | head -20

# Fix a broken Nextcloud app (replace APPNAME)
sudo docker exec nextcloud rm -rf /var/www/html/custom_apps/APPNAME
sudo docker exec -u www-data nextcloud php occ app:disable APPNAME
sudo docker exec -u www-data nextcloud php occ app:install APPNAME
```

---

## 9. Maintenance schedule

**Daily (30 seconds):** CasaOS dashboard, every container green. `tail /tmp/ingest.log` after a shoot.

**Weekly (5 minutes):** `df -h / /DATA`, `occ status`, Nextcloud file scan.

**Monthly (15 minutes):** `occ trashbin:cleanup`, `docker image prune -a`, scan Immich logs for errors, check Alienware temperatures in HWiNFO once it's running.

**Quarterly (1 hour):** update containers (check eMMC space first, back up configs), `apt update && apt upgrade`, review `journalctl -p 3 -xb`, test an SFTPGo share link.

**Yearly:** SMART check on every drive (`sudo smartctl -a /dev/sdX`), plan the next major Nextcloud upgrade.

---

## 10. Open items

🔴 **Backups.** There are no automated backups. One drive failure loses footage or family photos, and this must be in place before client files live here. Minimum: a nightly `rsync` or `rclone` copy of the Nextcloud library, Immich originals and database dumps to a drive outside the pool, plus an encrypted off-site copy (Backblaze B2 via `rclone crypt`, or pCloud).

🔴 **Move Docker's data-root off the eMMC.** Root cause of two outages. Target a direct path such as `/mnt/BstarDrive1/docker`, never `/DATA`. Rather than copying `/var/lib/docker` (which is what broke overlay2 before), consider pointing `daemon.json` at the empty new path and re-pulling images, but **first save the full run settings of every container started with `docker run`**, because those containers won't come back on their own.

🟡 **Mount the WD drive by UUID** at a fixed path like `/mnt/Vault1` instead of the `devmon` path. It already broke Immich once; Nextcloud and SFTPGo would break the same way. Update every container path that references it in the same session.

🟡 **Confirm all 1,632 Immich assets load** on web and mobile.

🟡 **Blackmagic Cloud test**, then retire `resolve-postgres`.

🟡 **First real SFTPGo delivery** to a client.

🟡 **Build the MCP server** (Twelve Labs and Immich search via n8n).

🟢 Decide on SD-card auto-trigger: back on, or permanently manual.

🟢 Decide whether to disable Immich's machine-learning container.

🟢 Nextcloud 34, only after the Docker migration is done.

🟢 Retire Seafile if nothing still depends on it.

🟢 Set storage quotas per Nextcloud user before onboarding clients.

---

## 11. Roadmap: Intrepid client tools

The goal: retainer clients get access to Intrepid's tools as part of the retainer, with paid AI jobs run on approval at prices below services like Topaz Labs.

### Job request flow (with approval)

1. The client drops a clip into their own Nextcloud folder, for example `Requests/Upscale`.
2. Nothing runs automatically. The request lands in a `Queue/Pending` folder and notifies Bstar.
3. Bstar reviews it, quotes it and moves it to `Queue/Approved`.
4. A watcher on the Alienware picks up approved jobs only (the Inbox/Outbox design from the Alienware roadmap).
5. The finished file lands in the client's `Delivered` folder or an SFTPGo share.

### Before offering this to clients

- **Backups first.** Client files on a system with no backups is a liability.
- **One Nextcloud account per client, with quotas.** Clients never see each other's folders or your personal files.
- **Check licenses before charging for any output.** Wav2Lip, ProPainter, MusicGen weights and XTTS are non-commercial. Each tool's license is listed in the Alienware roadmap; confirm on the project page before selling the result.
- **Capacity is one job at a time.** The GTX 1060 has 6GB; the queue is what keeps it honest.
- **Client access to Intrepid AI** needs a design decision: a web chat front-end behind a login, or sharing a single Tailscale node with each client. Never share your whole tailnet.

---

## 12. Rules for AI assistants

Any assistant helping maintain this stack, including Gemma, follows these:

1. **Check before acting.** Run `df -h /` and `docker ps` first. Verify the live system instead of trusting this document's snapshot.
2. **Never run destructive commands without asking**: `rm -rf`, dropping databases, formatting, factory resets.
3. **One step at a time.** One command, wait for the result, then the next, with a short reason why.
4. **Mobile-first.** Bstar is usually on his phone. Keep answers tight.
5. **Be honest about risk.** Say so upfront.
6. **No upselling.** If free or open-source works, don't recommend paid.
7. **Voice typos are normal.** "Xenoblade" means ZimaBlade; "interposed" means Intrepid.
8. **Don't re-suggest these unless asked:** Nginx for Nextcloud, a Nextcloud database migration, VAAPI on the ZimaBlade, Ollama on the ZimaBlade.
9. **Never move databases or Docker's data-root onto `/DATA`.**

