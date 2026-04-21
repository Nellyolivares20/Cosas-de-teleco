# 🧰 Equipo de Certificación — Net Chaser TNC950

Este documento describe el certificador de red utilizado durante el laboratorio, su procedimiento de configuración inicial y las principales funciones empleadas durante las pruebas de los puntos **P014–P019**.

---

## 📸 Ficha del Equipo

| Campo | Valor |
|---|---|
| 🏷️ **Marca** | Platinum Tools |
| 🔧 **Modelo** | Net Chaser TNC950 |
| 🎯 **Tipo de equipo** | Ethernet Speed Certifier |
| 🧵 **Tipos de cable soportados** | UTP / STP / FTP |
| 📐 **Categorías soportadas** | CAT5e, CAT6, CAT6a |
| 📏 **Rango de medición** | Hasta 300 m (TDR) |
| ⚡ **Prueba de PoE** | Sí — hasta 802.3at |
| 🌐 **Prueba de red activa** | DHCP / Ping / BERT |
| 📊 **Parámetros certificados** | Wiremap, Length, Skew, SNR, BERT |
| 🖨️ **Reportes** | Exportables en PDF vía software Net Chaser |

---

## 🎛️ Descripción General

El **Net Chaser TNC950** es un certificador de velocidad Ethernet de nivel intermedio que combina pruebas de **capa física** (wiremap, longitud, skew, SNR) con pruebas de **capa de enlace y red** (BERT, DHCP, Ping, PoE). Está compuesto por:

- 🟣 **Unidad principal (Main)** — con pantalla a color y botones de navegación.
- 🟢 **Unidad remota activa (Active Remote)** — que se conecta al otro extremo del enlace y permite la identificación de puertos (AR ID) y la medición bidireccional.

---

## ⚙️ Configuración Previa al Test

Antes de ejecutar cualquier prueba, el técnico debe configurar los siguientes parámetros en el certificador. Esta configuración fue la utilizada durante el laboratorio para la certificación de los puertos **P014–P019**:

### 🪛 Parámetros a configurar

| Parámetro | Valor seleccionado | Justificación |
|---|---|---|
| 🧵 **Tipo de cable** | UTP | El cableado del laboratorio es par trenzado no apantallado |
| 📐 **Categoría** | CAT5E | Categoría detectada y declarada del enlace |
| 📏 **Estándar de prueba** | TIA | Norma vigente en Chile y Latinoamérica |
| 🔗 **Tipo de enlace** | Permanent Link | Se certifica el cableado horizontal (sin patch cords) |
| 🆔 **ID del punto** | P014–P019 | Identificación de cada toma |

> ℹ️ **Nota:** La selección ISO o TIA depende de lo que permita el equipo. En este caso se utilizó TIA por ser el estándar de referencia habitual.

---

## 🛠️ Funciones Principales del Certificador

### 1️⃣ 🧪 Auto-Test

El **Auto-Test** ejecuta de manera automática la **suite completa de pruebas** según la categoría configurada (CAT5e, CAT6 o CAT6a). En una sola acción el equipo realiza:

- ✅ Verificación del **Wiremap**.
- 📏 Medición de **longitud** por TDR.
- 📊 Cálculo de **skew** (diferencia de tiempo entre pares).
- 🔀 Evaluación de la **SNR** de cada par.
- 🔢 Ejecución del **BERT** (Bit Error Rate Test).
- 📶 Validación de capacidad para **VoIP** y **1 Gbps**.

🔹 El resultado final se muestra como **PASS** (✅ verde) o **FAIL** (❌ rojo) en la pantalla, junto con el detalle por pin.

🔹 En nuestras pruebas, el Auto-Test permitió identificar rápidamente que **P017 estaba abierto** en los pines 1-2 y que **P019 fallaba el BERT** aun cuando el wiremap era correcto.

---

### 2️⃣ ⚡ Verificación de PoE (Power over Ethernet)

La función de **PoE Test** permite detectar si el puerto de un switch está entregando alimentación eléctrica a través del cable y medir su capacidad. Reporta:

