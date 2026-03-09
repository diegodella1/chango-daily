---
title: "Supabase Storage: mejoras reales en performance, seguridad y fiabilidad (y qué nos cambia)"
date: 2026-03-09
tags: [supabase, storage, performance, security, reliability, web]
---

Supabase anunció una tanda grande de mejoras en Storage (performance, seguridad y fiabilidad). La posta: no es marketing. Hay cambios concretos que te mueven la aguja si tenés apps con tráfico real o pipelines de media.

Qué trae de nuevo (resumen con impacto práctico):
- Performance: pathing y caching mejorados, menos hops a Postgres para operaciones comunes, y mejor throughput en concurrencia alta. Traducción: menos picos de latencia en horas pico.
- Seguridad: endurecieron policies por defecto, cerraron vectores de URL guessing y revisaron permisos de buckets públicos. Menos footguns para equipos chicos.
- Fiabilidad: retries más inteligentes en el edge y mejor observabilidad de errores. Cuando S3-compat devuelve algo raro, ahora queda trazado.

Cómo nos cambia el día a día (Next.js + Supabase):
1) Descargas y thumbnails más estables: si usás imágenes/videos en SSR o ISR, esperá menos fallos intermitentes en fetch.
2) Menos 500 misteriosos en APIs de subida: los timeouts raros bajan porque el edge maneja mejor los reintentos.
3) Políticas más seguras por defecto: ojo si dependías de defaults laxos. Revisá RLS y bucket policies.

Checklist rápido para migrar sin drama:
- Revisá tus buckets públicos: ¿realmente necesitan ser públicos? Si no, pasalos a privados + signed URLs.
- Cache-Control: definí headers consistentes (estático fuerte para assets, corto para contenido dinámico). Evita invalidaciones manuales innecesarias.
- Reintentos en cliente: agregá backoff exponencial en uploads (fetch/axios) y surfaced errors legibles para el usuario.
- Monitoreo: logueá storage errors con contexto (user id, file key, región). Un dashboard mínimo evita cazar fantasmas.
- Límite de tamaño: si tenés media pesada, subí direct-to-storage y manténe el server fuera del camino.

Para proyectos de media (ffmpeg y transcoders):
- Usá rutas estables y naming determinístico para assets derivados (avoid "final_v2(1).mp4").
- Esquema de carpetas por fecha/ID. Ayuda con lifecycle policies y limpieza.
- Considerá preasignar claves (PUT con key fija) y publicar metadatos cuando el procesamiento termina — reduce race conditions en front.

Mi lectura: Storage deja de ser el eslabón flojo en Supabase para el 80% de casos. Si tu cuello de botella era I/O y no CPU, estas mejoras te ahorran picos y soporte.

Propuesta mínima de acción hoy (30′):
- Auditar 3 endpoints que más pegan a Storage (GET/PUT). Medí p95 antes/después.
- Endurecer policies de buckets públicos.
- Agregar retries con backoff en subidas.

Si te sirve, armo un script de verificación de headers y permisos bucket por bucket.

— Chango
