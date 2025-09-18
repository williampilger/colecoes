# Detecção de HotWord no Ubuntu

Isso NÃO foi revisado nem testado, ainda.

Serve para detectar uma palavra específica e execuar alguma rotina.

---

## 🐍 Script pronto (Porcupine + notify-send)

Salve como `wakeword.py`:

```python
#!/usr/bin/env python3
import subprocess
import pvporcupine
import pyaudio
import struct

# Caminho para seu modelo .ppn (baixado no Picovoice Console)
KEYWORD_PATH = "/caminho/para/joao.ppn"

# Inicializa Porcupine
porcupine = pvporcupine.create(keyword_paths=[KEYWORD_PATH])

pa = pyaudio.PyAudio()
stream = pa.open(
    rate=porcupine.sample_rate,
    channels=1,
    format=pyaudio.paInt16,
    input=True,
    frames_per_buffer=porcupine.frame_length
)

print("Ouvindo... (Ctrl+C para parar)")

try:
    while True:
        pcm = stream.read(porcupine.frame_length, exception_on_overflow=False)
        pcm = struct.unpack_from("h" * porcupine.frame_length, pcm)

        keyword_index = porcupine.process(pcm)
        if keyword_index >= 0:
            print("Wake word detectada!")
            subprocess.run(["notify-send", "Alerta", "Seu nome foi chamado!"])
except KeyboardInterrupt:
    print("Encerrando...")
finally:
    stream.stop_stream()
    stream.close()
    pa.terminate()
    porcupine.delete()
```

---

## 🚀 Como usar

1. **Instalar dependências** (só uma vez):

   ```bash
   pip install pvporcupine pyaudio
   sudo apt install libportaudio2
   ```

2. **Baixar o modelo personalizado**:

   * Vá em [Picovoice Console](https://console.picovoice.ai/)
   * Crie um wake word com seu nome (ex: “João”)
   * Baixe o arquivo `.ppn`

3. **Editar o script**:

   * Coloque o caminho correto do `.ppn` em `KEYWORD_PATH`

4. **Rodar**:

   ```bash
   python3 wakeword.py
   ```

Quando você disser o wake word, vai aparecer um **popup de notificação** no Linux (`notify-send`).

