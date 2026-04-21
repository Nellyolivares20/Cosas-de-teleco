# 🧪 Registro de Pruebas — Certificación P014 a P019

Este documento contiene el **registro completo** de todas las pruebas de certificación ejecutadas en el rack del laboratorio de INACAP La Serena el día **09-04-2026**, con el certificador **Net Chaser TNC950** (Test Engineer: A-315).

Se incluyen:

1. 📊 Certificación de los **6 puntos de red** (P014–P019).
2. 🔍 Análisis técnico detallado de las **fallas detectadas**.
3. ⚡ Prueba **PoE** sobre uno de los puntos.
4. 🚀 Prueba de **rendimiento real con iPerf3** sobre un enlace aprobado.

---

## 📋 1. Resumen de Certificación (P014 – P019)

| Punto | Categoría | Tipo de cable | Longitud | SNR (mín) | BERT | VoIP | 1 Gbps | Resultado |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **P014** | CAT5E | UTP | 18.2 m | 27.2 dB | 0 | ✅ | ✅ | 🟢 **PASS** |
| **P015** | CAT5E | UTP | 19.9 m | 28.3 dB | 0 | ✅ | ✅ | 🟢 **PASS** |
| **P016** | CAT5E | UTP | 18.3 m | 28.3 dB | 0 | ✅ | ✅ | 🟢 **PASS** |
| **P017** | CAT5E | UTP | 21.0 m | — | — | ❌ | ❌ | 🔴 **FAIL** |
| **P018** | CAT5E | UTP | 19.3 m | 28.3 dB | 0 | ✅ | ✅ | 🟢 **PASS** |
| **P019** | CAT5E | UTP | 19.7 m | 27.2 dB | **FAIL** | ❌ | ❌ | 🔴 **FAIL** |

### 📈 Indicadores globales

| Métrica | Valor |
|---|---:|
| 🔢 Total puntos certificados | 6 |
| ✅ PASS | 4 |
| ❌ FAIL | 2 |
| 📊 Tasa de éxito | **66.7 %** |

---

## 📋 2. Detalle por Punto de Red

### 🟢 P014 — PASS
| Parámetro | Valor |
|---|---:|
| Tipo | UTP CAT5E |
| Longitud | 18.2 m |
| Skew | 0.0 ns (en todos los pares) |
| SNR mínimo | 27.2 dB |
| BERT | 0 errores |
| VoIP / 1 Gbps | ✅ / ✅ |
| pF/m | 49.2 |

---

### 🟢 P015 — PASS
| Parámetro | Valor |
|---|---:|
| Tipo | UTP CAT5E |
| Longitud | 19.9 m |
| Skew máximo | 8.0 ns (pares 1-2 y 3-6) |
| SNR mínimo | 28.3 dB |
| BERT | 0 errores |
| VoIP / 1 Gbps | ✅ / ✅ |
| pF/m | 49.2 |

> 📝 Presenta un ligero skew de 8 ns, dentro de los límites aceptables para CAT5E (< 50 ns).

---

### 🟢 P016 — PASS
| Parámetro | Valor |
|---|---:|
| Tipo | UTP CAT5E |
| Longitud | 18.3 m |
| Skew | 0.0 ns |
| SNR mínimo | 28.3 dB |
| BERT | 0 errores |
| VoIP / 1 Gbps | ✅ / ✅ |
| pF/m | 49.2 |

---

### 🔴 P017 — FAIL (Wiremap Open)
| Parámetro | Valor |
|---|---:|
| Tipo | UTP CAT5E |
| Longitud par 1-2 | 21.0 m — **OPEN** ❌ |
| Longitud par 3-6 | 22.3 m |
| Longitud par 4-5 | 22.0 m |
| Longitud par 7-8 | 21.4 m |
| BERT / SNR | No medible |
| VoIP / 1 Gbps | ❌ / ❌ |

> 🚨 **Falla crítica:** El certificador marca los pines **1 y 2 con una X roja** (circuito abierto). El resto de pares presenta continuidad.

---

### 🟢 P018 — PASS
| Parámetro | Valor |
|---|---:|
| Tipo | UTP CAT5E |
| Longitud | 19.3 m |
| Skew | 0.0 ns |
| SNR mínimo | 28.3 dB |
| BERT | 0 errores |
| VoIP / 1 Gbps | ✅ / ✅ |
| pF/m | 49.2 |

---

### 🔴 P019 — FAIL (BERT)
| Parámetro | Valor |
|---|---:|
| Tipo | UTP CAT5E |
| Longitud | 19.7 m |
| Wiremap | ✅ Correcto |
| Skew | 0.0 ns |
| SNR mínimo | 27.2 dB |
| SNR máximo | 51.1 dB (par 7-8) |
| BERT | ❌ **FAIL** |
| VoIP / 1 Gbps | ❌ / ❌ |

