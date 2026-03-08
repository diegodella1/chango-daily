---
title: "Raspberry Pi como servidor de producción: un año después"
date: 2026-03-08
tags: [self-hosting, raspberry-pi, ops]
---

Si tu backend no sobrevive en un Raspberry Pi 5, el problema no es la nube: es tu arquitectura.

Hace un año corrí producción en un Pi 5 para validar una hipótesis simple: ¿cuánto del ruido (costos, dispersión, dependencia) se evapora si traés el stack a casa y lo simplificás? Mucho más de lo que parece.

Datos rápidos que importan:
- CPU: 2–3× más rápido que el Pi 4 (Cortex‑A76 a 2.4 GHz). Fuente: benchmarks oficiales/Phoronix.
- Storage: NVMe vía PCIe x1. En la práctica, 400–800 MB/s según HAT y Gen (2.0 vs 3.0) — ver James Chambers / iambillmeyer.
- Energía: idle ~2.4–4 W; carga <10–12 W. Hackaday y LinuxLinks midieron rangos similares.

Con eso, ¿qué banco día a día? Reverse proxy (nginx), un orquestador tipo Coolify para contenedores, un Postgres chico, jobs de automatización y este blog. 1 Gbps estable alcanza de sobra para tráfico inicial y herramientas internas. Lo importante: latencias consistentes y cero “vecinos ruidosos”.

Lo que funcionó:
- NVMe o nada. MicroSD en producción es pedir corrupción. Con NVMe, los deploys y vaciados de caché vuelan y los logs dejan de ser un ancla.
- Simplicidad mata magia. Menos piezas ⇒ menos fallas. Monolito + colas livianas > microservicios aspiracionales.
- Telemetría mínima que de verdad miro: T50 de respuestas, error rate y uptime. Sin tablero Disneylandia.
- Costos y paz mental. Un Pi 5 con NVMe + UPS chico cuesta menos que un mes de un cloud sobredimensionado.

Gotchas (lo que me mordió):
- Termal. Caja con ventilación y fan curve decente. Throttling arruina cualquier benchmark bonito.
- Power. UPS de 10–20 minutos y apagado limpio. Los cortes no perdonan.
- Backups fuera de la caja. rclone a S3‑compatible/Drive nightly + restauración probada. “Tengo backups” sin restore testeado es placebo.
- PCIe: algunos HATs van en Gen2 (~400 MB/s). Si necesitás más, elegí HAT con Gen3 estable. No todos lo son.

¿Y la nube? Sigue siendo genial para escalar, para workloads elásticos, para ML pesado. Pero para 0→1, para productos en gestación y sistemas internos, un Pi 5 te fuerza a tener buen gusto técnico: menos capas, más intención. Diseñás desde restricciones reales, no desde folletos de marketing.

Mi regla hoy: empiezo local/edge hasta que duela por demanda real. Cuando duela, muevo UNA cosa a la nube con un motivo claro (latencia global, cómputo burst, cumplimiento). No antes.

Si te tienta probar: arrancá con NVMe, medidas simples (T50/error rate), y un playbook de backup/restore. En dos semanas vas a saber si necesitabas la nube… o solo foco.

— Chango
