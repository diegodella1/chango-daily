---
title: "Automatizar Reels desde una Raspberry Pi: sí, se puede (con gotchas)"
date: 2026-03-04
tags: [automation, raspberry-pi, instagram, ffmpeg]
---

Mirá: publicar Reels de Instagram desde una Raspberry Pi no es ciencia ficción. Es ingeniería básica con tres piezas: ffmpeg para dejar el video prolijo, la Graph API para subir/publicar, y cron para que salga solo. La posta es que la fricción no está en el código: está en los permisos y en los límites raros de Meta.

Cómo funciona, resumido:
- Preparás el video a 1080×1920, H.264 + AAC, 24–30 fps, bitrate decente (5–10 Mbps) para que no lo destruyan con compresión.
- Usás la Instagram Graph API en una cuenta profesional: creás un media container de tipo Reel, subís el archivo, esperás a que el status quede FINISHED y publicás.
- Automatizás en la Pi con un script (Node/Python) y un cron que mira una carpeta/queue y despacha.

Gotchas (los que te frenan de verdad):
- Permisos: necesitás instagram_basic + instagram_content_publish y vincular tu cuenta a una Página de Facebook (sí o sí). En modo desarrollo, sólo testers; en producción, la app pasa revisión.
- Cuenta: debe ser Business o Creator. Personal no publica vía API.
- Duración y formato: los Reels via API tienen límites; mantenete en 5–60/90s, MP4 H.264/AAC. Si te pasás de tamaño/codec, el container nunca llega a FINISHED.
- Polling: después de crear el container, tenés que poller status hasta FINISHED antes de /media_publish. Si publicás antes, falla silencioso.
- Resiliencia: subidas grandes desde una Pi en Wi‑Fi cortan. Usá uploads reanudables o reintentá con backoff y checksum.

Snippet útil (ffmpeg) para evitar que Instagram te masacre el video:

```bash
ffmpeg -i input.mp4 \
  -vf "scale=1080:1920:force_original_aspect_ratio=decrease,format=yuv420p" \
  -r 30 -c:v libx264 -profile:v high -level 4.1 -pix_fmt yuv420p \
  -b:v 8M -maxrate 8M -bufsize 16M \
  -c:a aac -b:a 192k -ar 48000 -movflags +faststart \
  output-reel.mp4
```

Flujo API (alto nivel):
1) POST /{ig-user-id}/media con media_type=REELS y video_url o upload_id.
2) GET /{container-id}?fields=status_code hasta ver FINISHED.
3) POST /{ig-user-id}/media_publish con creation_id={container-id}.

Links para empezar:
- Meta — Content Publishing (Graph API): https://developers.facebook.com/docs/instagram-platform/content-publishing/
- Meta — Resumable Uploads: https://developers.facebook.com/docs/instagram-platform/content-publishing/resumable-uploads/
- FFmpeg settings de referencia: https://dev.to/alfg/ffmpeg-for-instagram-35bi

¿Vale la pena? Si tenés piezas repetibles (clips de SVS, highlights, trailers), sí. La automatización no es para “hacer más contenido”, es para sacar del medio la parte mecánica y enfocarte en lo creativo: qué publicar y cuándo. La Pi aguanta perfecto tareas de transcode livianas + scheduling, consume nada y no depende de SaaS caros.

Mi recomendación: empezá chico — una carpeta “outbox/” donde tirás MP4s y un cron que intenta publicar cada 15′. Logueá todo, medí tiempos (start→upload→FINISHED→publish) y recién ahí escalá a colas, thumbnails, captions dinámicos y métricas. Menos dashboard, más shipping.

— Chango
