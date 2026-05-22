---
title: "Installing Home Assistant on a Raspberry Pi"
date: 2026-05-22 10:00:00 +0200
categories: [Smart Home]
tags: [home-assistant, raspberry-pi, haos, setup]
---

Home Assistant OS (HAOS) is the recommended installation method if you want a dedicated, appliance-style setup — no Docker management, no OS tuning, just a system that boots straight into HA and handles its own updates. A Raspberry Pi 4 or 5 is the most common hardware choice for this.

## What You Need

| Item | Notes |
|---|---|
| Raspberry Pi 4 or 5 | Minimum 2 GB RAM; 4 GB or 8 GB recommended |
| MicroSD card (32 GB+) | Look for the **A2** label — it's rated for random I/O, which matters here |
| MicroSD card reader | To flash from your computer |
| Ethernet cable | Use wired for the initial setup; Wi-Fi can be configured later |
| Dedicated power supply | The official Pi PSU or equivalent — phone chargers and laptop USB ports will cause instability |

> **SSD over SD card**: If you plan to run this long-term, a USB SSD is significantly more reliable than a microSD. You can boot from SD and migrate storage later, or buy a Pi 5 with an NVMe HAT and skip SD entirely.

## Step 1 — Flash the Image

Download and install **Raspberry Pi Imager** from [raspberrypi.com/software](https://www.raspberrypi.com/software/).

1. Open Raspberry Pi Imager
2. Under **Operating System**, choose:
   `Other specific-purpose OS` → `Home automation` → `Home Assistant` → select the version matching your hardware (RPi 4 or RPi 5)
3. Under **Storage**, select your SD card
4. Click **Write** — do not apply any custom OS settings, HAOS manages its own hostname and SSH

## Step 2 — First Boot

1. Insert the flashed SD card into the Pi
2. Connect Ethernet
3. Connect power — no monitor needed

Wait 2–5 minutes for the first boot to complete (it pulls container images). Then open a browser on any device on the same network and go to:

```
http://homeassistant.local:8123
```

If that doesn't resolve, find the Pi's IP from your router's DHCP client list and use that directly:

```
http://192.168.x.x:8123
```

## Step 3 — Onboarding

The wizard will walk you through:

1. Creating your owner account (this is the admin account — store the password somewhere safe)
2. Naming your home and setting your location — **set this accurately**, integrations like Mawaqit and sun-based automations depend on it
3. Optionally sharing anonymised analytics with the HA team

After that you land on the default dashboard. The system is ready.

## Step 4 — Recommended Post-Install Basics

**Assign a static IP (or DHCP reservation)**
Your router should always give the Pi the same address. Do this via a DHCP reservation in your router settings — look up your device by MAC address. This prevents automations and external access from breaking when the Pi reboots.

**Install HACS**
Most community integrations (including [Mawaqit](/posts/mawaqit-home-assistant/)) are distributed through HACS. Install it via the terminal add-on:

1. Go to **Settings → Add-ons → Add-on Store** → install **Terminal & SSH**
2. Open the Terminal add-on and run:

```bash
wget -O - https://get.hacs.xyz | bash -
```

3. Restart Home Assistant, then go to **Settings → Devices & Services → Add Integration** and search for HACS

**Enable automatic backups**
Settings → System → Backups → turn on automatic backups. Point them at a network share or cloud storage if possible — the SD card and the Pi can fail together.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `homeassistant.local` doesn't resolve | mDNS blocked on your network — use IP directly |
| Slow or freezing UI | Underpowered PSU or slow SD card — replace both |
| Won't boot | SD card write failed — re-flash and verify checksum |
| Stuck on "Preparing Home Assistant" | First boot can take 10+ min on slow cards — wait it out |

---

Once you're up and running, check out [Integrating Mawaqit Prayer Times into Home Assistant](/posts/mawaqit-home-assistant/) as a first integration to try.
