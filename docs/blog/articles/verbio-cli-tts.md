---
draft: false
date: 2026-05-12
authors: [Visin]
categories:
  - VoIP
  - Asterisk
  - TTS
tags:
  - verbio
  - tts
  - voip
  - asterisk
  - sox
  - audio
  - cli
---

# 🎙️ Domina Verbio TTS desde CLI: De texto a WAV en 2 comandos 🔥

¿Necesitas generar locuciones para tu IVR en Asterisk sin depender de servicios externos? **Verbio TTS** al rescate, y lo mejor: desde la **maldita terminal** como Dios manda 💀⚡

<!-- more -->

---

## 🧠 ¿Qué es Verbio TTS? 🤔

Verbio es un motor de Text-To-Speech que viene incluido en algunos bundles de Asterisk (como los paquetes de voces en español que vimos [aquí](/paquete-voces-en-espa-ol)). Viene con `ttsdemo`, un CLI que te permite generar audio directamente desde tu shell favorita 🐚✨

---

## 🚀 El pipeline mágico: 2 comandos y listo 🪄

### 🔧 Paso 1: Generar RAW con `ttsdemo`

```bash title="🎤 Generar audio raw desde texto"
ttsdemo "aurora" "es" "<prosody rate='150'> Hola, esto es una prueba <break time='300ms'>"
```

**Flags desglosadas 🧩:**

| Parámetro | 🏷️ Valor | 📝 Qué significa |
|-----------|-----------|------------------|
| `aurora` | Nombre de voz 🗣️ | Voz femenina española (hay más voces disponibles) |
| `es` | Código ISO 🌐 | Español (soporta `en`, `fr`, `de`, `pt`, etc.) |
| `3er arg` | String SSML 🏷️ | Texto con etiquetas SSML para control prosódico |

Esto genera el archivo **`ttsdemo_output.raw`** — audio en **raw PCM** sin header, crudo como la potencia bruta del CLI 💪

### 🧪 Paso 2: Convertir RAW → WAV con SoX

```bash title="🔊 Convertir raw a wav con sox"
sox -t raw -r 8000 -c 1 -e signed-integer -b 16 ttsdemo_output.raw MI_AUDIO.wav
```

**Desglose de flags sox 🔍:**

| Flag | 🏷️ Valor | 📝 Explicación técnica |
|------|-----------|------------------------|
| `-t raw` | Tipo RAW | Formato de entrada sin header 🦴 |
| `-r 8000` | Sample rate 📊 | 8kHz — el estándar de oro de la telefonia VoIP ☎️ |
| `-c 1` | Canales 🔉 | Mono (1 canal), que es lo que entiende Asterisk |
| `-e signed-integer` | Encoding 🧮 | Entero con signo (no float, no A-law, no µ-law) |
| `-b 16` | Bit depth 📐 | 16 bits por muestra — calidad CD 📀 |

> 💡 **Dato freak**: 8kHz/16bits/mono es el mismo formato que usa **G.711** (PCM µ-law/A-law), el códec de voz más básico y universal en telefonía IP. ¡Estás generando audio nativo para Asterisk! 🤯

---

## 🧪 Más allá del ejemplo básico 🧬

### 🎛️ SSML Avanzado

```bash title="✨ SSML con control fino"
ttsdemo "aurora" "es" "<speak>
  <prosody rate='80' pitch='+2st'>
    Bienvenido a su sistema telefónico automatizado 📞
  </prosody>
  <break time='500ms'/>
  <prosody rate='120'>
    Pulse <say-as interpret-as='digits'>1</say-as> para ventas,
    pulse <say-as interpret-as='digits'>2</say-as> para soporte técnico 🛠️
  </prosody>
</speak>"
```

**Etiquetas SSML más útiles 🏷️:**

| Etiqueta 🏷️ | 🎯 Efecto | Ejemplo |
|-------------|-----------|---------|
| `<prosody rate='X'>` | Velocidad (50-200%, default 100) | `rate='150'` → 50% más rápido 🏃 |
| `<prosody pitch='X'>` | Tono (±st, ±Hz) | `pitch='+2st'` → 2 semitonos arriba 🎵 |
| `<break time='X'/>` | Pausa (ms, s) | `break time='300ms'` → silencio de 300ms 🤫 |
| `<say-as interpret-as='digits'>` | Leer dígitos uno a uno | `123` → "uno dos tres" 🔢 |
| `<emphasis level='strong'>` | Énfasis en la palabra | `strong` / `moderate` / `reduced` 🔥 |
| `<phoneme alphabet='ipa' ph='...'>` | Pronunciación forzada | Para nombres raros 🧙 |

