---
title: "T50 y fiabilidad: cómo medir un piloto sin autoengañarse"
date: 2026-03-10
tags: [ops, metrics, pilotos]
---

Hook: Si no medís bien, el piloto miente. La ilusión de “anduvo” mata más decisiones que un bug.

Qué medir (lo mínimo que sirve):
- T50: mediana del tiempo “trigger → resultado visible”. No el mejor caso ni el promedio: la mediana.
- Fiabilidad: % de corridas que cumplen dentro de umbral (p. ej., T50 ≤ 8 s) sin intervención manual.

Por qué T50 y no promedio:
- El promedio te esconde colas largas. Dos corridas lentas te deforman la lectura.
- La mediana te dice “lo típico”. Después mirás p95/p99 si necesitás perfilar.

Definir el umbral antes de empezar:
- “Éxito” = resultado correcto dentro de X segundos y sin parches.
- Seteá también el tamaño de muestra (mín. 30 corridas) y el contexto (hora del día, carga, red).

Checklist de piloto sano:
1) Evento de inicio logueado con timestamp consistente.
2) ID de corrida correlacionable punta a punta (trace ID simple).
3) Log del “resultado visible” en el mismo reloj.
4) Script de resumen que calcule T50, p95 y fiabilidad (éxitos/n). Sin Excel manual.
5) Bitácora de incidentes (qué pasó, causa, fix). 

Criterios de graduación (ejemplo):
- Go: T50 ≤ 8 s (p50), p95 ≤ 15 s, fiabilidad ≥ 97% en 7 días, 0 intervenciones manuales > 2 min.
- Iterar: falla uno de los tres; registrar hipótesis y reintentar 48 h.
- Kill: 2 semanas sin mejora o dependencia externa incontrolable.

Errores comunes:
- Cambiar el umbral a mitad del piloto para “pasar”. No.
- Medir feliz path en horario ideal y generalizar.
- No loguear fallas “porque fue una boludez”. Las boludeces son patrones.

Pequeño snippet (pseudo):
```
for run in runs:
  t = result_ts(run) - start_ts(run)
  collect(t, status=ok)
print(p50, p95, reliability)
```

Cerrar con una sola página: gráfico simple de distribución, tabla p50/p95 y fiabilidad, y una decisión escrita.

— Chango