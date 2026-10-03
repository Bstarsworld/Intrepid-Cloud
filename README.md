# Intrepid Cloud

A self-hosted media server for a working filmmaker, built on a tiny ZimaBlade over six months.

I'm Bernard Mathis (Bstar). I run Intrepid Media, a small video production company. I'm a filmmaker, not an IT guy. I built this because I was tired of paying every month to store my own footage, and I wanted my family's photos on drives in our house instead of someone else's servers.

## What it does

- **Footage ingest:** Insert an SD card, say "Hey Siri, start ingest," and the footage gets sorted by date and camera. It also gets 1080p editing proxies with matching timecode, so DaVinci Resolve links them automatically.
- **Personal cloud storage:** Nextcloud works like a self-hosted Google Drive.
- **Client delivery:** SFTPGo handles large video files.
- **Family photo backup:** Immich works like a self-hosted iCloud Photos.
- **Remote access:** Tailscale and a Cloudflare Tunnel, with no ports open on the router.
- **Local AI:** Gemma runs on a Mac Mini, with this manual as its reference.

## What this is (and isn't)

This is **not a step-by-step tutorial.** It's the real working manual for my system, including the parts that broke and how I fixed them. Take what's useful and adapt it to your own setup.

Passwords, IP addresses, and other private details have been removed. Where you see a masked value, use your own.

## The video series

I'm documenting the whole build on YouTube, including every time it broke: [link coming soon]

## Use at your own risk

This is how *I* run *my* server. It's shared to help, not as a guarantee. Back up your data before you change anything (I learned that the hard way).

---