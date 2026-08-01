# TOLHUIN Chord — Engine

Motor de síntesis y armonía del **TOLHUIN Chord**, un sintetizador de acordes
diatónicos. Este repositorio contiene sólo el **motor**: la parte del código que
**no depende del hardware** y compila en cualquier PC con `g++`/`clang++`.

> Hardware-independent chord-synthesizer engine (pure C++17, no external deps):
> band-limited synthesis + diatonic harmony + a step sequencer, all host-testable.

El firmware completo (ESP32-S3 + DAC I2S, OLED, MIDI USB/BLE, controles físicos)
vive en otro proyecto; acá está el corazón portable y testeado.

---

## Qué incluye

| Módulo | Archivo | Qué hace |
|--------|---------|----------|
| **DSP** | `dsp.cpp/.h` | Osciladores band-limited (PolyBLEP), wavetables, ADSR, filtro SVF, LFO, delay/reverb/chorus/tremolo, percusión, looper, arpegiador y un **reloj maestro** del que derivan su fase batería/arp/trémolo (sync a MIDI clock). |
| **Armonía** | `harmony.cpp/.h` | Motor de armonía **diatónica**: acordes por grado, voicings, conducción de voces, reglas duras (sin dominantes, sin b9/b2). |
| **Secuenciador** | `evloop.cpp/.h` | Motor de eventos/patrones (event loop) puro y testeable. |
| **Tipos** | `config.h`, `state.h` | Constantes globales (sample rate, bloques) y tipos compartidos. |

Todo es **C++17 puro**: sin Arduino, sin FreeRTOS, sin librerías externas. Lo
específico de la placa está detrás de `#ifdef ARDUINO`.

## Compilar y testear

Requiere un compilador C++ de host (`g++` de MinGW/WinLibs, o `clang++`).

**Windows (PowerShell):**

```powershell
powershell -ExecutionPolicy Bypass -File test\run_tests.ps1
```

**Manual (cualquier plataforma):**

```bash
g++ -std=c++17 -I. test/test_harmony.cpp harmony.cpp -o test_harmony && ./test_harmony
g++ -std=c++17 -I. test/test_dsp.cpp     dsp.cpp     -o test_dsp     && ./test_dsp
g++ -std=c++17 -I. test/test_evloop.cpp  evloop.cpp  -o test_evloop  && ./test_evloop
```

Los tests son deterministas (sin oído): verifican ausencia de NaN/clip, RMS,
presencia de fundamentales (Goertzel), timing de la grilla, fase del reloj,
conducción de voces diatónica, etc. `test_dsp` exporta WAVs opcionales para
escucha.

## Uso básico

```cpp
#include "dsp.h"

dspInit();                       // estado limpio
dspNoteOn(60, T_STRINGS);        // C4
dspNoteOn(64, T_STRINGS);        // E4
dspNoteOn(67, T_STRINGS);        // G4

float outL[256], outR[256];
dspRenderStereo(outL, outR, 256); // un bloque estéreo (con efectos)
```

El motor renderiza a un buffer `float`; conectarlo a un DAC/salida de audio es
responsabilidad del integrador (el firmware lo hace por I2S). Ver `dsp.h` y
`harmony.h` para la API completa, y `tools/render_ref.cpp` para un ejemplo de
render a WAV en host.

## Licencia

**GPL-3.0-or-later** (ver [`LICENSE`](LICENSE)). Copyleft: cualquier obra derivada
que se distribuya debe publicarse bajo la misma licencia.

## Nota sobre los samples de batería

`drum_samples.h` contiene datos PCM generados por `tools/gen_drum_samples.py` a
partir de archivos WAV. **Verificá la procedencia/licencia de esos samples antes
de distribuir** si no son propios o de dominio público — o regeneralos desde
fuentes libres. El motor también tiene batería **sintetizada** (sin samples).