- 🔌 **Modo de alimentación** (Mode A o Mode B → qué pares llevan la energía).
- 🏷️ **Tipo/estándar detectado** (802.3af, 802.3at o 802.3bt).
- ⚡ **Potencia disponible** en vatios (W).
- 🔋 **Voltaje con carga mínima y con carga máxima**.

📊 **Resultado obtenido en el laboratorio:**

| Parámetro | Valor |
|---|---:|
| Modo | A |
| Pares con PoE | 1-2 y 3-6 |
| Estándar | **802.3af** |
| Potencia | **12.95 W** |
| Voltaje @ carga mínima | 50.1 V |
| Voltaje @ carga máxima | 48.9 V |

🟢 Esto confirma que el switch del rack entrega PoE 802.3af de acuerdo a la especificación (44–57 V, hasta 15.4 W en el PSE / 12.95 W en el PD).

---

### 3️⃣ 🌐 Prueba de DHCP / Ping

Esta función valida la **conectividad lógica** del enlace una vez confirmada la capa física. El certificador actúa como un host más en la red:

- 🔎 Solicita una **dirección IP al servidor DHCP**.
- 🎯 Recibe máscara, gateway y DNS.
- 📡 Ejecuta un **ping** contra el gateway u otra IP objetivo.
- 📉 Reporta **latencia**, **pérdida de paquetes** y **tiempo de respuesta**.

🔹 Esta prueba es complementaria a la certificación física y permite detectar problemas aguas arriba del enlace (configuración del switch, VLAN, servidor DHCP caído, etc.).

🔹 En el laboratorio se verificó la conectividad con el PC de trabajo, que obtuvo IP **10.0.0.186** (ver detalles en `pruebas.md`).

---

### 4️⃣ 📍 Localizador de Fallas (TDR)

El **TDR (Time Domain Reflectometry)** es la función más potente para el diagnóstico de fallas físicas. Emite un pulso eléctrico por el cable y analiza el **reflejo** que retorna:

- 📏 Calcula la **longitud** total del cable.
- 🎯 Identifica **a cuántos metros exactos** se encuentra un **corte (Open)** o **cortocircuito (Short)**.
- 🔁 Detecta **reflexiones anómalas** que indican malos empalmes o conectores defectuosos.
- 📊 Mide el **skew** (diferencia de longitud eléctrica entre pares del mismo cable).

🔹 **Aplicación real en este laboratorio:**
En el puerto **P017**, el TDR reportó una longitud de **21.0 m en los pines 1-2 con status "Open"**, mientras que los demás pares medían 22.0 – 22.3 m. Esta diferencia permite al técnico ir directamente a los **21 metros del recorrido del cable** para localizar el punto exacto de la falla, sin tener que abrir todo el tendido.

---

## 🎛️ Otras Funciones Relevantes

| Función | Descripción |
|---|---|
| 🆔 **Port Discovery** | Identifica el número de puerto del switch al que está conectado el cable |
| 🔦 **Tone Generator** | Genera tono audible para localizar cables con un probe |
| 🔄 **Active Remote ID** | Permite identificar hasta 20 puntos distintos con los remotos numerados |
| 💾 **Memory / Save** | Guarda los resultados en memoria interna para exportar como PDF |

---

## 📌 Recomendaciones de Uso

- 🔋 Verificar nivel de batería antes de iniciar una jornada de certificación.
- 🧹 Limpiar los conectores RJ-45 de la unidad principal y remota para evitar falsas fallas.
- 📝 Configurar correctamente el ID del punto **antes** de cada test para que el reporte PDF sea trazable.
- 🔌 Desconectar el enlace del switch antes de ejecutar pruebas de wiremap/TDR (evita falsos positivos por actividad de red).
- ⚠️ Para pruebas PoE, **sí** mantener conectado al switch que entrega alimentación.

---

## 📎 Referencias

- Manual de usuario **Net Chaser TNC950** – Platinum Tools.
- ANSI/TIA-1152 – *Requirements for Field Test Instruments for Balanced Twisted-Pair Cabling*.
- IEEE 802.3af / 802.3at – *PoE y PoE+*.
