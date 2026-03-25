---
title: "El smoke test que te salva lanzamientos: curl -w y T50 en 60 segundos"
date: 2026-03-06
tags: [devops, performance]
---

Mirá: si antes de lanzar no podés responder “¿cuál es mi TTFB/T50 hoy?”, no estás listo. No hace falta Grafana ni un cluster. Con curl y 60 segundos tenés un smoke test que te evita vergüenzas.

Qué medimos rápido
- TTFB (time_starttransfer): cuándo llega el primer byte.
- TTLB (time_total): cuánto tarda todo el request.
- Errores y códigos HTTP.

Snippet mínimo (bash):

```bash
URL="https://tu-servicio.com/health"
for i in {1..10}; do
  curl -s -o /dev/null \
    -w "run=%{num_connects} dns=%{time_namelookup}s conn=%{time_connect}s ttfb=%{time_starttransfer}s ttlb=%{time_total}s http=%{http_code} bytes=%{size_download}\n" \
    "$URL"
done | tee /tmp/smoke.out
```

Variables de curl -w: time_namelookup, time_connect, time_starttransfer, time_total, http_code, size_download (ver manpage). Con eso ya ves si el cuello está en DNS, conexión o backend.

Umbrales prácticos (mi regla de juego):
- T50 TTFB < 300 ms para orígenes cercanos; 500–700 ms si hay CDN o distancia.
- T50 TTLB < 1000 ms para endpoints simples.
- Error rate ~0% en 10 intentos. Cero drama.

Cómo sacar el T50 rápido:

```bash
awk '{print $4}' /tmp/smoke.out | sed 's/ttfb=//' | sed 's/s//' | sort -n | awk 'NR==5{print "T50_TTFB=" $1 "s"}'
awk '{print $5}' /tmp/smoke.out | sed 's/ttlb=//' | sed 's/s//' | sort -n | awk 'NR==5{print "T50_TTLB=" $1 "s"}'
```

Si querés un mini-burst para estabilidad:

```bash
# oha: https://github.com/hatoo/oha
oha -z 10s -c 4 "$URL"
```

Qué te dice esto en 1 minuto
- DNS lento: time_namelookup alto → fijate resolvers o caché.
- TCP/TLS pesado: time_connect alto → handshake, certificados, red.
- Backend tapado: TTFB alto con connect bien → app/lógica/DB.
- Payload gordo: TTLB se dispara con TTFB normal → compresión/caché.

Ejemplo real: si veo TTFB T50 de 900 ms en un health que pega a DB, corto. Lo bajo a un endpoint estático o pongo un caché simple. No lanzo algo que ya me grita “voy a pedir perdón en producción”.

Dato: Google empuja a bajar TTFB porque impacta en interacción inicial (ver web.dev/ttfb). No por SEO mágico, sino porque la gente se va si no pasa nada rápido.

La posta es que un smoke test no reemplaza observabilidad, pero te da una señal binaria antes de tocar “deploy”. Pasó el umbral, ship. No pasó, arreglá lo obvio y volvé a correrlo. Volver a lo simple te ahorra incidentes que después te comen semanas.

— Chango
