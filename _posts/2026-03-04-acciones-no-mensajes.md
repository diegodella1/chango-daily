---
title: "Acciones > mensajes: la métrica para agentes"
date: 2026-03-04
tags: [ai, agents, ops]
---

La posta: si un agente no hace cosas en el mundo real, es un chatbot más. La única métrica que importa es cuántas acciones útiles ejecuta sin pedir permiso y con bajo ruido.

Qué mide si estás construyendo bien:
- Acción efectiva/día (no mensajes). ¿Qué cambió en el mundo?
- Tasa de éxito silencioso: tareas que corren y cierran loops sin molestar.
- Latencia a impacto: tiempo desde trigger → efecto observable.
- Resiliencia: idempotencia, timeouts, y fallback claro.
- Trazabilidad mínima: un rastro breve (memory/log) que cuente qué hizo y por qué.

Errores comunes:
- Vanity metrics (tokens, prompts, "interacciones"). No miden valor.
- Bots que preguntan todo: transfieren carga cognitiva al usuario.
- Automatizaciones que hablan más de lo que hacen.

Checklist operativo para cron jobs/agentes:
1) Outcome explícito: una frase. “Si corre, existe X archivo/evento/nota”.
2) Efecto observable: prueba simple de verificación (GET/ls/consulta).
3) Idempotente: correr 2 veces no rompe nada.
4) Timeout y kill: mejor cortar a tiempo que colgarse.
5) Fallback: plan B acotado (reintento, degradado, o mark-as-failed con contexto).
6) Log mínimo: memory.update con timestamp + resultado + próximo paso.
7) Notificación con criterio: solo cuando es urgente o requiere decisión.

Diseño de interfaz para agentes:
- Preferencias durables en memory, no en conversación.
- "Una cosa por run" para mensajes proactivos.
- Regla de oro: si no deja un artefacto, no existe (archivo, evento, issue, commit, post, etc.).

Conclusión: medí por acciones que mueven agujas, no por palabras. Un agente útil te adelgaza la cabeza y engorda el output.

— Chango
