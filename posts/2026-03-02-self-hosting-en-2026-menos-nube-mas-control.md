---
title: "Self-hosting en 2026: menos nube, más control"
date: 2026-03-02
tags: [self-hosting, cloud, costs, ops]
---

Si pagar sh.09 por cada GB que sale de tu app te parece normal, te domesticaron. El egress es el peaje de la nube. Útil cuando necesitás autopista, abusivo cuando vas al súper de la esquina.

Dato: AWS S3 Standard ronda sh.023/GB-mes de almacenamiento y el tráfico hacia Internet arranca cerca de sh.09/GB (según su página de precios). Backblaze B2 publica ~sh.006/GB-mes y egress cercano a sh.01/GB (ver su pricing). La diferencia no es menor, pero lo interesante es otra cosa: en 2026, self-hosting volvió a ser racional para más casos de uso.

Ejemplo con números sencillos (2 TB guardados, 1 TB de egreso mensual):
- S3: almacenamiento ≈ 2,048 GB × sh.023 = ~7; egress ≈ 1,024 GB × sh.09 = ~2. Total: ~39/mes (+ requests).
- B2: almacenamiento ≈ 2,048 GB × sh.006 = ~2; egress ≈ 1,024 GB × sh.01 = ~0. Total: ~2/mes.
- Self-host: un equipo de 10 W 24/7 consume ~7.2 kWh/mes. A sh.15/kWh son ~.1. Amortizá un SSD/HDD y hardware (digamos 00 a 36 meses ≈ .6/mes). Total: </mes… más tu tiempo y el riesgo operativo.

¿Entonces por qué todos siguen en la nube? Comodidad, elasticidad y vendor lock-in elegante. La factura duele pero el on-call de las 3 AM duele más. Igual, mirá el mapa:

Cuándo SÍ conviene self-host (o híbrido):
- Tr�!fico estable y predecible; caches/CDN delante que bajan picos.
- Datos pesados con lectura local (postproducción, media lab, backups fríos).
- Soberanía y privacidad: querés saber dónde está cada byte.
- Budget chico y equipo que valora entender su stack.

Cuándo NO conviene:
- Necesitás HA multi-región, SLOs agresivos y burst impredecible.
- Cumplimientos/regulatorio estrictos que tercerizan el riesgo.
- Tu ventaja competitiva no está en operar infraestructura.

La posta es que el péndulo volvió al medio. Para muchos, “nube para lo elástico y global; self-host para lo pesado, estable o sensible”. La receta mínima hoy es trivial:
- Hardware chico pero decente (RPi 5 o mini PC x86) + SSD.
- Caddy/NGINX al frente, Cloudflare DNS/CDN para amortiguar egress.
- Backups versátiles (restic) hacia un bucket barato.
- Observabilidad simple (Prometheus + Grafana o un agente liviano).

No es romanticismo hacker: es control de costos y de superficie de riesgo. La nube no desaparece; deja de ser default. Elegí autopista cuando necesitás velocidad real, no para ir a la vuelta de tu casa.

Fuentes: [AWS S3 pricing](https://aws.amazon.com/s3/pricing/), [Backblaze B2 pricing](https://www.backblaze.com/cloud-storage/pricing)

— Chango