# 📐 Parámetros Críticos de Certificación

Este documento define los parámetros técnicos medidos por el certificador **Net Chaser TNC950** durante la evaluación de los enlaces UTP del laboratorio, su significado normativo según **TIA/EIA-568**, y el análisis de los valores reales obtenidos en las pruebas de los puertos **P014–P019**.

---

## 🗺️ 1. Mapa de Cableado (Wiremap)

### 📖 Definición

El **Wiremap** o mapa de cableado es la prueba que verifica la **continuidad eléctrica** y la **correcta correspondencia pin a pin** entre los dos extremos del enlace. Comprueba que cada uno de los 8 hilos del cable UTP esté conectado al pin correcto según la norma **T568A** o **T568B**, sin aberturas, cortocircuitos, pares invertidos, pares cruzados (miswires) ni pares divididos (split pairs).

### ⚙️ Cómo afecta a la transmisión

Un error de Wiremap impide físicamente que los datos viajen correctamente entre los equipos. Algunos efectos típicos:

- 🔴 **Circuito abierto (Open)** → el pin no tiene continuidad, no hay enlace.
- 🔴 **Cortocircuito (Short)** → dos hilos se tocan, se pierde la señal diferencial.
- 🔴 **Pares cruzados (Miswire)** → los pines están mal asignados, el switch no detecta el enlace o cae a una velocidad menor.
- 🔴 **Par dividido (Split pair)** → los hilos son continuos pero no pertenecen al mismo par trenzado → aumenta dramáticamente el NEXT.

### 📊 Valores obtenidos en las pruebas

| Puerto | Wiremap | Observación |
|:---:|:---:|:---|
| P014 | ✅ OK | Los 4 pares correctamente conectados |
| P015 | ✅ OK | Continuidad correcta |
| P016 | ✅ OK | Continuidad correcta |
| **P017** | ❌ **OPEN en pines 1-2** | Par naranja abierto → enlace imposible |
| P018 | ✅ OK | Continuidad correcta |
| P019 | ✅ OK | Wiremap correcto (la falla está en BERT) |

---

## 📏 2. Longitud

### 📖 Definición

Es la **distancia física del cable** medida desde un extremo al otro del enlace, expresada en metros. El certificador utiliza la técnica **TDR (Time Domain Reflectometry)** para calcularla: emite un pulso eléctrico y mide el tiempo que tarda el reflejo en regresar, convirtiendo ese tiempo en distancia mediante la velocidad de propagación nominal (NVP) del cable.

### ⚙️ Cómo afecta a la transmisión

La norma **TIA/EIA-568** establece un **límite máximo de 90 metros** para el *enlace permanente* (cable horizontal entre el panel de parcheo y la roseta), más 10 metros adicionales para los patch cords (total 100 m en el *channel*).

- Si se supera esta distancia:
  - 📉 Aumenta la **atenuación** de la señal.
  - 📉 Disminuye la **relación señal/ruido (SNR)**.
  - 📉 Pueden aparecer errores de bit y pérdida de paquetes.

### 📊 Longitudes obtenidas

| Puerto | Longitud | Cumple ≤ 90 m |
|:---:|:---:|:---:|
| P014 | 18.2 m | ✅ |
| P015 | 19.9 m | ✅ |
| P016 | 18.3 m | ✅ |
| P017 | 21.0 m | ✅ (pero abierto) |
| P018 | 19.3 m | ✅ |
| P019 | 19.7 m | ✅ |

🟢 **Conclusión:** Todas las longitudes se encuentran muy por debajo del límite normativo de 90 m, por lo tanto la longitud **no es causa de falla** en ningún punto.

---

## 📉 3. Pérdida de Inserción (Atenuación)

### 📖 Definición

La **atenuación** o **pérdida de inserción (Insertion Loss)** es el **debilitamiento de la señal eléctrica** a medida que viaja a lo largo del cable, medido en **decibelios (dB)**. Depende de la longitud del cable, la frecuencia de la señal, la calidad del cobre y la temperatura.

> A mayor frecuencia → mayor atenuación.
> A mayor longitud → mayor atenuación.

