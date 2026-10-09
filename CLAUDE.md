# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Clon de Asteroids en HTML5 Canvas puro (UI y comentarios en español). No hay bundler, package.json, linter ni tests. Archivos: `index.html` (canvas 800×600), `game.js` (toda la lógica), `favicon.svg`.

## Ejecutar

Abrir `index.html` en el navegador, o `npx serve .` y visitar `http://localhost:3000`.

## Arquitectura de `game.js`

Un solo script en modo estricto, sin módulos. Secciones en orden:

- **Input**: `keys` (estado sostenido) y `justPressed` (flanco). `pressed(code)` consume el flanco; se usa para disparar (`Space`) y reiniciar, de modo que mantener pulsado no autodispara.
- **Entidades** (`Bullet`, `Asteroid`, `Ship`, `Particle`): cada una con `update(dt)` y `draw()`, y un flag `dead` que se filtra al final del frame. Todo usa `dt` en segundos.
- **Mundo toroidal**: posiciones se envuelven con `wrap(v, W|H)`; la colisión es circular con `dist()` y `radius` (la nave vs. asteroide usa `a.radius * 0.82`).
- **Asteroides**: tamaño 1–3 indexa las tablas `RADII`, `SPEEDS`, `POINTS` (nótese que los puntos son mayores para tamaños menores). `split()` genera dos hijos de `size - 1`.
- **Estado**: variables globales (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) y máquina `state` = `'playing' | 'dead' | 'gameover'`. En `'dead'` hay un `deadTimer` de 2 s antes de `ship.reset()` (que da 3 s de invencibilidad con parpadeo). `initGame()` reinicia todo; `nextLevel()` se dispara cuando `asteroids` queda vacío y spawnea `3 + level` asteroides fuera de `SAFE_DIST` del centro.
- **Power-up Disparo Triple**: `PowerUp` aparece una vez por nivel, al destruir el asteroide número `powerUpKillTarget` (aleatorio 4–20, sorteado en `rollPowerUp()` desde `initGame()` y `nextLevel()`; siempre se alcanza porque cada nivel tiene ≥28 destrucciones posibles). Al recogerlo, `ship.tripleTimer = 10` y `tryShoot()` devuelve 3 balas en abanico. `tripleTimer` vive en el constructor de `Ship` (no en `reset()`) y se pone a 0 en `killShip()`.
- **Loop**: `requestAnimationFrame` con `dt` limitado a 0.05 s; `update(dt)` luego `draw()`.

Las constantes de gameplay (rotación, empuje, drag, cooldown, velocidad de bala) están inline dentro de los métodos de cada clase.