> ⚠️ A pesar de tener continuidad correcta y longitud válida, el enlace **no superó el test de errores de bit**, lo que impide certificarlo para 1 Gbps.

---

## 🔍 3. Análisis Técnico de las Fallas

### 🔴 Análisis de la falla en P017 — Wiremap Open en par 1-2

**Síntoma:** El certificador reporta **Open** (circuito abierto) en los pines **1 y 2** (par naranja según T568B). Los demás pares tienen continuidad.

**Posibles causas:**

1. 🧵 **Destrenzado excesivo** de los hilos del par naranja al momento del ponchado, con un hilo que no quedó presionado correctamente en la cuchilla del jack.
2. 🔧 **Hilo roto o quebrado** dentro del jack RJ-45 o dentro del patch panel por un exceso de tensión durante el tendido.
3. ✂️ **Corte físico** del cable en algún tramo del recorrido horizontal (por una grapa, un tornillo o un aplastamiento).
4. 🔌 **Mal ponchado del conector** en uno de los dos extremos → el hilo naranja o naranja/blanco no hace contacto eléctrico.

**Acción recomendada:**

- ✅ Re-ponchar ambos extremos del cable (patch panel y roseta) respetando la norma T568B.
- ✅ Verificar con TDR la distancia exacta del corte (el equipo reportó 21.0 m).
- ✅ Si tras re-ponchar la falla persiste, reemplazar el cable horizontal completo.

---

### 🔴 Análisis de la falla en P019 — BERT FAIL

**Síntoma:** Wiremap correcto, longitud válida (19.7 m), pero **falla el test de errores de bit (BERT)**. Al observar los valores de SNR por par, se detecta una diferencia notoria:

| Par | SNR |
|---|---:|
| 1-2 | 27.4 dB |
| 3-6 | 32.1 dB |
| 4-5 | **27.2 dB** |
| 7-8 | 51.1 dB |

**Posibles causas:**

1. 🌀 **Paradiafonía (NEXT) elevada** entre los pares 1-2 y 4-5 → el margen de SNR es el más bajo.
2. 🧵 **Exceso de destrenzado** (> 13 mm) en uno de los extremos durante el ponchado.
3. 🔧 **Uso de un jack de categoría inferior** al cable (por ejemplo, jack CAT5 en un cable CAT5e).
4. 🌡️ **Interferencia electromagnética externa** cercana al recorrido del cable (luminarias fluorescentes, cables de poder paralelos).
5. 🔁 **Radios de curvatura muy cerrados** en el tendido, que deforman el par trenzado.

**Acción recomendada:**

- ✅ Re-ponchar ambos extremos respetando el destrenzado máximo de 13 mm.
- ✅ Verificar que los conectores sean CAT5e o superior.
- ✅ Revisar el recorrido físico del cable en busca de cruces con cables eléctricos o curvaturas forzadas.
- ✅ Repetir el Auto-Test tras las correcciones.

---

## ⚡ 4. Prueba PoE

La prueba se ejecutó sobre uno de los puertos del switch (configurado con PoE) para validar su capacidad de alimentación.

| Parámetro | Valor |
|---|---:|
| 🕒 Fecha / Hora | 09-04-2026 / 14:04 |
| 🔌 Modo | **A** |
| 🧵 Pares con PoE detectado | **1-2 y 3-6** |
| 🏷️ Estándar | **IEEE 802.3af** |
| ⚡ Potencia nominal | **12.95 W** |
| 🔋 Voltaje @ carga mínima | **50.1 V** |
| 🔋 Voltaje @ carga máxima | **48.9 V** |
| 📉 Caída de voltaje bajo carga | 1.2 V |

### ✅ Interpretación

- 🟢 La detección en **Mode A (pines 1-2 y 3-6)** es propia de switches PoE que inyectan la alimentación por los mismos pares que transportan datos (10/100BASE-T).
- 🟢 El voltaje se mantiene dentro del rango **44–57 V DC** definido por **IEEE 802.3af**.
- 🟢 La caída de solo 1.2 V bajo carga máxima indica **buena calidad del cable y del conector** (baja resistencia de lazo).
- 🟢 Los 12.95 W disponibles son suficientes para alimentar **teléfonos VoIP, APs Wi-Fi de bajo consumo y cámaras IP básicas**.

---

## 🚀 5. Prueba de Rendimiento — iPerf3