### ⚙️ Cómo afecta a la transmisión

- 📡 Una atenuación excesiva significa que la señal que llega al receptor es demasiado débil para ser interpretada correctamente.
- ⚠️ Esto se traduce en **errores de bit (BERT)**, **retransmisiones TCP** y **caída de la velocidad efectiva**.
- 📐 Los límites de atenuación están definidos por la norma TIA/EIA-568 según la **categoría** del cable y la frecuencia de prueba.

### 🔗 Relación con el SNR medido

El certificador Net Chaser reporta la **Relación Señal/Ruido (SNR)** en dB, que es un indicador directo del margen entre la señal útil y el ruido (incluida la atenuación):

| Puerto | SNR mínimo medido | Interpretación |
|:---:|:---:|:---|
| P014 | 27.2 dB | Buen margen |
| P015 | 28.3 dB | Muy buen margen |
| P016 | 28.3 dB | Muy buen margen |
| P017 | — | No medible (cable abierto) |
| P018 | 28.3 dB | Muy buen margen |
| P019 | 27.2 dB | Suficiente, pero BERT falló |

---

## 🔀 4. Paradiafonía — NEXT (Near-End Crosstalk)

### 📖 Definición

El **NEXT** (*Near-End Crosstalk* o **Paradiafonía**) es la **interferencia electromagnética** que un par trenzado activo induce sobre otro par adyacente, medida en el **extremo cercano** al transmisor. Se expresa en dB y **cuanto más alto es el valor, mejor** (porque indica que el ruido inducido es mucho menor que la señal útil).

### ⚙️ Cómo afecta a la transmisión

- 🌀 Es uno de los parámetros más críticos en redes gigabit, porque **Gigabit Ethernet (1000BASE-T) utiliza los 4 pares simultáneamente** en ambas direcciones.
- ⚠️ Un NEXT bajo genera interferencia entre los pares, degrada el SNR y provoca **errores de bit**.
- 🔧 Las causas típicas de NEXT alto son:
  - Exceso de destrenzado de los pares en el conector (más de 13 mm).
  - Mala calidad del ponchado en el jack o plug RJ-45.
  - Uso de componentes de categoría inferior a la del cable.
  - Curvaturas cerradas del cable.

### 📊 Relación con los resultados BERT

La prueba **BERT** (*Bit Error Rate Test*) inyecta tráfico controlado sobre el enlace y cuenta los errores de bit resultantes. Un BERT fallido suele ser consecuencia directa de problemas de NEXT o atenuación.

| Puerto | BERT | Interpretación |
|:---:|:---:|:---|
| P014 | 0 errores ✅ | NEXT y atenuación dentro de norma |
| P015 | 0 errores ✅ | NEXT y atenuación dentro de norma |
| P016 | 0 errores ✅ | NEXT y atenuación dentro de norma |
| P017 | — ❌ | No evaluable (Open) |
| P018 | 0 errores ✅ | NEXT y atenuación dentro de norma |
| **P019** | **FAIL** ❌ | Errores de bit → posible NEXT alto o mal ponchado |

---

## 🧾 Resumen Normativo

| Parámetro | Límite TIA/EIA-568 (CAT5E, 100 MHz) | Criterio |
|---|:---:|:---|
| Longitud máxima | 90 m (permanent link) | A menor, mejor |
| Insertion Loss | ≤ 24 dB @ 100 MHz | A menor, mejor |
| NEXT | ≥ 30.1 dB @ 100 MHz | A mayor, mejor |
| Wiremap | Continuidad 1:1 en los 8 hilos | Sin aberturas ni cruces |
| BERT | 0 errores de bit | Tolerancia 0 |

---

## 🎯 Conclusión

Los parámetros medidos por el Net Chaser TNC950 permiten certificar si un enlace cumple con la categoría especificada. En este laboratorio, **4 de 6 puntos cumplieron con los criterios**, mientras que los 2 puntos rechazados (P017 y P019) presentan fallas de capa física (Wiremap y BERT respectivamente) que deben ser corregidas antes de poner el enlace en producción.
