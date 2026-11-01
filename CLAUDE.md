# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Comandos

No hay build, lint, paquetes ni tests: no existe `package.json`, bundler ni framework. No inventes comandos de test o lint.

- Ejecutar: abrir `index.html` en el navegador, o servir la carpeta con un servidor estático (`python3 -m http.server 8000`, `npx serve .`).- Verificar cambios: no hay suite automatizada. Hay que probar en el navegador (teclas, rotación en paredes, limpieza de líneas, Game Over, pausa y reinicio).

## Arquitectura

Todo el juego vive en `game.js` (script clásico cargado al final de `index.html`, sin módulos). Lo que no se ve con una lectura rápida:

- **Estado global mutable**: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`. Todas las funciones leen y escriben estas variables directamente; no hay estado separado para la UI.
- **Ciclo de vida**: `init()` reinicia todo y arranca `requestAnimationFrame(loop)`. `endGame()` cancela el bucle; `togglePause()` lo cancela o lo relanza. Cualquier cambio en el flujo de estados debe pasar por estos tres puntos.
- **Bucle**: `loop` acumula `dt` en `dropAccum` y solo baja la pieza cuando supera `dropInterval`; el dibujado se hace en cada frame con `draw()`. Entrada por `keydown` en `document`, que llama a `updateHUD()` después de cada tecla.
- **Bloqueo de piezas**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. Ahí se decide Game Over (si la pieza nueva colisiona al aparecer).
- **Colisiones**: `collide()` permite celdas con `ny < 0` (encima del tablero) y bloquea solo en paredes laterales, suelo y bloques fijados. Rotación y `ghostY()` dependen de esa regla.
- **Rotación**: `rotateCW` crea una matriz nueva; `tryRotate` prueba desplazamientos `[0, -1, 1, -2, 2]` (wall kicks). Si todos fallan, la rotación se descarta.
- **Puntuación y nivel**: `clearLines` suma `LINE_SCORES[n] × level`, recalcula `level = floor(lines/10) + 1` y `dropInterval`. El hard drop suma 2 por celda y el soft drop 1 por fila (en `hardDrop` / `softDrop`).
- **Dibujado**: el tablero se pinta en `ctx` (300×600) y la siguiente pieza en `nextCtx` (120×120). La fantasma usa `globalAlpha = 0.2` vía `drawBlock`.

Los valores de `COLS`, `ROWS` y `BLOCK` están en `game.js`, pero el tamaño del `<canvas id="board">` está fijado en `index.html`. Si cambias uno, cambia el otro (ver «Personalización» en `README.md`).

## Documentación

`README.md` es la fuente de verdad para controles, flujo del juego y personalización. No copies su contenido aquí; si cambia el comportamiento del juego, actualiza el README.
