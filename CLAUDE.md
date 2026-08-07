# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Implementación de Tetris en JavaScript vanilla (ES6+) con HTML5 Canvas. Sin frameworks, sin dependencias, sin bundler ni transpilador — no hay `package.json`.

## Running the game

No hay build ni test suite. Para ejecutar, simplemente sirve/abre `index.html`:

```bash
open index.html        # macOS
start index.html        # Windows
npx serve .              # o cualquier servidor estático
```

No hay linter ni tests configurados en este repo.

## Architecture

Tres archivos, cada uno con una responsabilidad única:

- **`index.html`** — DOM estático: `<canvas id="board">` (300×600, tablero 10×20 celdas), `<canvas id="next-canvas">` (vista previa de la siguiente pieza), panel de score/lines/level y overlay de pausa/game over.
- **`style.css`** — tema dark/retro arcade; sin lógica.
- **`game.js`** — toda la lógica del juego (~300 líneas, un único archivo sin módulos, todo en el scope global).

### Modelo de datos

- **Tablero**: matriz `ROWS × COLS` (`board[r][c]`); `0` = celda vacía, `1–7` = índice de color de la pieza fijada.
- **Piezas**: definidas en `PIECES` como matrices cuadradas (índice = tipo de pieza). La pieza activa es `{ type, shape, x, y }`.
- **Rotación**: `rotateCW` transpone + invierte filas; no hay matriz de rotación por estado (no es el sistema SRS estándar).

### Flujo principal

```
init() → createBoard() → spawn() (current = next, next = randomPiece())
       → requestAnimationFrame(loop)

loop(ts): acumula dt → si dt ≥ dropInterval, baja la pieza o lockPiece() → draw() → siguiente frame

keydown: mover / tryRotate() / softDrop() / hardDrop() / togglePause()
```

`lockPiece()` encadena `merge()` (fija la pieza en `board`) → `clearLines()` → `spawn()`. Si la pieza recién generada colisiona al aparecer, se dispara `endGame()`.

### Puntos a tener en cuenta al modificar

- **Colisiones** (`collide`) es la única fuente de verdad para límites del tablero y solapes; tanto el movimiento, la rotación (`tryRotate`, con wall kicks ±1/±2 columnas) como el ghost piece (`ghostY`) dependen de ella.
- **Constantes ajustables** en `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval` inicial. Si cambias `COLS`/`ROWS`/`BLOCK`, hay que actualizar manualmente `width`/`height` del `<canvas id="board">` en `index.html` (deben ser `COLS × BLOCK` y `ROWS × BLOCK`).
- **Nivel/velocidad**: el nivel sube cada 10 líneas (`clearLines`); `dropInterval = max(100, 1000 - (level-1)*90)`.
- No hay estado inmutable ni gestión de módulos: `board`, `current`, `next`, `score`, etc. son variables globales mutadas directamente por las funciones del juego.
