---
tags: 
  - SIP
  - VoIP
  - Verbio

---
# Use the cli to generate audio file with Verbio TTS

Generate a raw audio file named "ttsdemo_output.raw".

    ttsdemo "aurora" "es" "<prosody rate='150'> Hola, esto es una prueba <break time='300ms'>"

Then use sox app to convert file to 16bits wav audio file.

    sox -t raw -r 8000 -c 1 -e signed-integer -b 16 ttsdemo_output.raw MI_AUDIO.wav