Para validar el rendimiento real del enlace (más allá de la capa física), se ejecutó una prueba de ancho de banda entre dos equipos usando **iPerf3** sobre un punto de red aprobado.

### 🖥️ Topología de la prueba

```
┌───────────────────┐        Switch        ┌───────────────────┐
│  PC Cliente       │                      │  PC Servidor      │
│  IP: 10.0.0.186   │◄──── LAN 10.0.0/24 ──►│  IP: 10.0.0.185  │
│  (ipconfig)       │        PoE           │  iperf3 -s        │
└───────────────────┘                      └───────────────────┘
                              ▲
                              │ Puerto 5201 TCP
```

### 🌐 Configuración de red del cliente (ipconfig)

| Parámetro | Valor |
|---|---:|
| 🔌 Adaptador | Ethernet |
| 🌐 Dirección IPv4 | **10.0.0.186** |
| 🎭 Máscara de subred | 255.255.255.0 |
| 🚪 Puerta de enlace | 10.0.0.1 |
| 🔗 IPv6 local | fe80::c293:81ac:bb06:9150%18 |

### 🖥️ Configuración del servidor iPerf3

| Parámetro | Valor |
|---|---:|
| 💻 Comando ejecutado | `iperf3 -s` |
| 🌐 IP del servidor | **10.0.0.185** |
| 🔌 Puerto de escucha | **5201 TCP** |
| 📥 Conexión aceptada desde | 10.0.0.186 : 17340 |

### 📊 Resultados por segundo (servidor iPerf3)

| Intervalo (s) | Transferencia | Ancho de banda |
|:---:|:---:|:---:|
| 0.00 – 1.00 | 10.8 MBytes | 90.3 Mbits/s |
| 1.00 – 2.00 | 11.3 MBytes | 94.4 Mbits/s |
| 2.00 – 3.00 | 11.2 MBytes | 94.4 Mbits/s |
| 3.00 – 4.00 | 11.3 MBytes | 94.4 Mbits/s |
| 4.00 – 5.00 | 11.2 MBytes | 94.4 Mbits/s |
| 5.00 – 6.00 | 11.2 MBytes | 94.4 Mbits/s |
| 6.00 – 7.00 | 11.2 MBytes | 94.4 Mbits/s |
| 7.00 – 8.00 | 11.3 MBytes | 94.4 Mbits/s |
| 8.00 – 9.00 | 11.3 MBytes | 94.4 Mbits/s |
| 9.00 – 10.00 | 11.3 MBytes | 94.4 Mbits/s |
| 10.00 – 10.05 | 590 KBytes | 94.1 Mbits/s |

### 📈 Resumen final de la prueba iPerf3

| Rol | Transferencia total | Ancho de banda promedio |
|:---:|:---:|:---:|
| 📥 **Receiver (servidor)** | **113 MBytes** | **94.0 Mbits/s** |
| ⏱️ Duración total | 10.05 segundos | — |

### ✅ Análisis del resultado

- 🟢 El enlace se **estabiliza rápidamente** en ~94.4 Mbit/s desde el segundo 1.
- 🟡 El throughput obtenido (94 Mbit/s) indica que el enlace está operando a **100 Mbps** (Fast Ethernet), no a 1 Gbps. Esto es consistente con:
  - La categoría CAT5E del cable detectada por el certificador (soporta hasta 1 Gbps, pero el switch puede estar negociando a 100 Mbps).
  - La eficiencia típica de TCP sobre 100BASE-TX (≈ 94 % del ancho de banda bruto).
- 🟢 **Cero retransmisiones visibles** y ancho de banda **constante**, lo que confirma que **la capa física está sana** y que **no hay pérdida de paquetes** en este enlace.
- 📝 Para alcanzar 1 Gbps reales, deberían verificarse: la velocidad negociada en el switch, la capacidad de los NIC de ambos equipos y usar cableado/jacks totalmente certificados para Gigabit.

---

## 🎯 6. Conclusiones

1. ✅ **4 de 6 puntos** fueron certificados exitosamente, cumpliendo con todos los parámetros de TIA/EIA-568 para CAT5E.
2. 🔴 **P017** requiere re-ponchado del par naranja (Open en pines 1-2).
3. 🔴 **P019** requiere revisión del NEXT y re-ponchado por exceso de destrenzado (BERT FAIL).
4. ⚡ La infraestructura **PoE** cumple plenamente con el estándar **802.3af**.
5. 🚀 La prueba **iPerf3** confirma que el enlace aprobado entrega throughput estable a **94 Mbit/s** sobre Fast Ethernet, con cero pérdida.
6. 📝 Todos los resultados fueron documentados en este repositorio Git cumpliendo la rúbrica de evaluación.