---

## 🎯 Integración con Asterisk 🔌

```apl title="📞 extensions.conf — Playback en tu dialplan"
[ivr-principal]
exten => s,1,Answer()
    same => n,Wait(1)
    same => n,Playback(custom/IVR/bienvenida)
    same => n,Background(custom/IVR/opciones)
    same => n,WaitExten(5)
```

Y en tu `asterisk.voice.conf` o script de generación:

```bash title="🔄 Batch: generar todas las locuciones del IVR"
#!/bin/bash
# 🏭 Generación masiva de locuciones para tu IVR
VOICE="aurora"
LANG="es"
OUT_DIR="/var/lib/asterisk/sounds/custom/IVR"

declare -A MESSAGES=(
    ["bienvenida"]="Bienvenido a su sistema telefónico"
    ["opciones"]="Para ventas pulse 1, para soporte pulse 2"
    ["despedida"]="Gracias por su llamada. Hasta luego"
    ["error"]="Lo sentimos, opción no válida"
)

for name in "${!MESSAGES[@]}"; do
    ttsdemo "$VOICE" "$LANG" "<prosody rate='140'>${MESSAGES[$name]}</prosody>"
    sox -t raw -r 8000 -c 1 -e signed-integer -b 16 \
        ttsdemo_output.raw "${OUT_DIR}/${name}.wav"
    echo "✅ Generado: ${name}.wav"
done
```

---

## 🔬 Formato RAW: El CODEC oculto 🤫

El archivo `ttsdemo_output.raw` es **PCM lineal signed 16-bit little-endian** a 8kHz. ¿Por qué raw? Porque Verbio escupe el audio directamente sin contenedor — puros **samples en bruto** 🥩.

**Estructura del raw (hexdump):**

```hexdump
00000000  f0 ff 00 00 10 00 20 00  ...sample 1...sample 2...
```

Cada **2 bytes** = 1 sample signed 16-bit LE. Si abres el raw en Audacity: `File → Import → Raw Data → Signed 16-bit PCM → Little-endian → 8000Hz → Mono` 🎛️

---

## ⚡ Bonus: Más formatos de salida con SoX 🛠️

```bash title="🔊 Convertir a otros formatos útiles"
# 📼 GSM (ocupa menos, calidad aceptable para voz)
sox ttsdemo_output.raw -r 8000 -c 1 -e signed-integer -b 16 \
    output.gsm

# 📼 SLN (formato nativo de Asterisk)
sox ttsdemo_output.raw -r 8000 -c 1 -e signed-integer -b 16 \
    output.sln

# 🔊 WAV 44100Hz (calidad CD para música en espera)
sox ttsdemo_output.raw -r 44100 -c 2 -e signed-integer -b 16 \
    output.wav

# 🗜️ MP3 (para web/bienvenidas web)
sox ttsdemo_output.raw -r 44100 -c 1 -e signed-integer -b 16 \
    output.mp3

# 🎙️ U-Lay (G.711 mu-law, directo para canales TDM)
sox ttsdemo_output.raw -r 8000 -c 1 -e signed-integer -b 16 \
    output.ulaw
```

---

## 🐛 Troubleshooting 🩺

| 🚨 Problema | 🩹 Solución |
|-------------|-------------|
| `ttsdemo: command not found` | No tienes instalado el paquete de voces Verbio 📦 |
| `No such voice: aurora` | Prueba con otras voces: `list` o revisa la instalación 🗣️ |
| Audio robótico / pitch incorrecto | Has puesto mal `-b` o `-e` en sox 🤖 |
| Asterisk no reproduce el WAV | Convierte a SLN o verifica `-r 8000 -c 1` ☎️ |
| `sox FAIL formats: can't open output` | Revisa permisos del directorio destino 🔐 |

---

## 🏁 Conclusión

En **2 comandos** tienes locuciones profesionales para tu IVR sin depender de Google TTS, AWS Polly ni nada de la nube ☁️❌. Todo desde tu terminal, todo local, todo **#YOLO en terminal** 💀🔥

```bash
# 📋 Resumen ejecutivo
ttsdemo "aurora" "es" "Tu mensaje aquí" && \
sox -t raw -r 8000 -c 1 -e signed-integer -b 16 \
    ttsdemo_output.raw tu_audio.wav
```

¡A grabar locuciones se ha dicho! 🎙️🔥
