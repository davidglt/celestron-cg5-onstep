# OnStep en ESP32 con CNC Shield — Compilación y puesta en marcha

## Hardware

- **Placa**: ESP32 D1 Mini32
- **Shield**: CNC Shield v3
- **Montura**: Sky-Watcher EQ8-R Pro
- **Plataforma**: Astroberry (Raspberry Pi)

---

## 1. Compilar el firmware

```bash
arduino-cli compile --fqbn esp32:esp32:d1_mini32 ~/OnStep
```

### Error conocido: `tone()` default argument

Con `esp32:esp32` 2.0.17 aparece este error:

```
/home/astroberry/OnStep/src/HAL/ESP32/Analog.h:54:97: error: default argument given for parameter 3 of 'void tone(...)'
```

**Solución**: eliminar el argumento por defecto duplicado en `Analog.h`:

```bash
python3 -c "
import re
with open('/home/astroberry/OnStep/src/HAL/ESP32/Analog.h', 'r') as f:
    content = f.read()
content = content.replace(
    'void tone(uint8_t pin, unsigned int frequency, unsigned long duration = 0)',
    'void tone(uint8_t pin, unsigned int frequency, unsigned long duration)'
)
with open('/home/astroberry/OnStep/src/HAL/ESP32/Analog.h', 'w') as f:
    f.write(content)
print('Hecho')
"
```

Verifica:
```bash
grep "weak.*tone" ~/OnStep/src/HAL/ESP32/Analog.h
# Debe mostrar la línea SIN '= 0'
```

### Resultado esperado tras compilar

```
Sketch uses 1226173 bytes (93%) of program storage space.
Global variables use 45428 bytes (13%) of dynamic memory.
```

---

## 2. Subir el firmware

```bash
arduino-cli upload \
  --fqbn esp32:esp32:d1_mini32 \
  --port /dev/ttyUSB0 \
  ~/OnStep
```

Si da error de permisos en el puerto:
```bash
sudo usermod -a -G dialout astroberry
# Cerrar sesión y volver a entrar
```

### Resultado esperado

```
Wrote 1232752 bytes (783950 compressed) at 0x00010000 in 11.3 seconds
Hash of data verified.
Hard resetting via RTS pin...
```

---

## 3. Verificar comunicación LX200

OnStep no envía texto continuo por serie; responde a comandos LX200 con `#` como terminador.

```bash
# Verificar que OnStep responde
echo -e ":GVP#" | socat - /dev/ttyUSB0,b9600,raw,echo=0,crnl
# Respuesta esperada: On-Step
```

Otros comandos útiles:

```
:GU#   → Estado general del sistema (flags)
:GC#   → Fecha actual
:GL#   → Hora local
:Gg#   → Longitud
:Gt#   → Latitud
:Te#   → Activar motores
:Mw#   → Mover eje RA (velocidad guiado)
:Q#    → Parar
```

### Flags de :GU#

| Flag | Significado |
|------|-------------|
| `N` | Motores activos |
| `n` | Motores desactivados |
| `H` | En Home |
| `P` | Parking |
| `I` | Sin inicializar |
| `G` | En GoTo |

---

## 4. Asignación de drivers en CNC Shield

El CNC Shield v3 tiene 4 sockets:

| Socket | Color (este setup) | Eje OnStep | Función |
|--------|--------------------|------------|---------|
| **X**  | Amarillo | AXIS1 | AR (Ascensión Recta) |
| **Y**  | Amarillo | AXIS2 | DEC (Declinación) |
| **Z**  | Amarillo | AXIS3 | No usado / Foco |
| **A**  | Rojo     | AXIS4 | No usado / Foco 2 |

> Para la EQ8-R Pro solo se usan los sockets **X** e **Y**.

### Orientación de los drivers

- Con **TMC2209**: el pin EN queda hacia el conector de alimentación (fila de abajo).
- Ajustar la corriente **antes** de conectar los motores.

---

## 5. Conexión INDI

- Driver: `indi_lx200_OnStep`
- Puerto: `/dev/ttyUSB0`
- Baudrate: `9600`

---

## Librerías utilizadas

| Librería | Versión |
|----------|---------|
| BluetoothSerial | 2.0.0 |
| Wire | 2.0.0 |
| EEPROM | 2.0.0 |
| Rtc by Makuna | 2.5.0 |
| Adafruit BME280 | 2.3.0 |
| Adafruit BusIO | 1.17.4 |
| SPI | 2.0.0 |
| Adafruit Unified Sensor | 1.1.15 |

**Plataforma**: `esp32:esp32` 2.0.17
