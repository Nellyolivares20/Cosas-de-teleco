# 🌐✨ Certificación de Redes UTP — Laboratorio INACAP

> 🎓 **Carrera:** Ingeniería en Telecomunicaciones  
> 🏫 **Institución:** INACAP La Serena  
> 👨‍🏫 **Docente:** Daniel Ruz Moreno  
> 🧑‍💻 **Ingeniero de Pruebas:** A-315  
> 📅 **Fecha:** 09 de Abril, 2026  

---

## 🗺️ ¿De qué trata este repositorio?

Este proyecto documenta la **certificación física y verificación lógica** de puntos de red UTP realizados en terreno. Usamos el certificador 🔬 **Net Chaser TNC950** para validar el cumplimiento del estándar **TIA/EIA-568**, y la herramienta ⚡ **iPerf3** para medir el rendimiento real del enlace.

---

## 📁 Estructura del Repositorio

```
📦 certificacion-redes-utp/
 ┣ 📄 README.md       → Portada y descripción general
 ┣ 📄 parametros.md   → Definiciones técnicas de certificación
 ┣ 📄 equipo.md       → Uso y specs del Net Chaser TNC950
 ┗ 📄 pruebas.md      → Registro completo de testeos
```

---

## 🧪 Resultados de Certificación

### 🔌 Puertos P014 — P019 | Net Chaser TNC950

| 🔖 Puerto | 📦 Tipo | 📏 Longitud | 📶 SNR | 🔁 BERT | 📞 VoIP | 🚀 1 Gbps | 🏁 Resultado |
|-----------|---------|------------|-------|---------|---------|-----------|-------------|
| P 014 | CAT5E | 18.2 m | 27.2 | 0 errores | ✅ | ✅ | ✅ **PASS** |
| P 015 | CAT5E | 19.9 m | 28.3 | 0 errores | ✅ | ✅ | ✅ **PASS** |
| P 016 | CAT5E | 18.3 m | 28.3 | 0 errores | ✅ | ✅ | ✅ **PASS** |
| P 017 | CAT5E | 21.0 m | --- | --- | ❌ | ❌ | ❌ **FAIL** |
| P 018 | CAT5E | 19.3 m | 28.3 | 0 errores | ✅ | ✅ | ✅ **PASS** |
| P 019 | CAT5E | 19.7 m | 27.2 | ❌ FAIL | ❌ | ❌ | ❌ **FAIL** |

#### ⚠️ Análisis de Fallas

> 🔴 **P 017** — Wiremap abierto en pines 1 y 2. Sin conectividad en esos pares. Causa probable: mal crimpado o Jack defectuoso.

> 🔴 **P 019** — BERT con errores de transmisión. Causa probable: interferencia en el par, exceso de destrenzado o daño físico en el cable.

---

## ⚡ Prueba PoE

```
📡 Modo:              A
📍 Pines activos:     1-2 y 3-6
📋 Estándar:          IEEE 802.3af
💡 Potencia:          12.95 W
🔋 Voltaje mín carga: 50.1 V
🔋 Voltaje máx carga: 48.9 V
```
> ✅ PoE detectado y funcionando correctamente dentro del estándar 802.3af.

---

## 🚀 Prueba de Rendimiento — iPerf3

```
🖥️  Servidor:    10.0.0.185 : 5201
💻  Cliente:     10.0.0.186
⏱️  Duración:    ~10 segundos
📦  Transferido: 113 MB
📊  Bandwidth:   ~94 Mbits/sec ✅
```

> 💡 El enlace opera dentro del rango esperado para **Fast Ethernet (100 Mbps)** sobre cableado CAT5E certificado.

---

## 🛠️ Herramientas Utilizadas

| 🔧 Herramienta | 📌 Función |
|----------------|-----------|
| 🔬 Net Chaser TNC950 | Certificación física del cableado UTP |
| ⚡ iPerf3 | Medición de ancho de banda TCP |
| 💻 Windows CMD (ipconfig) | Verificación de configuración IP |

---

## 🗃️ Archivos de Evidencia

| 📄 Archivo | 📝 Descripción |
|-----------|---------------|
| `PRUEBA_PUERT14-19.pdf` | Reporte Net Chaser — puertos P014 a P019 |
| `PRUEBA_POE.pdf` | Reporte prueba PoE |
| `IPERF_SERVIDOR.png` | Captura servidor iPerf3 (10.0.0.185) |
| `IPERF_CLEINTE_.jpeg` | Captura IP cliente (10.0.0.186) |

---

## 📐 Estándares Aplicados

- 📘 **TIA/EIA-568-C.2** — Cableado estructurado en edificios comerciales
- ⚡ **IEEE 802.3af** — Power over Ethernet (PoE)
- 🔵 **T568B** — Estándar de terminación de pares utilizado en el laboratorio

---

<div align="center">

🎓 *Evaluación 1 — Certificación de Cableado Estructurado*  
🏫 *INACAP La Serena — Ingeniería en Telecomunicaciones*

</div>
