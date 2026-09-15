---
title: "Giving My Amstrad PC1512 a Second Life (and a Second Floppy Drive)"
date: 2026-09-15
tags: [retrocomputing, amstrad, pc1512, gotek, dos]
---

# Giving My Amstrad PC1512 a Second Life (and a Second Floppy Drive)

I've had my Amstrad PC1512 SD for years, but lately its original 5.25" diskettes have been giving up on me — read errors, "abort/retry" prompts, the works. Forty-year-old magnetic media doesn't last forever. This post is the story of how a dying floppy problem turned into a full weekend (okay, several weekends) project: researching, ordering, cutting plastic, flashing firmware, and eventually getting a proper solid-state floppy emulator running as a second drive.

![The PC1512 as it normally sits, with its keyboard and monitor](/assets/images/pc1512-overview.jpg)

## The problem: no such thing as a USB 5.25" drive

My first instinct was to look for a USB 5.25" floppy drive to read the failing diskettes on a modern PC. Turns out that's a dead end — those bridge boards were only ever made for 3.5" drives. 5.25" media predates USB by too long; nobody ever built the adapter. If you're chasing this same idea, stop — it doesn't exist.

## The real fix: Gotek + FlashFloppy

The actual solution the retrocomputing community uses is a **Gotek floppy emulator**. It physically replaces a real floppy drive but reads disk images off a USB stick instead of spinning media. Flash it with the open-source **FlashFloppy** firmware, and it convincingly emulates a 360K/720K 5.25" drive — which happens to be exactly what a PC1512 expects.

A few PC1512-specific things worth knowing if you're doing this yourself:

- The PC1512 doesn't use the IBM "cable twist" trick to distinguish drive A from drive B. Instead, each drive is told which letter it is via a physical **jumper** — DS0 for A:, DS1 for B:.
- The internal floppy cable uses an old-style **card-edge connector**, not the pin-header connector a Gotek ships with by default. You need a card-edge-to-34-pin adapter to bridge the two.
- My machine turned out to already have a second, unused data connector and a spare power connector sitting on the internal cable — meaning I didn't need any extra cabling at all, just the adapter.

![The Gotek board wired up with the card-edge adapter, tapped into the spare internal connectors](/assets/images/gotek-cardedge-adapter.jpg)

## Sourcing parts and finding help

I tracked down a Dutch retro-hardware seller (gotek-retro.eu) who explicitly confirmed the card-edge adapter works on a PC1512 — a nice bit of luck. I ordered a 5.25" Gotek unit with a built-in mounting bracket, OLED display, and rotary encoder, plus the card-edge adapter, and asked the seller to pre-set the drive-select jumper to DS1 before shipping, since I didn't want to mess with a jumper myself.

For the hands-on side of the install, I also looked into local hackerspaces — Maakplek in Groningen and the Oldenburger Computer-Museum near Emden both turned out to be solid options for anyone wanting in-person help with a project like this.

## Cutting the bezel

My PC1512 is the single-drive (SD) variant, with a second, empty drive bay behind a blank plastic cover. That cover, it turns out, was never meant to be user-removable — the original manual explicitly says only an authorized Schneider dealer should fit a second drive. There's no clean pop-out mechanism, just structural clips holding it in place from the factory. In the end, cutting it out with a hobby knife was the correct approach, not a workaround.

![The blank drive-bay cover before cutting it free](/assets/images/bezel-blank-cover.jpg)

## Power and jumpers

The Gotek's onboard power connector is a small floppy-style header, completely different from the big Molex connector already in the case. A €5 Molex-to-floppy-power adapter solved that in about two minutes.

## The software side: NVR, DEVICE, and DOS Plus

Getting the hardware physically installed was only half the job — the machine also needed to be *told* a second drive existed. On the PC1512, that's done through the **Amstrad Non-Volatile RAM Utility**, accessible from a program on one of the original system disks (mine turned out to be renamed `RTC` on this disk revision, rather than the more commonly documented `NVR.EXE`). One menu option, "Number of Disk Drives," is all it takes.

Along the way I also:
- Fixed a dying CMOS battery (this machine uses 4×AA cells tucked under the monitor, plus someone before me had already added an external holder wired in via a spare lead — a common mod, since the original battery setup can leak and corrode the board).
- Rescued a marginal-but-not-dead system disk with a stubborn "Not ready" error on `DEVICE.CMD` — turned out to be a flaky sector, not a dead disk, and a retry got it reading again.
- Learned the PC1512 runs a DOS Plus / MS-DOS hybrid environment, with some non-standard `.CMD` utilities alongside the usual `.COM`/`.EXE` files.

## Building a virtual PC1512 first

Before touching the real hardware repeatedly, I set up **PCem** on Linux (Pop!_OS) as a sandbox — a proper software emulation of an Amstrad PC1512, ROMs and all. This let me safely test disk images, NVR settings, and DOS commands without wearing out the real machine's aging floppy mechanics. Genuinely useful: I could try things, break things, and reset instantly, something you really don't want to be doing on 40-year-old hardware.

## Turning a pile of old game zips into working floppy images

Once the Gotek was running, the next challenge was turning a folder full of abandonware `.zip` downloads into actual bootable floppy images. I wrote a small batch script using `mtools` that:

- Extracts each zip
- Measures the extracted size
- Picks 360K or 720K as appropriate
- Strips obviously unnecessary VGA/EGA/Tandy-only files for games that support multiple graphics standards (since the PC1512 is CGA-only)
- Automatically splits anything still too big across multiple 720K disk images

A few lessons learned the hard way:
- Floppy root directories have a hard limit on the number of files they can hold, regardless of free space — solved by putting files in a subfolder instead of the disk's root.
- Not every DOS-era game is compatible with 8086/CGA hardware — anything from roughly 1990 onward tends to assume a 286 and EGA/VGA as a baseline. A-10 Tank Killer, Prince of Persia 2, Wing Commander, and Norton Commander 5.5 all fall outside what this machine can run; the original Prince of Persia, Lemmings, Tapper, and Wings of Fury are firmly within its era and all work.

## Where it stands now

The Gotek is mounted, wired, and confirmed working as drive B:, browsable via its OLED screen and rotary encoder. The original drive A: is untouched and still fully functional as a fallback. I've got a small library of period-correct games and utilities converted to disk images, and a PCem sandbox for testing anything new before it touches the real hardware.

![Lemmings selected on the Gotek's OLED screen, ready to load from drive B:](/assets/images/gotek-oled-lemmings.jpg)

![Lemmings actually running on the PC1512's CGA monitor](/assets/images/lemmings-running-cga.jpg)

Next up: possibly hooking the PC1512 up to the modern internet over its serial port using mTCP and a WiFi-modem bridge — because if a 1986 machine can run Lemmings again, it can probably also check its email. More on that if it happens.

---

*If you're doing a similar Gotek conversion on a PC1512 or PC1640, feel free to reach out — happy to compare notes on jumpers, connectors, and stubborn floppy disks.*
