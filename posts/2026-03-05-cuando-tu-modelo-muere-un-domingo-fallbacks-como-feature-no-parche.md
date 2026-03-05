---
title: "Cuando tu modelo muere un domingo: fallbacks como feature (no parche)"
date: 2026-03-05
tags: [ai, reliability, product, agents]
---

Te despertás un lunes y tu agente no responde. No es un bug: apagaron el modelo ayer. El 9 de marzo Google baja Gemini 3 Pro Preview. Si hardcodeaste ese endpoint, tu app se cae sola. No es “mala suerte”. Es diseño pobre.

La posta es que dependés del calendario de otro. Y encima los modelos fallan en patrones caprichosos: timeouts, respuestas vacías, cambios de formato. Papers como ReliabilityBench (2601.06112) vienen mostrando lo obvio: un error se propaga como dominó si no tenés barandas. Y sí, los gateways de IA no son marketing: sirven para rotear, reintentar y caerse con gracia cuando el proveedor estornuda.

¿Qué significa diseñar “fallbacks como feature” y no como parche?

1) SLOs explícitos. Elegí una SLI operativa (p.ej., tasa de tareas completadas y T50/T95 de latencia). Poné un SLO (99,5% de completitud semanal). Sin objetivo, no hay ingeniería; hay fe.

2) Timeouts, retries y backoff. No esperes 60s “por las dudas”. Corta a 8–12s, reintenta 1–2 veces con backoff exponencial y cancela. El usuario prefiere un “no pude, reintentá” rápido a un spinner eterno.

3) Ruteo multi‑proveedor. Primario y secundario definidos por tarea. “Clasificar” puede ir a un modelo barato/local; “razonar largo” al premium. Si el primario falla o degrada (latencia > umbral, tasa de error > X), cortás con un circuit breaker y saltás al backup.

4) Degradación útil. No todo requiere LLM. Para tareas críticas, prepará una versión “tonta” que igual cierre el loop: reglas, plantillas, defaults. Mejor respuesta simple hoy que respuesta brillante nunca.

5) Tests de aceptación agentic. No testees prompts; testea tareas. 10–20 escenarios end‑to‑end corriendo a diario con datos frescos. Si dos o más rompen, entrás en modo “protección”: bajás features no esenciales y priorizás fiabilidad.

Ejemplo concreto: hoy tus llamadas usan gemini‑3‑pro‑preview. Cambiá a un alias interno “reasoner-primary”. Detrás, un gateway evalúa: ¿latencia T50 < 5s y error < 2% en 15 min? Sigue. ¿No? Conmuta a “reasoner‑backup” (3.1 o GPT‑4.x, lo que tengas). Logueá cada salto. Mantené un tablero mínimo con: T50, tasa de éxitos, ratio de fallbacks y error budget quemado.

Y hacé un “chaos hour” semanal: apaga el primario 15 minutos. Si tu negocio tiembla, no tenés arquitectura: tenés suerte.

Mirá: los cierres y deprecations no van a frenar. El switch off del 9/3 es aviso gratis. Si tu producto depende de un solo endpoint, tu negocio depende del humor de otro. Volvé dueño del camino crítico: SLOs, gateway, degradación, y pruebas vivas. Ahí aparece la tranquilidad.

— Chango
