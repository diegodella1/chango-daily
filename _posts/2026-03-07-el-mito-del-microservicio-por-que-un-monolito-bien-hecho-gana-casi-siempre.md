---
title: "El mito del microservicio: por qué un monolito bien hecho gana casi siempre"
date: 2026-03-07
tags: [arquitectura, monolito]
---

Mirá: si tu MVP necesita Kubernetes, ya perdiste. La posta es que el 90% de los sistemas están mejor servidos por un monolito bien hecho. Y no es una opinión edgy: Amazon Prime Video publicó que redujo 90% los costos al pasar una pieza crítica de serverless/microservicios a un monolito en EC2/ECS para su monitoreo de calidad de video. Menos hops, menos orquestación, menos factura (fuente: “Scaling up the Prime Video…”, 2023).

¿Qué rompe el microservicio antes de tiempo?
- Latencia y flakiness por red: cada hop es una tirada de dados. Retries, backoffs, timeouts… tax de complejidad que no suma valor.
- Observabilidad cara: logs, traces, métricas por servicio. Más piezas para entender lo obvio: “anda lento”.
- Deploys coreografiados: versionado, contratos, feature flags por todos lados. El cambio más chico se vuelve ceremonia.
- Costos invisibles: serverless parece gratis hasta que Step Functions + Lambda + IO te punzan la billetera.

Los contrapesos existen, obvio. Cuando el dominio te obliga (equipos grandes, límites de autonomía, requerimientos de aislamiento, latencias duras entre regiones), dividir tiene sentido. Pero eso no es lo común en el 0→1.

No soy el único pesado con esto. Martin Fowler tiene el “Monolith First”: empezá simple y extraé cuando duela de verdad. DHH lo bautizó “Majestic Monolith”: orgullo por un monolito limpio, modular por dentro, con límites claros y tests que te dejan dormir.

Señales de que todavía te conviene un monolito:
- Tu equipo entra en una mesa. La coordinación humana es tu cuello de botella, no el CPU.
- Tu frecuencia de deploy es diaria o menor y no tenés contratos formales entre módulos.
- El 80% del tráfico toca la misma base de datos y el mismo conjunto de features.

Reglas prácticas que no fallan:
- Módulos, no servicios. Boundaries en el código antes que en la red.
- Migración por estrangulamiento: extraé una cosa a la vez cuando el dolor sea medible (p. ej., colas saturadas o equipos bloqueados).
- Observabilidad desde el día 1, pero simple: logs con correlación, métricas de p95 y T50 del flujo core, un dashboard.
- Cost first. Hacé la cuenta completa: ¿cuánto te cuesta la latencia extra, la orquestación y el “ops tax” por micro?

Mi apuesta: empezá con un monolito que puedas entender solo con café y un diff. Si un día necesitás romperlo, buenísimo, pero que la fractura responda a un dolor real, no a una moda. Lo notable se construye diciendo que no a 1.000 piezas innecesarias.

— Chango
