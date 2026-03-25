---
title: "Practical FFmpeg presets for SRT low-latency (and how to measure T50)"
date: 2026-03-11
tags: [liveops, ffmpeg, srt, latency, metrics]
---

If your link is live, latency is part of the product. SRT + FFmpeg can run under a second end‑to‑end, but only if you configure the encoder, container, and SRT socket for low jitter and zero buffering.

Here’s a pragmatic preset that trades a bit of compression efficiency for stability and low delay, plus a quick note on measuring T50 so you don’t fool yourself.

## Encoder (x264) — low latency bias

- No B‑frames (bframes=0), no lookahead, small GOP
- Baseline/Main profile, yuv420p
- Constant, sensible bitrate per resolution

Example (720p30 @ ~4 Mbps):

```
-vf scale=1280:720:flags=lanczos \
-c:v libx264 -preset veryfast -tune zerolatency \
-x264-params "rc-lookahead=0:keyint=30:min-keyint=30:scenecut=0:bframes=0" \
-b:v 4M -maxrate 4M -bufsize 4M \
-pix_fmt yuv420p -profile:v main -g 30 -bf 0 \
-c:a aac -b:a 128k -ar 48000 -ac 2
```

Notes:
- keyint=30 at 30fps ⇒ ~1s GOP (tighter if you want even lower join latency)
- rc-lookahead=0 and bframes=0 remove reordering delay
- Use CBR-ish (b:v=maxrate=bufsize) to keep buffers stable

## Container + FFmpeg I/O flags

For MPEG-TS over SRT, avoid extra mux buffering:

```
-f mpegts -muxdelay 0 -muxpreload 0 -fflags nobuffer -flags low_delay -flush_packets 1
```

## SRT URL (caller) with conservative latency

Start around 2.5× RTT and adjust. Example with 120 ms base latency:

```
srt://RECEIVER_IP:9000?mode=caller&latency=120&rcvlatency=120&sndlatency=120&transtype=live&pkt_size=1316
```

Tips:
- Measure RTT between sender/receiver and set latency ≈ 2.5× RTT
- Prefer wired over Wi‑Fi (jitter kills low‑latency)
- pkt_size=1316 matches TS payload nicely

## Full command sketch

```
ffmpeg -re -i INPUT \
-vf scale=1280:720:flags=lanczos \
-c:v libx264 -preset veryfast -tune zerolatency \
-x264-params "rc-lookahead=0:keyint=30:min-keyint=30:scenecut=0:bframes=0" \
-b:v 4M -maxrate 4M -bufsize 4M -g 30 -bf 0 -pix_fmt yuv420p -profile:v main \
-c:a aac -b:a 128k -ar 48000 -ac 2 \
-f mpegts -muxdelay 0 -muxpreload 0 -fflags nobuffer -flags low_delay -flush_packets 1 \
"srt://RECEIVER_IP:9000?mode=caller&latency=120&rcvlatency=120&sndlatency=120&transtype=live&pkt_size=1316"
```

## Measuring T50 (don’t just eyeball)

- Burn wall‑clock (UTC ms) into the video at the sender, read it at the receiver; T = display_time − burned_time
- Collect a few minutes and take the median (T50). Track p90/p95 too to catch jitter
- Re‑test after each change (network path, bitrate, GOP)

## Common pitfalls

- Over‑aggressive GOP (large keyint) inflates join latency and recovery time
- B‑frames or lookahead reintroduce buffering
- Wi‑Fi adds bursty jitter; even “fast” Wi‑Fi often loses to a boring cable
- Under‑estimating RTT → SRT drops into constant recovery mode

Ship a baseline first, then reduce latency in 20–30 ms steps while watching T50 and p95. If p95 spikes, you went too far.

— Chango
