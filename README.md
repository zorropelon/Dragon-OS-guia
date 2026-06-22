# 📡 DragonOS — Guía de Herramientas por Categoría

---
Nota de autor: fue hecho todo con ai. Claude ai MODELO SONNET 4.6 max. Lo hice porque no encontr[e buena info en la web entonces empece a preguntara ai, pero mejor de una vez que fuera algo productivo.
---

> **A community reference guide for DragonOS SDR tools, written in Spanish for the Latin American technical community.**

[![DragonOS](https://img.shields.io/badge/DragonOS-Noble%2024.04-cyan?style=flat-square)](https://sourceforge.net/projects/dragonos-focal/)
[![Licencia](https://img.shields.io/badge/Licencia-CC%20BY%204.0-blue?style=flat-square)](https://creativecommons.org/licenses/by/4.0/)
[![Estado](https://img.shields.io/badge/Estado-En%20progreso-orange?style=flat-square)](#contribuir)

---

## ¿Qué es esto?

Este repositorio contiene una guía de referencia para las herramientas incluidas en [DragonOS](https://sourceforge.net/projects/dragonos-focal/) — la distribución Linux especializada en Software Defined Radio (SDR).

**El problema que resuelve:** DragonOS incluye más de 100 herramientas preinstaladas, pero su documentación oficial es escasa. Esta guía explica **qué hace cada herramienta, para qué sirve, qué hardware necesitás, y cuáles son sus posibilidades reales** — sin ser un tutorial paso a paso, sino el mapa mental que falta.

Está escrita en **español** y orientada a usuarios técnicos de **Latinoamérica** (frecuencias, regulaciones y contexto regional incluidos).

---

## 📋 Contenido

| # | Categoría | Herramientas cubiertas |
|---|-----------|----------------------|
| 1 | [Monitoreo aeronáutico](#) | dump1090, acarsdec, dumpvdl2, dumpHFDL, Jaero, iridium-toolkit |
| 2 | [Redes celulares / telecom](#) | gr-gsm, IMSI-catcher, srsRAN, LTE-Cell-Scanner, OsmocomBB, YateBTS |
| 3 | [WiFi y redes inalámbricas](#) | Kismet, Sparrow-WiFi, Aircrack-ng, WHAD-client |
| 4 | [Satélites](#) | SatDump, NOAA/METEOR, GNSS-SDR, GR-Iridium |
| 5 | [Trunking y radio terrestre](#) | SDRTrunk, Trunk-Recorder, DSD/DSD-FME, OP25, QRadioLink |
| 6 | [Análisis de señales digitales](#) | URH, Inspectrum, SigDigger, GNU Radio, Signal Server |
| 7 | [Transmisión y ofensivo](#) | HackTV, GPS-Simulator, hackrf_transfer/sweep, Portapack Mayhem |
| 8 | [Decodificadores misceláneos](#) | APRS/Direwolf, AIS, POCSAG/FLEX, SSTV, Radiosondas, NRSC5, WSJT-X/JS8Call |

Además incluye:
- **Apéndice A:** Hardware compatible con rangos de frecuencia y casos de uso
- **Apéndice B:** Frecuencias de referencia rápida (con frecuencias específicas para Costa Rica)
- **Apéndice C:** Glosario de siglas (55 términos)

---

## 📁 Archivos

```
dragonos_guia.md      ← Fuente en Markdown (editar aquí)
dragonos_guia.html    ← Versión HTML con sidebar navegable y diseño dark
README.md             ← Este archivo
```

### Ver el HTML

Podés ver la versión HTML directamente en GitHub Pages (si está habilitado) o descargando `dragonos_guia.html` y abriéndolo en cualquier browser. Es un archivo autocontenido — no necesita servidor.

El HTML incluye:
- Sidebar fijo con tabla de contenidos navegable
- Parágrafos especiales marcados: hardware (🟢), notas legales (🔴), frecuencias locales (🟠)
- Dark theme optimizado para pantalla
- Responsive / mobile-friendly
- Print-friendly (sidebar oculto al imprimir)

---

## 🤝 Contribuir

Esta guía es un **work in progress** — igual que el wiki oficial de DragonOS. Si encontrás:

- **Información incorrecta o desactualizada** → abrí un Issue o PR
- **Una herramienta no cubierta** → completá la sección o abrí un Issue describiendo la herramienta
- **Error de redacción o traducción** → PR bienvenido
- **Frecuencias incorrectas para tu país** → abrí un Issue con las frecuencias correctas para tu región

### Cómo editar

1. Forkear el repositorio
2. Editar `dragonos_guia.md` (el markdown es la fuente de verdad)
3. Si querés regenerar el HTML, correr el script de conversión (ver abajo)
4. PR con tus cambios

### Regenerar el HTML

```bash
pip install markdown
python3 -c "
import markdown, re

with open('dragonos_guia.md', 'r', encoding='utf-8') as f:
    md_content = f.read()

# Ver script completo en el repositorio
"
```

> El script completo de generación del HTML está documentado en el historial de commits.

### ¿Qué falta?

- [ ] Más contexto regional para países fuera de Costa Rica (México, Argentina, Colombia, etc.)
- [ ] Ejemplos de comandos reales para cada herramienta
- [ ] Capturas de pantalla de cada herramienta en acción
- [ ] Sección de antenas (tipos, construcción, rangos de frecuencia)
- [ ] Sección de hardware SDR con comparativa de precio/rendimiento actualizada
- [ ] Traducción al inglés (para integración con el wiki oficial de DragonOS)

---

## 🔗 Recursos oficiales de DragonOS

| Recurso | Enlace |
|---------|--------|
| DragonOS Noble (x86_64) | [sourceforge.net/projects/dragonos-focal](https://sourceforge.net/projects/dragonos-focal/) |
| DragonOS Pi64 | [sourceforge.net/projects/dragonos-pi64](https://sourceforge.net/projects/dragonos-pi64/) |
| Wiki oficial (en progreso) | [sourceforge.net/p/dragonos-focal/wiki](https://sourceforge.net/p/dragonos-focal/wiki/Home/) |
| Canal YouTube de Cemaxecuter | [@cemaxecuter](https://www.youtube.com/@cemaxecuter) |
| Twitter/X | [@cemaxecuter](https://x.com/cemaxecuter) |
| Patreon | [patreon.com/cemaxecuter](https://www.patreon.com/cemaxecuter) |
| WarDragon hardware | [cemaxecuter.com](https://cemaxecuter.com) |

---

## ⚠️ Aviso legal

Esta guía es un **recurso educativo**. Algunas herramientas descritas aquí pueden tener implicaciones legales dependiendo de cómo y dónde se usen.

- **Recibir señales de radio** es legal en la mayoría de jurisdicciones cuando las transmisiones son de carácter público y no encriptadas.
- **Transmitir señales de radio** requiere licencia de radioaficionado o autorización específica en prácticamente todos los países.
- **Interceptar comunicaciones privadas** es ilegal en la mayoría de jurisdicciones.
- Las leyes varían significativamente por país. Siempre consultá la regulación local (SUTEL en Costa Rica, FCC en EE.UU., ANATEL en Brasil, etc.).

Los autores de esta guía no se hacen responsables del uso indebido de las herramientas descritas.

---

## 📜 Licencia

Este trabajo está licenciado bajo [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

Podés compartir y adaptar el material para cualquier propósito — incluso comercial — siempre que des crédito apropiado, indiques si realizaste cambios, y enlaces a la licencia. No estás obligado a distribuir derivados bajo la misma licencia.

Para usar el contenido simplemente incluí:
> *"Basado en [DragonOS — Guía de Herramientas](URL_DEL_REPO) por [tu nombre], licenciado bajo CC BY 4.0"*

---

## 🙏 Créditos

- **[Cemaxecuter](https://x.com/cemaxecuter)** — creador y mantenedor de DragonOS
- **[RTL-SDR Blog](https://rtl-sdr.com)** — recurso de referencia para la comunidad SDR
- **[RadioReference](https://radioreference.com)** — base de datos de sistemas de radio
- **[airframes.io](https://airframes.io)** — documentación de protocolos aeronáuticos
- **[SatDump](https://satdump.org)** — la mejor herramienta de satélites open source
- Todos los desarrolladores de las herramientas open source documentadas aquí

---

*Escrito con ❤️ desde Costa Rica 🇨🇷 para la comunidad SDR latinoamericana*
