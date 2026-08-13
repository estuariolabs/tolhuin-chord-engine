# El Último Ritual del Duende Sin Nombre

Groovebox/instrumento sonoro-visual ritual, no una novela visual. El cuento
(una casa-hueco de árbol, seis objetos, un libro que se pudre) es sólo la
estructura de fondo; el centro de la app es la interacción sonora repetitiva
con Web Audio API real.

## Estado actual (fase 1)

- Pantalla **"La Casa"**: escena en pixel art procedural (canvas, sin
  assets externos) con los 6 objetos ubicados en la casa-hueco.
- **Brasero** funcional: cada toque cicla `apagado → brasas → fuego → pira →
  apagado`, sumando/retirando capas de drone grave (osciladores
  saw/square desafinados, con LFO lento de detune y ganancia gradual vía
  `setTargetAtTime`, sin clicks ni golpes de volumen).
- Grafo de audio ya preparado para las siguientes fases: el bus del drone
  pasa por un `BiquadFilter` (para la Pipa) antes del master — hoy "abierto"
  (18kHz), sin efecto audible todavía.
- Los otros 5 objetos (Pipa, Piedras, Ventana/rama, Repisa, Libro) ya están
  dibujados y son tocables en la escena, pero todavía no tienen mecánica de
  audio propia — quedan para las próximas fases.
- Estética CRT: scanlines, dithering, viñeta y ruido de estática (SVG
  `feTurbulence` con semilla reasignada a saltos discretos), paleta fría de
  negro / gris / verde fósforo quemado / ámbar apagado. Todo cambia en
  frames sueltos, sin transiciones suaves.

## Cómo probarla

Es un único archivo HTML sin build. Cualquier servidor estático alcanza,
por ejemplo:

```bash
npx http-server ritual-del-duende -p 8080
# abrir http://localhost:8080/index.html
```

(No se puede abrir con `file://` directo en algunos navegadores por CORS del
módulo de Babel-in-browser; usar un servidor local.)

Requiere un primer click en "tocá para entrar" — los navegadores exigen un
gesto del usuario para arrancar el `AudioContext`.

## Stack

React + Web Audio API vía CDN (sin bundler), todo en `index.html`. Sin
librerías de audio externas. Pixel art dibujado a mano con `fillRect` sobre
`<canvas>` con `image-rendering: pixelated`.

## Próximos pasos

Pipa (filtro + bitcrush), Piedras (secuenciador de percusión ritual),
Ventana/rama (reverb/delay + niebla), Repisa (micro-variaciones de timbre) y
el Libro Sin Título (lectura no lineal disparada por los gestos acumulados
en los otros objetos).
