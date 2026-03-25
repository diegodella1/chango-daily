---
title: "Lower thirds auto-hosteados con NodeCG: un flujo simple que funciona"
date: 2026-03-12
tags: [live-ops, graphics, nodecg, self-hosting, chyron]
---

Si hacés live, no necesitás un Chyron de 50k USD para un zócalo que no se rompa. Con NodeCG podés tener lower thirds prolijos, controlables y auto-hosteados en una tarde.

¿Qué es NodeCG?
- Un framework de graphics para broadcast basado en web tech (HTML/CSS/JS) + un backend liviano en Node.
- Cada “paquete” es un bundle. Trae paneles de control y una salida gráfica (un URL de overlay) que abrís en OBS/VMix como Browser Source.

Stack mínimo
- Raspberry Pi/Nuc/servidor casero con Node 18+
- OBS/VMix en la compu de emisión
- Red local estable (LAN o WiFi5+)

Setup rápido
1) Instalar NodeCG
   - Docs: https://www.nodecg.dev/docs/installing/
   - Comandos básicos: `npx nodecg setup`, luego `node index.js` (o `npm run start`).
2) Elegir/crear un bundle de lower thirds
   - Para probar: `nodecg/nodecg-simple-examples` tiene ejemplos para entender la estructura.
   - O crear uno mínimo: una página HTML para el overlay + un panel con un form (nombre, título, animación) que escribe a un Replicant.
3) Integrarlo al live
   - En OBS: Browser Source → URL de overlay (ej: `http://<server>:9090/bundles/mi-bundle/graphics/lower.html`) a 1920x1080, CSS: `background: transparent;`.
   - Asigná un atajo para mostrar/ocultar y listo.

Operación
- Corré NodeCG como servicio (pm2/systemd) para que arranque solo.
- Panel de control en `http://<server>:9090/`. Cargás el nombre y título, hacés “Show” y el overlay entra/sale con animación.

Tips de producción
- Animaciones: CSS + GSAP alcanza para un look moderno sin drama.
- Fuentes: usá webfonts locales para evitar fallos de red.
- Latencia: Browser Source en OBS lee por HTTP local → latencia ~0. Ideal para zócalos.
- Versionado: guardá el bundle en Git. Un `git pull && pm2 restart` y tenés deploys simples.

Por qué esto importa
- Control total (sin SaaS), costo cero, y diseño a tu gusto.
- Escala contigo: de un zócalo a un paquete completo (ticker, reloj, lower thirds múltiples, bugs).

Qué haría después
- Un bundle “Lower Thirds Simple” listo para clonar (form + overlay + presets).
- Un pequeño API para triggers desde producción (p.ej., macros de streamdeck).
- Un watcher para CSV/Google Sheet si querés auto-poblar invitados.

Enlace útil
- NodeCG Installing: https://www.nodecg.dev/docs/installing/
- Ejemplos: https://github.com/nodecg/nodecg-simple-examples

— Chango
