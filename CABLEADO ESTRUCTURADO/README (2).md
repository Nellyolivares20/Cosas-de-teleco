# 🌐 Certificación y Documentación de Redes UTP

[![Status](https://img.shields.io/badge/Estado-Completado-brightgreen)]()
[![Norma](https://img.shields.io/badge/Norma-TIA%2FEIA--568-blue)]()
[![Categoría](https://img.shields.io/badge/Categor%C3%ADa-CAT5E-orange)]()
[![Equipo](https://img.shields.io/badge/Certificador-Net%20Chaser%20TNC950-purple)]()

---

## 📋 Descripción General

Este repositorio documenta la **Actividad de Laboratorio de Certificación y Documentación de Redes UTP** desarrollada en el marco de la asignatura **Certificación de Cableado Estructurado** de la carrera de **Ingeniería en Telecomunicaciones** en INACAP Sede La Serena.

El objetivo del laboratorio es ejecutar pruebas de certificación sobre puntos de red físicos del laboratorio utilizando el certificador **Net Chaser TNC950**, verificar el cumplimiento de estándares **TIA/EIA-568**, y documentar profesionalmente el proceso mediante un repositorio Git con archivos Markdown.

Se certificaron **6 puntos de red** (P014–P019), se realizó una prueba **PoE** y se midió el **rendimiento real del enlace** mediante **iPerf3** entre cliente y servidor.

---

## 👥 Integrantes del Equipo

| Campo | Valor |
|---|---|
| 🎓 **Carrera** | Ingeniería en Telecomunicaciones |
| 🏫 **Sede** | INACAP La Serena |
| 📘 **Asignatura** | Certificación de Cableado Estructurado |
| 👨‍🏫 **Docente** | Daniel Ruz Moreno |
| 🆔 **Test Engineer** | A-315 |
| 📅 **Fecha de ensayo** | 09-04-2026 |

---

## 📏 Estándares y Normativas Aplicadas

| Estándar | Aplicación |
|---|---|
| 📐 **ANSI/TIA-568** | Especificaciones generales de cableado estructurado comercial |
| 🔌 **TIA/EIA-568-B** | Esquema de pinout utilizado en los patch cords certificados |
| 📶 **IEEE 802.3af** | Estándar PoE detectado en la prueba (hasta 12.95 W) |
| 🧵 **ISO/IEC 11801** | Referencia internacional de clases y categorías |
| 🔧 **Categoría CAT5E** | Categoría de cableado detectada por el certificador |

---

## 🗂️ Estructura del Repositorio

```
📦 certificacion-redes-utp/
├── 📄 README.md         → Portada y descripción general (este archivo)
├── 📄 parametros.md     → Definiciones técnicas (Wiremap, NEXT, Atenuación, Longitud)
├── 📄 equipo.md         → Manual de uso del certificador Net Chaser TNC950
└── 📄 pruebas.md        → Registro de testeos, análisis de fallas y rendimiento iPerf3
```

---

## 📊 Resumen Visual de Resultados

### ✅ Certificación de Puntos de Red (P014–P019)

| Puerto | Resultado | Motivo |
|:---:|:---:|:---|
| 🟢 **P014** | PASS | Enlace correcto, 1 Gbps, VoIP OK |
| 🟢 **P015** | PASS | Enlace correcto, 1 Gbps, VoIP OK |
| 🟢 **P016** | PASS | Enlace correcto, 1 Gbps, VoIP OK |
| 🔴 **P017** | FAIL | Wiremap Open en pares 1-2 |
| 🟢 **P018** | PASS | Enlace correcto, 1 Gbps, VoIP OK |
| 🔴 **P019** | FAIL | BERT: errores de bit detectados |

### 📈 Estadística Global

| Métrica | Valor |
|---|---:|
| 🔢 Total de puntos certificados | 6 |
| ✅ Puntos aprobados (PASS) | 4 |
| ❌ Puntos rechazados (FAIL) | 2 |
| 📊 Tasa de éxito | **66.7 %** |

### ⚡ Prueba PoE

| Parámetro | Valor |
|---|---:|
| 🔌 Estándar | IEEE 802.3af |
| ⚡ Potencia | 12.95 W |
| 🔋 Voltaje (mín. carga) | 50.1 V |
| 🔋 Voltaje (máx. carga) | 48.9 V |

### 🚀 Rendimiento iPerf3

| Parámetro | Valor |
|---|---:|
| 🖥️ Servidor | 10.0.0.185 |
| 💻 Cliente | 10.0.0.186 |
| ⏱️ Duración | 10 s |
| 📦 Transferencia | 113 MBytes |
| 📶 Ancho de banda | **94.0 Mbits/sec** |

---

## 🎯 Objetivos Cumplidos

- ✔️ **Técnico** → Ejecución de pruebas de certificación en 6 puntos físicos del rack.
- ✔️ **Normativo** → Verificación del cumplimiento de TIA/EIA-568 e identificación de categorías.
- ✔️ **Gestión** → Documentación profesional del proceso mediante repositorio Git y Markdown.

---

## 📎 Referencias

- Manual Net Chaser TNC950 – *Platinum Tools*
- ANSI/TIA-568-C.2 – *Balanced Twisted-Pair Telecommunications Cabling*
- IEEE 802.3af-2003 – *Power over Ethernet*
- iPerf3 Documentation – *https://iperf.fr*
