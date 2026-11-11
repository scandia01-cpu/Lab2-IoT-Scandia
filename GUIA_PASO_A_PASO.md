# Guía paso a paso · Laboratorio 2 (Scandia)

Archivos ya completos en esta carpeta:

| Archivo | Estado |
|---|---|
| `app.py` | GET `/mediciones` y POST `/mediciones` implementados (probados: 24 registros → `201`) |
| `sketch.ino` | Receta de 4 pasos completa en `setup()`. **Falta solo cambiar `API_URL`** (paso 3) |
| `diagram.json` | Circuito ESP32 + DHT22 (GPIO 15) + OLED SSD1306 (SDA 21, SCL 22) según `plano.svg` |
| `libraries.txt` | DHT sensor library for ESPx · Adafruit GFX Library · Adafruit SSD1306 · Adafruit BusIO |

Orden general: **GitHub → Render → Wokwi → descargar datos → actualizar GitHub**.

---

## 1. Subir el proyecto a un repositorio nuevo de GitHub

1. github.com → **+** → **New repository** → nombre `Lab2-IoT-Scandia` → **Public** → sin README → **Create repository**.
2. Clic en **uploading an existing file** → arrastre **todo el contenido** de esta carpeta (`app.py`, `sketch.ino`, `diagram.json`, `libraries.txt`, `plano.svg`, `README.md`, `.gitignore`, esta guía) → **Commit changes**.

> `.gitignore` empieza con punto y Windows puede ocultarlo: en el Explorador active **Vista → Mostrar → Elementos ocultos** para verlo y arrastrarlo también.

## 2. (Opcional) Probar Flask en su computador

En una terminal dentro de la carpeta:

```
py -3.13 -m pip install flask
py -3.13 app.py
```

Abra `http://127.0.0.1:5000/salud` (→ `{"estado":"ok"}`) y `http://127.0.0.1:5000/mediciones` (→ `[]`). Para probar el POST, abra **otra** ventana de PowerShell:

```powershell
$datos = '{"dispositivo":"doctorado-01","mediciones":[{"hora_simulada":"12:00","temperatura":24.2,"humedad":54.0}]}'
Invoke-RestMethod -Uri 'http://127.0.0.1:5000/mediciones' -Method Post -ContentType 'application/json' -Body $datos
```

Debe responder `recibidas : 1`, y `/mediciones` mostrará el registro. Cierre el servidor con **Ctrl+C**.

## 3. Desplegar la API en Render

1. Entre a <https://dashboard.render.com> (puede iniciar sesión con su cuenta de GitHub).
2. **New → Web Service** → conecte GitHub y elija `Lab2-IoT-Scandia`, rama `main`.
3. Configure:

| Campo | Valor |
|---|---|
| Language | `Python 3` |
| Root Directory | vacío |
| Build Command | `pip install flask gunicorn` |
| Start Command | `gunicorn app:app --bind 0.0.0.0:$PORT --workers 1` |
| Instance Type | `Free` |

4. **Deploy Web Service**. Espere a que diga **Live** (2–5 min).
5. Copie su URL (arriba, p. ej. `https://lab2-iot-scandia.onrender.com`) y pruebe en el navegador:
   - `https://SU-SERVICIO.onrender.com/salud` → `{"estado":"ok"}`
   - `https://SU-SERVICIO.onrender.com/mediciones` → `[]`

## 4. Wokwi

1. Abra <https://wokwi.com/projects/new/esp32>.
2. Pestaña `diagram.json`: Ctrl+A y pegue el `diagram.json` de esta carpeta. Aparecerán el ESP32, el DHT22 y la OLED ya conectados.
3. Cree la pestaña `libraries.txt` (flecha ▾ junto a las pestañas → **New file…**) y pegue el contenido de `libraries.txt`.
4. Pestaña `sketch.ino`: Ctrl+A y pegue `sketch.ino`. **Cambie la línea**
   ```cpp
   const char* API_URL = "https://TU-SERVICIO.onrender.com/mediciones";
   ```
   por su URL de Render, manteniendo `/mediciones` al final.
5. **Save** y ▶. Antes, abra `…/salud` en el navegador para "despertar" Render.

Debe ver en la OLED las horas 00:00 a 23:00 y en el Serial Monitor:

```
Conectando WiFi...
WiFi conectado
Lectura inicial: T=24.0 C  H=55.0 %
Registros enviados: 24
HTTP: 201
{"recibidas":24}
Codigo HTTP final: 201
```

## 5. Verificar y descargar

1. Abra `https://SU-SERVICIO.onrender.com/mediciones` → 24 registros con `hora_simulada`, `temperatura`, `humedad`, `dispositivo`.
2. Abra `https://SU-SERVICIO.onrender.com/descargar` → guarda `mediciones.json`. Guárdelo (Render gratis borra los datos al dormirse).

## 6. Actualizar GitHub con la versión final

En su repositorio → **Add file → Upload files** → suba de nuevo `sketch.ino` (con su `API_URL` real) y, si cambió el circuito en Wokwi, su `diagram.json` → **Commit changes**.

Resultado verificable: repositorio con `app.py`, `sketch.ino`, `diagram.json`, `libraries.txt`; API en Render respondiendo GET y POST; `mediciones.json` descargado con 24 mediciones.

## Problemas frecuentes

| Síntoma | Causa / solución |
|---|---|
| `HTTP: -1` o `-11` | Render dormido o lento: abra `/salud`, espere a que responda y reinicie Wokwi |
| `HTTP: 404` | `API_URL` sin `/mediciones` al final, o URL mal copiada |
| `HTTP: 501` | Render está usando un `app.py` antiguo: verifique que el de GitHub sea el completo y haga **Manual Deploy** |
| `Error: lectura invalida del DHT22` | Revise la conexión DATA del DHT22 a **D15** |
| OLED en negro | Revise SDA → **D21**, SCL → **D22** y dirección 0x3C |
| Error de compilación por librería | Revise que `libraries.txt` tenga las 4 líneas |
| Error de Render en el deploy | Revise que Build/Start Command estén exactamente como en la tabla |
