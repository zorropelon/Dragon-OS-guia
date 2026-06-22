# DragonOS — Guía de Herramientas por Categoría

> Guía de referencia para entender qué hace cada herramienta incluida en DragonOS, para qué sirve, qué hardware necesitás, y cuáles son sus posibilidades reales. No es un tutorial paso a paso — es el mapa mental que falta.

**Versión de referencia:** DragonOS Noble (24.04)  
**Última actualización:** Junio 2025  
**Hardware de referencia:** RTL-SDR v3/v4, HackRF One, Airspy R2/Mini/HF+, LimeSDR

---

## Tabla de contenidos

1. [Monitoreo aeronáutico](#1-monitoreo-aeronáutico)
2. [Redes celulares / telecom](#2-redes-celulares--telecom)
3. [WiFi y redes inalámbricas](#3-wifi-y-redes-inalámbricas)
4. [Satélites](#4-satélites)
5. [Trunking y radiocomunicaciones terrestres](#5-trunking-y-radiocomunicaciones-terrestres)
6. [Análisis de señales digitales](#6-análisis-de-señales-digitales)
7. [Transmisión y herramientas ofensivas](#7-transmisión-y-herramientas-ofensivas)
8. [Decodificadores misceláneos](#8-decodificadores-misceláneos)

---

## 1. Monitoreo aeronáutico

> *Los aviones transmiten datos constantemente en múltiples protocolos. Con SDR podés recibir posición, mensajes de texto, voz, telemetría — sin necesitar nada más que el dongle.*

### 1.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **SDR++** | Visualización de espectro general, plugin ADS-B nativo |
| **SDRAngel** | Demoduladores de voz aeronáutica AM integrados |
| **GQRX** | Recepción de voz AM de torre/piloto, monitoreo de banda |
| **OpenWebRX+** | Acceso remoto vía browser a feeds aeronáuticos |

### 1.2 ADS-B — dump1090 / view1090

**¿Qué es?** ADS-B (Automatic Dependent Surveillance–Broadcast) es el sistema por el cual cada avión comercial transmite continuamente su posición GPS, altitud, velocidad, rumbo y número de vuelo en **1090 MHz**. No es un radar — el avión te dice él mismo dónde está. `dump1090` es el decodificador estándar que recibe esas transmisiones Mode S y las convierte en datos utilizables.

**¿Qué podés hacer con él?**
- Crear tu propio radar de tráfico aéreo local en tiempo real
- Ver en un mapa web (localhost:8080) todos los aviones en un radio de ~200–400 km según la antena
- Alimentar plataformas comunitarias como FlightAware, ADSBExchange o AirSquitter
- Recibir posición en tierra (aviones taxiando), no solo en vuelo
- Exportar datos via TCP (puerto 30003) a otras aplicaciones (Virtual Radar Server, tar1090, etc.)

**Hardware mínimo:** RTL-SDR + antena 1090 MHz (la simple dipolo incluida con el dongle funciona)  
**Hardware ideal:** RTL-SDR v3 + antena colineal exterior elevada. Con buena antena se pueden ver aviones a 400+ km.

**Variantes en DragonOS:**
- `dump1090` original (Salvatore Sanfilippo) — la base
- `dump1090-mutability` — fork con mejor decodificación, paquete Debian nativo
- `dump1090-fa` (FlightAware) — versión con alimentación directa a FlightAware
- `view1090` — cliente ligero para visualizar datos de un dump1090 corriendo en otro host

**Modo de operación:** Solo CLI, sin GUI propia. El mapa web es un servidor HTTP embebido. Para visualización avanzada se combina con `tar1090` (mapa moderno) o Virtual Radar Server.

**Puerto en DragonOS:** `/usr/local/bin/dump1090` o `/usr/src/` según versión.  
**GitHub:** `github.com/antirez/dump1090` (original) / `github.com/flightaware/dump1090` (FA)

---

### 1.3 ACARS — acarsdec

**¿Qué es?** ACARS (Aircraft Communications Addressing and Reporting System) es un sistema de mensajería de texto entre aviones y tierra que existe desde los años 70 y sigue activo. Funciona en VHF (~129–136 MHz). Los mensajes incluyen: clearances de ATC, reportes meteorológicos, estado de sistemas del avión, información de gate, estimados de llegada, y mensajes operacionales de aerolínea. `acarsdec` es el decodificador multicanal para SDR.

**¿Qué podés hacer con él?**
- Leer mensajes de texto reales entre aerolíneas y sus aviones en vuelo
- Monitorear hasta 8 frecuencias simultáneamente con un solo RTL-SDR
- Ver clearances CPDLC (Controller-Pilot Data Link Communications) y reportes ADS-C
- Alimentar bases de datos locales (SQLite via `acarsserv`) o plataformas como airframes.io
- Exportar en JSON para procesamiento propio (Node-RED, scripts, dashboards)

**Hardware mínimo:** RTL-SDR + antena VHF. Funciona indoor aunque con alcance reducido.  
**Hardware ideal:** Airspy R2 o SDRplay — mayor ancho de banda permite cubrir más frecuencias.

**Frecuencias principales (región IARU R2 / Latinoamérica):**
`129.125` / `130.025` / `130.425` / `131.125` / `131.525` / `131.550` / `131.725` / `131.825` MHz

**Particularidad clave:** Todas las frecuencias a monitorear deben estar dentro de un rango de 2 MHz (limitación del RTL-SDR). Para cubrir más, se necesitan múltiples dongles o hardware con mayor ancho de banda.

**Modos de salida:** texto plano, JSON, PlanePlotter UDP, SQLite, MQTT — lo que lo hace fácil de integrar.

**Puerto en DragonOS:** `/usr/src/acarsdec/`  
**GitHub:** `github.com/TLeconte/acarsdec`

---

### 1.4 VDL Mode 2 — dumpvdl2

**¿Qué es?** VDL Mode 2 (VHF Digital Link Mode 2) es el sucesor digital de ACARS. Usa la misma banda VHF (~136 MHz) pero con modulación digital D8PSK a 31.5 kbps — mucho más eficiente. Está reemplazando progresivamente a ACARS en rutas de alta densidad. Transmite los mismos tipos de mensajes que ACARS pero con mayor capacidad y soporte para protocolos avanzados (CPDLC, ADS-C, CM). `dumpvdl2` es el decodificador de referencia en Linux, escrito por el mismo autor que `dumphfdl` y RTL-Airband.

**¿Qué podés hacer con él?**
- Decodificar mensajes CPDLC (instrucciones de ATC por datos, no voz)
- Recibir reportes ADS-C de posición de aviones sobre rutas oceánicas
- Monitorear hasta 8 canales VDL2 simultáneamente
- Exportar via JSON, UDP, ZeroMQ para integración con otros sistemas
- Alimentar airframes.io o bases de datos propias

**Hardware mínimo:** RTL-SDR. Funciona, aunque VDL2 es más exigente en SNR que ACARS analógico.  
**Hardware ideal:** Airspy o SDRplay para mejor decodificación con mayor ancho de banda.

**Frecuencia principal:** `136.975 MHz` (canal común / CSC). Secundarias: `136.725`, `136.775`, `136.875` MHz.

**Diferencia con ACARS:** VDL2 es digital — no se "escucha" como beeps, se ve como ráfagas de datos. Mayor densidad de información por mensaje, protocolo más rico (X.25, ICAO Context Management).

**Puerto en DragonOS:** `/usr/src/dumpvdl2/`  
**GitHub:** `github.com/szpajder/dumpvdl2`

---

### 1.5 HF-DL — dumpHFDL

**¿Qué es?** HFDL (High Frequency Data Link) es la versión de ACARS que corre en HF (shortwave, 3–30 MHz) en lugar de VHF. Se usa específicamente para vuelos sobre océanos y áreas remotas donde no hay cobertura VHF ni satélite. Una red de estaciones terrestres distribuidas globalmente mantiene el sistema activo. Los aviones sobre el Atlántico, Pacífico y Ártico lo usan como respaldo o primario. `dumphfdl` (mismo autor que dumpvdl2) es el único decodificador multicanal open source para Linux.

**¿Qué podés hacer con él?**
- Rastrear vuelos transoceánicos que no aparecen en FlightRadar24
- Recibir reportes de posición ADS-C de vuelos sobre el Atlántico desde Costa Rica
- Monitorear múltiples subandas HF simultáneamente
- Ver el contenido de mensajes ACARS/CPDLC sobre HF

**Hardware mínimo:** RTL-SDR v3 en modo direct sampling (acceso a HF). Funciona pero con limitaciones serias de sensibilidad.  
**Hardware ideal:** Airspy HF+ o SDRplay RSP1A — el HF requiere buena sensibilidad. Antena de hilo largo (wire antenna) exterior.

**Particularidad crítica:** HF tiene propagación ionosférica — las señales rebotan en la ionósfera. Desde Costa Rica podés recibir vuelos sobre el Atlántico Norte o incluso el Pacífico si las condiciones son correctas. La recepción varía enormemente con la hora del día y el ciclo solar.

**Subandas HFDL activas (MHz):** 2.998 / 4.681 / 5.652 / 6.532 / 8.825 / 10.081 / 11.384 / 13.276 / 17.901 / 21.934

**Puerto en DragonOS:** `/usr/src/dumphfdl/`  
**GitHub:** `github.com/szpajder/dumphfdl`

---

### 1.6 L-Band / Inmarsat — Jaero + SDRReceiver

**¿Qué es?** Inmarsat es una red de satélites geoestacionarios que provee comunicaciones a aviones sobre océanos. Transmite en L-Band (~1.545–1.555 GHz). Los canales aeronáuticos (AERO) llevan mensajes ACARS, ADS-C, CPDLC y voz. A diferencia de Iridium (LEO), Inmarsat está fijo en el cielo — apuntás la antena una vez y listo. `Jaero` es el decodificador con GUI. `SDRReceiver` es el frontend que alimenta a Jaero con audio demodulado.

**¿Qué podés hacer con él?**
- Recibir mensajes ACARS y CPDLC de vuelos sobre el Atlántico y Pacífico vía satélite
- Ver posición ADS-C de aviones que no emiten ADS-B (sobre océanos)
- Decodificar voz aeronáutica satelital (con hardware adecuado)
- Alimentar airframes.io con datos L-Band

**Hardware mínimo:** RTL-SDR + antena de parche L-Band (~$15–30). El satélite Inmarsat-4 cubre las Américas desde ~25°W.  
**Hardware ideal:** SDRplay RSP1A o Airspy con LNA dedicado a 1.5 GHz (Nooelec SAWbird+).

**Antena:** La clave está en la antena. Una simple antena de parche pasiva de 1.5 GHz apuntada al satélite es suficiente. No necesitás rotador — el satélite es geoestacionario. Desde Costa Rica el satélite principal (I4-F3) está al norte con elevación ~50°.

**Flujo de trabajo:** `SDRReceiver` (configurado con .ini del satélite) → audio demodulado → `Jaero` (decodifica AERO ACARS). Se pueden correr múltiples instancias de Jaero para cubrir varios VFOs simultáneamente.

**Puerto en DragonOS:** Jaero en menú de aplicaciones / `SDRReceiver` en `/usr/local/bin/`

---

### 1.7 Iridium — iridium-toolkit / GR-Iridium

**¿Qué es?** Iridium es una constelación de 66 satélites LEO que orbitan a ~780 km. A diferencia de Inmarsat (geoestacionario), los satélites Iridium pasan sobre tu cabeza constantemente. Transmiten en L-Band (1616–1626.5 MHz) en ráfagas cortas (bursts) repartidas en múltiples frecuencias. Los aviones usan Iridium para voz y datos sobre áreas remotas. `gr-iridium` captura los bursts con GNU Radio. `iridium-toolkit` (Chaos Computer Club Munich) los decodifica.

**¿Qué podés hacer con él?**
- Decodificar mensajes ACARS de vuelos que usan Iridium como enlace
- Interceptar tráfico de balizas de emergencia (EPIRB, PLB) — uso legal para investigación
- Ver actividad de la constelación completa en tiempo real
- Análisis de seguridad del protocolo Iridium (investigación académica)

**Hardware mínimo:** RTL-SDR. Funciona pero captura poca fracción del espectro Iridium (2.4 MHz de 10 MHz totales).  
**Hardware ideal:** HackRF One (20 MHz de ancho de banda) — captura toda la banda Iridium de una. CPU potente requerida — es intensivo.

**Particularidad crítica:** Al ser LEO, los satélites se mueven — hay corrección Doppler significativa. El procesamiento es mucho más complejo que Inmarsat. No recomendado como primer proyecto SDR.  
**Antena:** Omnidireccional L-Band con vista al cielo despejada. Parche o helicoidal funciona.

**Flujo de trabajo:** `gr-iridium` (captura bursts) → archivo de tramas → `iridium-toolkit` (decodifica) → datos ACARS/voz

**Puerto en DragonOS:** `GR-Iridium` en `/usr/src/` + `iridium-toolkit` en `/usr/src/`  
**GitHub:** `github.com/muccc/iridium-toolkit` / `github.com/dholm/gr-iridium`

---

## 2. Redes celulares / telecom

> *Herramientas para análisis pasivo (y activo con hardware apropiado) de redes GSM, LTE y 5G. Zona de uso más delicado legalmente — contexto de investigación y auditoría.*

### 2.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **SDRAngel** | Demodulación GSM, visualización de celdas LTE, análisis de tráfico |
| **GQRX** | Monitoreo pasivo de espectro celular, identificación de bandas activas |
| **SDR++** | Barrido de espectro para identificar celdas activas |

### 2.2 GSM — gr-gsm / IMSI-catcher

**¿Qué es?** GSM (2G) es la generación más vieja de redes celulares, activa desde los 90. Aunque muchos países están apagando la red 2G, en Latinoamérica sigue activa (Claro y Movistar en Costa Rica usan 850 MHz para GSM). `gr-gsm` es un conjunto de bloques GNU Radio que permite recibir y decodificar transmisiones GSM con un RTL-SDR. Es básicamente un analizador de protocolo celular pasivo.

**¿Qué podés hacer con él?**
- Decodificar mensajes de control del canal de broadcast (C0) de cualquier celda GSM cercana
- Ver configuración de la celda, handovers, paging y mensajes del sistema
- Exportar tráfico GSM a Wireshark en formato GSMTAP (puerto UDP 4729) para análisis detallado
- Identificar celdas activas, su ARFCN, LAC (Location Area Code) y Cell ID
- Con `IMSI-catcher`: registrar los IMSIs de dispositivos GSM en el área (solo control channel — no contenido)

**Lo que NO podés hacer (sin claves de encriptación):** leer voz, SMS o datos. El tráfico de usuario en GSM va encriptado con A5/1 o A5/3. Decodificarlo requiere la clave Ki del SIM, que es ilegal obtener sin autorización.

**Hardware mínimo:** RTL-SDR. GSM usa canales de 200 kHz — perfectamente manejable.  
**Frecuencias en Costa Rica:** GSM 850 downlink `869–894 MHz` (Claro / Movistar)

**Herramientas incluidas:**
- `grgsm_livemon` — GUI de monitoreo en tiempo real con espectro
- `grgsm_capture` — captura a archivo para análisis posterior
- `grgsm_decode` — decodifica capturas guardadas, soporta A5/1, A5/2, A5/3
- `IMSI-catcher` — script que registra IMSIs detectados en el canal de control

**Nota legal:** recibir es legal en la mayoría de jurisdicciones. Interceptar tráfico de terceros o usar IMSIs para trackear personas es ilegal en Costa Rica (Ley 8968 de Protección de Datos).

**Puerto en DragonOS:** `/usr/src/gr-gsm/`  
**GitHub:** `github.com/ptrkrysik/gr-gsm`

---

### 2.3 LTE — srsRAN / LTE-Cell-Scanner

**¿Qué es?** `srsRAN` es el stack de software más completo para trabajar con redes 4G LTE y 5G NR de forma open source. Desarrollado por Software Radio Systems (SRS), permite construir una red LTE/5G completa en software. Tiene tres componentes principales: `srsUE` (emula un dispositivo móvil), `srsENB` (emula una estación base eNodeB), y `srsEPC` (core de red). `LTE-Cell-Scanner` es una herramienta independiente más ligera para detección pasiva de celdas.

**¿Qué podés hacer con él?**
- **Pasivo (RTL-SDR):** escanear el espectro LTE y detectar celdas activas — ver banda, EARFCN, PCI (Physical Cell ID), señal
- **Activo (LimeSDR/BladeRF/USRP):** construir tu propia red LTE privada completa para investigación
- Conectar teléfonos reales con SIMs programables a tu red LTE propia
- Estudiar el protocolo LTE completo desde la capa física hasta la capa de aplicación
- Desarrollar y testear aplicaciones de red celular sin depender de operadores

**Hardware para uso pasivo (solo escucha):** RTL-SDR funciona con LTE-Cell-Scanner  
**Hardware para uso activo (red propia):** LimeSDR, BladeRF xA4/xA9 o USRP B210 — full-duplex obligatorio

**Frecuencias LTE en Costa Rica:**
- Band 4 (AWS): downlink `2110–2155 MHz`, uplink `1710–1755 MHz` (Claro, Movistar, ICE)
- Band 2 (PCS): downlink `1930–1990 MHz` (ICE)
- Band 28 (700 MHz): `758–803 MHz` downlink (ICE, expansión rural)

**Nota crítica:** operar una red LTE privada que transmite en frecuencias licenciadas es ilegal sin permiso de SUTEL en Costa Rica. Para research se usan atenuadores y jaulas de Faraday, o frecuencias ISM con hardware adecuado.

**Puerto en DragonOS:** `/usr/src/srsRAN_4G/` y `/usr/src/srsRAN/`  
**Documentación:** `docs.srsran.com`

---

### 2.4 OsmocomBB

**¿Qué es?** `OsmocomBB` es firmware open source para el procesador de banda base de teléfonos celulares viejos (específicamente los basados en el chipset Calypso de Texas Instruments — Motorola C115, C123, C155, etc.). Reemplaza el firmware propietario del teléfono y permite usar el dispositivo como un analizador GSM programable. A diferencia de gr-gsm (que corre en una PC con SDR), OsmocomBB corre en el chipset de baseband del teléfono físico.

**¿Qué podés hacer con él?**
- Convertir un teléfono viejo en un sniffer GSM activo que puede escuchar uplink y downlink
- Realizar ataques de seguridad de investigación: IMSI catching, A5/1 cracking (con rainbow tables)
- Detectar IMSI catchers (stingrays) en el entorno — el teléfono puede ver si una celda se comporta de forma anómala
- Desarrollar aplicaciones de bajo nivel sobre el stack GSM
- Experimentar con el protocolo a nivel de burst individual

**Hardware requerido:** teléfono con chipset Calypso (Motorola C1xx, Sony Ericsson T190, etc.) + cable de debug especial (cable serial modificado). **No funciona con SDR moderno.**

**Estado actual:** proyecto maduro pero de nicho. Los teléfonos Calypso son cada vez más difíciles de conseguir. La red GSM en muchos países se está apagando, reduciendo su utilidad práctica.

**Nota:** OsmocomBB tiene múltiples sub-proyectos útiles en DragonOS: `OsmoTRX` (interfaz RF para BTS), `OsmoNITB` / `OsmoMSC` (core GSM), `OsmoHLR` (registro de abonados).

**GitHub:** `github.com/osmocom/osmocom-bb`

---

### 2.5 CalypsoBTS / YateBTS

**¿Qué es?** Ambas son implementaciones de estaciones base GSM (BTS) en software. `CalypsoBTS` convierte un teléfono Calypso en una mini-BTS. `YateBTS` es un fork de OpenBTS que usa el motor de telefonía Yate — permite crear una red GSM 2G completa y funcional con SDR, incluyendo llamadas, SMS y datos GPRS entre dispositivos registrados. Cemaxecuter demostró en DragonOS una llamada en conferencia entre Georgia (EE.UU.), California y Australia usando YateBTS + BladeRF.

**¿Qué podés hacer con él?**
- Crear una red GSM privada completa — teléfonos viejos pueden hacer llamadas y enviar SMS entre sí
- Estudiar el core de red GSM: MSC, HLR, SGSN — toda la arquitectura en una sola PC
- Conectar la red a VoIP/SIP para enrutar llamadas hacia el exterior
- Investigar vulnerabilidades del protocolo GSM en entorno controlado
- Mantener vivos teléfonos 2G que ya no tienen red del operador

**Hardware requerido:** SDR full-duplex — BladeRF xA4, LimeSDR, o USRP. El RTL-SDR no sirve (solo RX).  
**Hardware mínimo real:** BladeRF micro xA4 (~$200) — el más económico que funciona bien con YateBTS.

**Limitación fundamental:** GSM no tiene autenticación de red hacia el dispositivo — el teléfono se conecta a cualquier BTS sin verificar su identidad. Esto es una vulnerabilidad de diseño inherente al estándar 2G, y la base de los ataques MITM GSM históricos.

**Nota legal:** transmitir en frecuencias licenciadas sin permiso es ilegal. Uso legítimo: laboratorio aislado (jaula de Faraday o atenuadores), demostración académica, museos de tecnología, o con licencia experimental de SUTEL.

**Puerto en DragonOS:** YateBTS en menú de aplicaciones / CalypsoBTS en `/usr/src/`

---

## 3. WiFi y redes inalámbricas

> *Monitoreo de espectro 2.4/5 GHz, detección de redes, análisis de tráfico y herramientas de auditoría de seguridad inalámbrica.*

### 3.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **Sparrow-WiFi** | Es en sí mismo un frontend visual para WiFi + SDR combinado |
| **SDR++** | Visualización del espectro 2.4/5 GHz con HackRF |
| **SDRAngel** | Análisis de espectro banda ISM |

### 3.2 Kismet

**¿Qué es?** Kismet es el estándar de facto en monitoreo pasivo de redes inalámbricas en Linux. Es simultáneamente detector de redes, sniffer de paquetes y sistema de detección de intrusiones (WIDS). Su característica clave es que opera **completamente pasivo** — no transmite nada, no inyecta paquetes, solo escucha. Esto lo hace indetectable para los administradores de red. Originalmente solo WiFi, hoy soporta una cantidad asombrosa de protocolos adicionales con el hardware correcto.

**¿Qué podés hacer con él?**
- Detectar **todas** las redes WiFi en el área, incluyendo redes ocultas (hidden SSID) — las identifica por los probe requests de los clientes aunque no beaconeen su nombre
- Ver clientes asociados a cada AP, dispositivos que están probando redes, y movimiento de dispositivos entre APs
- Detectar redes rogues (APs falsos), evil twins, y cambios sospechosos de BSSID
- **Wardriving** con GPS — mapear redes en movimiento con coordenadas exactas
- Con hardware adicional: capturar Bluetooth LE/Classic, Zigbee (802.15.4), nRF (teclados/ratones inalámbricos), ADS-B, medidores de luz/agua/gas, termómetros inalámbricos, detectores de humo
- Exportar a PCAP, JSON, SQLite para análisis posterior con Wireshark u otras herramientas
- UI web en `localhost:2501` — no requiere cliente instalado, accesible desde cualquier browser

**Hardware para WiFi:** cualquier adaptador que soporte modo monitor. El chip RTL8812AU (Alfa AWUS036ACH) es el más recomendado en DragonOS.  
**Hardware para extras:** Ubertooth One (Bluetooth), nRF52840 dongle (Zigbee/BLE), RTL-SDR (ADS-B, AMR meters)

**Diferencia clave con otras herramientas:** Kismet no hace nada activo. Si necesitás inyectar paquetes o hacer deauth → Aircrack-ng. Si necesitás ver el espectro RF → Sparrow-WiFi. Kismet es el observador silencioso.

**Puerto en DragonOS:** disponible en menú / `kismet` en terminal  
**Web:** `kismetwireless.net` / GitHub: `github.com/kismetwireless/kismet`

---

### 3.3 Sparrow-WiFi

**¿Qué es?** Sparrow-WiFi es el punto donde el análisis WiFi tradicional se encuentra con el SDR. Es una herramienta GUI que combina en una sola pantalla: escaneo de redes WiFi, descubrimiento Bluetooth, visualización del espectro RF en tiempo real (con HackRF o Ubertooth), tracking GPS, y detección de drones por RemoteID. Cemaxecuter lo usa frecuentemente en sus demos de DragonOS Noble con HackRF + Ubertooth.

**¿Qué podés hacer con él?**
- Ver redes WiFi con sus canales, señal, encriptación y clientes — igual que inSSIDer pero en Linux
- **Overlay de espectro en tiempo real**: ver el espectro RF de 2.4 GHz (Ubertooth) o 2.4/5 GHz (HackRF) superpuesto sobre el mapa de canales WiFi — invaluable para diagnosticar interferencia no-WiFi
- Rastrear la fuente física de una señal WiFi en modo "hunt" — muestras múltiples por segundo con telemetría de señal
- Descubrir dispositivos Bluetooth LE/Classic cercanos — con Ubertooth en modo promiscuo captura también Classic BT
- Detectar drones por FAA RemoteID (WiFi ASTM F3411 y Bluetooth LE) con mapas y alertas
- Agente headless en Raspberry Pi para despliegues remotos o montado en dron/rover
- Integración con Aircrack-ng (plugin Falcon) para operaciones activas controladas

**Hardware mínimo:** adaptador WiFi con modo monitor + adaptador Bluetooth estándar  
**Hardware ideal:** HackRF One + Ubertooth One + adaptador WiFi Alfa AWUS036ACH + GPS

**La combinación HackRF + Ubertooth:** el HackRF hace sweep del espectro completo (0.5 MHz resolución en 2.4 GHz, 2 MHz en 5 GHz) mientras el Ubertooth captura tráfico Bluetooth en modo promiscuo. Juntos dan una imagen completa del espectro ISM que ninguna herramienta sola puede dar.

**Puerto en DragonOS:** menú de aplicaciones (con activación de venv)  
**GitHub:** `github.com/ghostop14/sparrow-wifi`

---

### 3.4 Aircrack-ng

**¿Qué es?** Aircrack-ng es el suite de herramientas más conocido para auditoría de seguridad de redes WiFi. No es una herramienta, es un conjunto completo: captura, inyección, análisis y cracking de claves. Está en prácticamente todas las distribuciones de seguridad (Kali, DragonOS, Parrot). Requiere un adaptador WiFi con soporte de modo monitor e inyección de paquetes — la tarjeta integrada del laptop generalmente no sirve.

**Herramientas del suite y qué hace cada una:**
- `airmon-ng` — pone la tarjeta en modo monitor (prerequisito para todo lo demás)
- `airodump-ng` — captura paquetes, lista redes y clientes, guarda handshakes WPA a archivo
- `aireplay-ng` — inyección de paquetes: deauthentication, fake authentication, chopchop, fragmentation
- `aircrack-ng` — cracking de claves WEP (estadístico) y WPA/WPA2 (diccionario/fuerza bruta sobre handshake capturado)
- `airbase-ng` — crea un AP falso (evil twin, Karmetasploit)
- `airdecap-ng` — desencripta archivos PCAP de redes WEP/WPA conocidas

**¿Qué podés hacer con él?**
- Auditar la seguridad de tu propia red WiFi — capturar el handshake WPA2 y testear la fortaleza de tu contraseña
- Detectar y documentar redes con encriptación débil (WEP, WPS habilitado)
- Simular ataques de deauthentication para testing de resiliencia
- Capturar tráfico de redes abiertas o WEP para análisis

**Hardware:** adaptador WiFi con modo monitor + inyección. Los más confiables: Alfa AWUS036ACH (rtl8812au), Alfa AWUS036ACM (mt7612u), TP-Link TL-WN722N v1 (ath9k_htc).

**Importante:** WEP está muerto — se rompe en minutos con suficientes IVs. WPA2 con contraseña fuerte y sin WPS es resistente a ataques prácticos. WPA3 es prácticamente inmune a ataques de diccionario offline.

**Nota legal:** usar estas herramientas en redes ajenas sin autorización explícita es delito en Costa Rica (Ley 9048, artículo 230). Uso legítimo: tu propia red, redes de laboratorio, con autorización escrita del propietario.

**Puerto en DragonOS:** disponible por defecto en el sistema  
**Web:** `aircrack-ng.org`

---

### 3.5 WHAD-client

**¿Qué es?** WHAD (Wireless HAcking Devices) es un framework Python presentado en DEF CON 32 (2024) que unifica la forma en que las herramientas de hacking inalámbrico interactúan con el hardware. La idea central: el hardware maneja las tareas de RF, la lógica vive en la computadora. Esto hace que cualquier herramienta WHAD funcione con cualquier hardware compatible — cambiás el dongle sin reescribir el código.

**¿Qué podés hacer con él?**
- Sniffing, inyección y conexión a dispositivos **Bluetooth Low Energy** (BLE) — leer características GATT, escribir valores, suscribirse a notificaciones
- Ataques BLE: MitM, hijacking de conexiones, fuzzing de perfiles GATT, spoofing de dispositivos
- **Zigbee (802.15.4)**: sniffing, inyección, análisis de redes IoT domóticas
- **Enhanced ShockBurst** (Logitech): interceptar teclados y ratones inalámbricos USB
- Captura a PCAP con visualización en Wireshark en tiempo real
- Scripting Python para automatizar análisis y exploits de dispositivos IoT

**Hardware compatible:** nRF52840 dongle (con firmware ButteRFly), Ubertooth One, HackRF, y otros dispositivos con firmware WHAD

**Diferencia con Kismet/Sparrow:** WHAD es activo e interactivo — no solo observa, puede conectarse, interactuar y atacar dispositivos BLE/Zigbee específicos. Es el Burp Suite del mundo inalámbrico IoT.

**Contexto DragonOS:** es una de las adiciones más recientes — fue presentado en DEF CON 32 en agosto 2024 y Cemaxecuter lo integró en DragonOS Noble relativamente rápido, lo que habla de su relevancia en el ecosistema.

**Puerto en DragonOS:** `/usr/src/` (activar venv antes de usar)  
**GitHub:** `github.com/whad-team/whad-client`  
**Documentación:** `whad.readthedocs.io`

---

## 4. Satélites

> *Recepción de satélites meteorológicos, satélites de órbita baja, GPS y satélites geoestacionarios. Algunos de los casos de uso más espectaculares del SDR — imágenes de satélite en tiempo real con hardware de menos de $30.*

### 4.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **SatDump** | Es en sí mismo el frontend principal — decodifica y renderiza imágenes de satélite |
| **SDR++** | Recepción del stream raw antes de pasar a SatDump (modo source) |
| **GQRX** | Monitoreo de señal durante el pase del satélite |
| **SDRAngel** | Seguimiento de satélites con corrección Doppler automática |

### 4.2 Satélites meteorológicos — SatDump / NOAA / METEOR

**¿Qué es?** SatDump es el decodificador universal de datos satelitales — el proyecto que unificó lo que antes requerían 5 herramientas distintas. Procesa señales de más de 90 satélites diferentes, convierte la señal RF directamente en imágenes georeferenciadas, composites de color, datos científicos calibrados, y productos listos para uso meteorológico. Es activamente desarrollado (actualizaciones casi diarias) y es hoy el estándar de facto en la comunidad SDR para satélites.

**¿Qué satélites podés recibir desde Costa Rica?**

*LEO meteorológicos en 137 MHz (pases de ~10 min, antena simple):*
- **NOAA-15, NOAA-18, NOAA-19** — APT (analógico), imagen visible e infrarroja, ~4 km/píxel. Los más fáciles de recibir — señal fuerte, protocolo simple
- **Meteor-M N°2-3 y N°2-4** (Rusia) — LRPT (digital), imagen en color visible e IR, ~1 km/píxel. Mejor resolución que NOAA pero señal más delicada

*Geoestacionarios en L-Band ~1.69 GHz (fijos en el cielo, antena apuntada):*
- **GOES-16** (75.2°W) — cobertura perfecta de las Américas incluyendo Costa Rica. Transmite HRIT/EMWIN cada 10 minutos. Imágenes de disco completo en 16 bandas espectrales
- **GOES-18** (137.2°W) — cubre el Pacífico Este, también visible desde CR
- **Metop-B y C** (EUMETSAT) — polares europeos, pasan por CR, AHRPT a 1.7 GHz con imagen de alta resolución

**¿Qué podés hacer con él?**
- Recibir imágenes de satélite meteorológico en tiempo real con un RTL-SDR y antena de $10
- Generar composites calibrados (temperatura superficial del mar, microfísica de nubes, vapor de agua)
- Georeferenciar imágenes automáticamente sobre mapas — la imagen sale alineada al mundo real
- Decodificar el sistema GOES DCS (40.000 plataformas de datos ambientales en las Américas — niveles de ríos, temperatura, precipitación)
- Automatizar la recepción de pases con el scheduler integrado + control de rotador de antena
- Exportar a PNG, GeoTIFF — compatible con QGIS, Google Earth, WRF numérico

**Hardware para 137 MHz (NOAA/Meteor):** RTL-SDR + antena V-dipole o QFH — la V-dipole se hace en 20 minutos con cable coaxial. Costo total del sistema: ~$25.  
**Hardware para 1.69 GHz (GOES/Metop):** RTL-SDR + antena de parche L-Band + LNA (Nooelec SAWbird+ GOES). El GOES-16 desde Costa Rica está a ~45° de elevación al norte.

**El caso más impactante para principiantes:** en tu primer intento con una V-dipole casera y un RTL-SDR podés recibir una imagen de Costa Rica desde el espacio en menos de una hora de setup. Es probablemente el resultado más satisfactorio y visual de todo el ecosistema SDR.

**Puerto en DragonOS:** menú de aplicaciones  
**Web:** `satdump.org` / GitHub: `github.com/SatDump/SatDump`

---

### 4.3 GPS / GNSS — GNSS-SDR

**¿Qué es?** GNSS-SDR es un receptor GPS/GNSS implementado completamente en software sobre GNU Radio. En lugar de depender de un chip GPS dedicado, procesa las señales crudas de los satélites de navegación en software — adquisición, seguimiento de código y portadora, decodificación del mensaje de navegación, y cálculo de posición/velocidad/tiempo (PVT). Es una herramienta de investigación, no un reemplazo para el GPS de tu teléfono.

**¿Qué podés hacer con él?**
- Entender cómo funciona GPS a nivel de señal — ver los PRN codes, el proceso de correlación, la decodificación del almanaque
- Investigar vulnerabilidades de GPS: spoofing, jamming, meaconing — el SDR permite inyectar señales falsas en entorno controlado
- Probar algoritmos de posicionamiento de alta precisión (RTK, PPP) en software
- Recibir múltiples constelaciones simultáneamente: GPS L1/L2, GLONASS L1, Galileo E1/E5, BeiDou B1
- Capturar y almacenar señales crudas para análisis posterior o replay

**Hardware:** necesita una **antena GPS activa** (patch con amplificador interno, ~1575 MHz) + bias tee para alimentarla.
- RTL-SDR v3 (tiene bias tee) — funciona para GPS L1, sensibilidad limitada
- Airspy R2 o USRP — mejor rendimiento, más constelaciones simultáneas
- HackRF — funciona pero no tiene LNA dedicado, peor SNR

**La señal GPS es extremadamente débil:** llega a ~-130 dBm — por debajo del nivel de ruido del receptor. Se extrae mediante correlación de pseudonoise (PRN) codes. Sin LNA de buena calidad, simplemente no se ve.

**Diferencia con GPS-Simulator (sección 7.3):** GNSS-SDR *recibe* señales reales de satélites GPS. GPS-Simulator *transmite* señales GPS falsas — son opuestos complementarios.

**Casos de uso reales:** investigación académica en spoofing/anti-spoofing, desarrollo de algoritmos de posicionamiento, calibración de receptores, educación en sistemas de navegación por satélite.

**Puerto en DragonOS:** disponible vía GNU Radio  
**Web:** `gnss-sdr.org`

---

### 4.4 GR-Iridium + Iridium-Toolkit

> *Ver descripción completa en sección [1.7 — Iridium](#17-iridium--iridium-toolkit--gr-iridium). En el contexto de satélites, aplica todo lo descrito ahí.*

**Resumen de contexto satelital:** Iridium es la única constelación LEO de comunicaciones cubierta en DragonOS. La diferencia respecto a NOAA/METEOR/GOES es que no transmite imágenes — transmite comunicaciones (voz, datos, mensajes ACARS aeronáuticos, señales de emergencia EPIRB/PLB). Su decodificación es técnicamente más compleja y requiere mayor CPU que los satélites meteorológicos.

**Hardware diferenciador:** para satélites meteorológicos un RTL-SDR es suficiente. Para Iridium, el HackRF One es el hardware mínimo práctico — su ancho de banda de 20 MHz captura toda la banda Iridium de una sola vez, mientras el RTL-SDR solo captura 2.4 MHz de los 10 MHz totales.

---

## 5. Trunking y radiocomunicaciones terrestres

> *Sistemas de radio usados por policía, bomberos, servicios de emergencia, transporte público. Muchos sistemas son digitales y encriptados, pero hay mucho que se puede monitorear legalmente.*

### 5.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **SDRTrunk** | Es en sí mismo un frontend especializado en trunking con waterfall propio |
| **SDRAngel** | Demoduladores P25, DMR, NXDN integrados |
| **GQRX** | Monitoreo de canales analógicos NFM individuales |
| **OpenWebRX+** | Acceso remoto a feeds de trunking |

### 5.2 SDRTrunk

**¿Qué es?** SDRTrunk es la aplicación de referencia para seguimiento y decodificación de sistemas de radio troncalizada (trunked radio) con SDR. El concepto de trunking: en lugar de asignar una frecuencia fija a cada grupo (bomberos, policía, ambulancias), un sistema troncalizado comparte un pool de frecuencias entre todos. Un **canal de control** asigna frecuencias a grupos bajo demanda. Para seguir una conversación hay que decodificar el canal de control y saltar a la frecuencia asignada — todo en tiempo real. SDRTrunk automatiza exactamente eso.

**¿Qué podés hacer con él?**
- Seguir conversaciones en sistemas P25 Phase 1 y 2, DMR, NXDN, MPT1327 en tiempo real
- Ver todos los talkgroups (grupos de conversación) activos y sus alias (si tenés la base de datos de RadioReference)
- Grabar llamadas por talkgroup a archivos de audio individuales
- Stream de audio a servidores remotos (Broadcastify, OpenMHz)
- Monitorear múltiples sistemas simultáneamente con múltiples dongles
- Ver mensajes de datos del canal de control: asignaciones de frecuencia, afiliaciones, ubicación GPS si el sistema la transmite

**Hardware mínimo:** 1 RTL-SDR para sistemas convencionales (no trunked). Para trunking real **se necesitan 2 RTL-SDRs** — uno dedicado al canal de control, uno para voz. Con SDRs de mayor ancho de banda (SDRplay RSP1A, Airspy R2) un solo dispositivo puede cubrir todo el sistema si las frecuencias caben en el ancho de banda.

**La barrera real no es hardware sino datos:** necesitás conocer las frecuencias del canal de control y los IDs del sistema. En EE.UU. RadioReference tiene todo mapeado. En Costa Rica no existe esa base de datos pública — requiere trabajo de campo previo para identificar el sistema.

**Protocolos:** P25 Phase 1 (C4FM/FSK4), P25 Phase 2 (TDMA), DMR (Tier II/III), NXDN, LTR, Passport, MPT1327.

**Puerto en DragonOS:** `/usr/src/` — aplicación Java, requiere JVM  
**GitHub:** `github.com/DSheirer/sdrtrunk`

---

### 5.3 Trunk-Recorder

**¿Qué es?** Trunk-Recorder es la versión headless (sin GUI) y orientada a producción de lo que SDRTrunk hace interactivamente. Está diseñado para correr 24/7 en un servidor o Raspberry Pi, grabar cada llamada de un sistema troncalizado, y subirla automáticamente a plataformas públicas como **OpenMHz** o **Broadcastify**. A diferencia de SDRTrunk que es para escucha interactiva, Trunk-Recorder es para archivar y publicar.

**¿Qué podés hacer con él?**
- Grabar cada llamada por talkgroup en archivos MP3/WAV con timestamp, duración, y metadatos
- Publicar automáticamente en OpenMHz (plataforma abierta) o Broadcastify (la más popular en EE.UU.)
- Monitorear múltiples sistemas P25/DMR/SmartNet simultáneamente con múltiples SDRs
- Configurar filtros por talkgroup — grabar solo ciertos grupos
- Configurar alertas cuando un talkgroup específico se activa

**Diferencia clave con SDRTrunk:** Trunk-Recorder no tiene GUI — se configura con un archivo JSON y corre en terminal. Es más estable para operación continua, consume menos recursos, y tiene mejor integración con plataformas públicas. SDRTrunk es para escuchar en tiempo real; Trunk-Recorder es para archivar.

**Hardware:** RTL-SDR funciona. Múltiples dongles para múltiples sistemas. Para P25 Phase 2 necesitás un SDR con al menos 1 MHz de ancho de banda simultáneo.

**Caso de uso típico:** estación en casa o servidor en nube que captura toda la actividad de una agencia de emergencias local y la publica en tiempo real para que cualquiera pueda escuchar. Legal en la mayoría de jurisdicciones para sistemas no encriptados.

**GitHub:** `github.com/TrunkRecorder/trunk-recorder`  
**Web:** `trunkrecorder.com`

---

### 5.4 DSD / DSD-FME

**¿Qué es?** DSD (Digital Speech Decoder) es el decodificador de voz digital más portable — toma audio como entrada y lo decodifica. No sabe nada de SDR ni de trunking por sí solo: recibe audio ya demodulado (de GQRX, SDR++ o cualquier fuente) y extrae la voz. `DSD-FME` es el fork mejorado que agrega soporte de trunking avanzado, más protocolos, y interfaz ncurses interactiva.

**¿Qué protocolos decodifica?**
- P25 Phase 1 (FDMA/C4FM)
- DMR (Digital Mobile Radio) — el más usado en radios comerciales
- NXDN — Kenwood NEXEDGE / Icom IDAS
- D-STAR — radio digital de Icom
- dPMR — versión europea de DMR

**¿Qué podés hacer con él?**
- Decodificar voz de cualquier radio digital que uses mientras se sintoniza en tiempo real
- Modo trunking (DSD-FME): conectarse a un RTL-SDR directamente y seguir un sistema P25/DMR
- Modo scanner: barrer un CSV de frecuencias buscando actividad digital
- Ver metadatos de las transmisiones: talkgroup ID, radio ID, tipo de llamada, encriptación activa
- Capturar audio decodificado a archivos WAV por llamada

**La limitación de encriptación:** si el sistema usa encriptación (AES, DES, RC4), DSD ve que hay tráfico y te indica el algoritmo, pero **no puede decodificar el audio** — escuchás ruido digital. Muchos sistemas de emergencias en EE.UU. ya migraron a encriptación total. En Costa Rica la situación varía.

**Flujo típico:** `RTL-SDR` → `GQRX` (demodulación NFM) → audio virtual (PulseAudio/pipe) → `dsd-fme` (decodificación P25/DMR)

**Puerto en DragonOS:** `/usr/src/dsd-fme/`  
**GitHub:** `github.com/lwvmobile/dsd-fme`

---

### 5.5 OP25

**¿Qué es?** OP25 (Osmocom Project 25) es el decodificador P25 de referencia en Linux, basado en GNU Radio. A diferencia de SDRTrunk (que solo hace Phase 1 completo) o DSD (que hace Phase 1 básico), OP25 soporta **P25 Phase 1 Y Phase 2** completos incluyendo trunking avanzado. Es más difícil de configurar que SDRTrunk pero más potente para sistemas Phase 2, que es hacia donde está migrando la mayoría de agencias públicas.

**¿Qué podés hacer con él?**
- Decodificar P25 Phase 1 y **Phase 2** (TDMA) — la diferencia crítica con otras herramientas
- Trunking completo con seguimiento de canal de control y salto automático a voz
- Transmisión P25 (con hardware TX apropiado) — HackRF, USRP
- Multi-receptor: múltiples dongles para cubrir más ancho de banda simultáneo
- Plots de constelación y símbolo para diagnóstico de calidad de señal

**La diferencia Phase 1 vs Phase 2:** Phase 2 usa TDMA (dos conversaciones en el mismo canal) y modulación QPSK en lugar de FDMA/C4FM de Phase 1. Los decodificadores de Phase 1 simplemente no funcionan con Phase 2 — se necesita OP25 o SDRTrunk actualizado.

**La curva de aprendizaje:** OP25 no tiene una GUI amigable — se configura con archivos `.tsv` y se lanza desde terminal. El fork **Boatbod** es el más activo y el recomendado. Trunk-Recorder usa las librerías de OP25 internamente.

**Hardware:** RTL-SDR básico. Para Phase 2 con múltiples talkgroups simultáneos se recomienda SDR con mayor ancho de banda (SDRplay, Airspy).

**GitHub:** `github.com/boatbod/op25`

---

### 5.6 QRadioLink

**¿Qué es?** QRadioLink es diferente a todas las herramientas anteriores de esta sección — es un **transceptor SDR completo** (RX + TX) orientado a radio amateur digital, no un decodificador de sistemas comerciales. Construido sobre GNU Radio con interfaz Qt5, permite transmitir y recibir modos digitales de voz: DMR, YSF (Yaesu System Fusion), D-STAR, M17, Codec2. Incluye integración VoIP via Mumble y puede operar como repetidor o nodo de red de radio amateur.

**¿Qué podés hacer con él?**
- Transmitir y recibir voz en DMR, D-STAR, YSF, M17, FM analógico con un SDR full-duplex
- Operar como repetidor/hotspot DMR usando LimeSDR o USRP — sin hardware MMDVM dedicado
- Hasta 7 portadoras simultáneas en 200 kHz de ancho de banda (multicarrier)
- Conectar a redes DMR globales (BrandMeister, DMR-MARC) via internet
- Linking de nodos radio vía VoIP/Mumble — unir repetidores de distintos lugares
- Experimentar con el protocolo M17 — el sucesor open source de D-STAR

**Hardware:** RTL-SDR (solo RX). Para TX: **LimeSDR-mini, LimeNET-Micro, USRP, BladeRF, ADALM-Pluto** — full-duplex obligatorio.

**Diferencia con las otras herramientas de esta sección:** SDRTrunk/Trunk-Recorder/DSD/OP25 son herramientas de **monitoreo pasivo** de sistemas existentes. QRadioLink es para **participar activamente** en comunicaciones de radio amateur con SDR.

**Contexto:** si sos radioaficionado con licencia y tenés LimeSDR, QRadioLink convierte tu PC en una estación base DMR capaz de cubrir zonas sin repetidor, experimentar con nuevos protocolos, o enlazar repetidores remotamente.

**Web:** `qradiolink.org` / GitHub: `github.com/qradiolink/qradiolink`

---

## 6. Análisis de señales digitales

> *Herramientas para identificar señales desconocidas, hacer ingeniería inversa de protocolos, analizar modulaciones, y extraer datos de transmisiones capturadas. El lado "forense" del SDR.*

### 6.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **Inspectrum** | Frontend de análisis visual de capturas IQ — el osciloscopio del SDR |
| **SigDigger** | Frontend con demodulación manual configurable y análisis espectral |
| **GQRX** | Captura de señales a archivo IQ para análisis posterior |
| **SDR++** | Grabación de IQ para análisis offline |

### 6.2 Universal Radio Hacker (URH)

**¿Qué es?** URH es la navaja suiza del análisis y reverse engineering de protocolos inalámbricos propietarios. Si capturás una señal de un control remoto, un sensor de alarma, un termóstato WiFi, un medidor inteligente, un portón eléctrico, o cualquier dispositivo IoT que transmite en RF — URH es la herramienta para descifrar qué dice. Combina captura, demodulación automática, análisis de protocolo, y generación/transmisión de señales en una sola GUI. Fue presentado en USENIX WOOT 2018 y desde entonces es referencia en seguridad RF.

**¿Qué podés hacer con él?**
- **Capturar** señales directamente desde el SDR dentro de la propia aplicación
- **Demodulación automática**: detecta el tipo de modulación (ASK/OOK, FSK, PSK) y los parámetros (bit rate, frecuencia de desviación) sin que vos sepas nada del protocolo
- **Análisis de protocolo**: identifica campos repetidos, preámbulos, checksums, y estructura de mensajes automáticamente o manualmente
- **Decodificación personalizada**: NRZ, Manchester, PWM, DC-free, whitening CC1101 — los encodings más usados en IoT
- **Fuzzing**: generar variaciones de mensajes para testear robustez de dispositivos
- **Simulación y transmisión**: una vez entendido el protocolo, transmitir mensajes propios con HackRF
- **Ataques de replay**: capturar el mensaje de apertura de un portón y retransmitirlo

**Casos de uso reales:** abrir un portón de cochera vecino (demo educativa), clonar una llave remota de auto (frecuencias 315/433 MHz), analizar el protocolo de sensores de temperatura inalámbricos, vulnerabilidades en sistemas de alarma RF.

**Hardware:** RTL-SDR para captura. **HackRF One** para el ciclo completo captura+transmisión (replay attacks, fuzzing en vivo).

**Frecuencias ISM típicas de IoT:** 315 MHz (EE.UU.), 433.92 MHz (Europa/CR), 868 MHz (Europa), 915 MHz (EE.UU.), 2.4 GHz (ZigBee, BLE, WiFi)

**Puerto en DragonOS:** disponible en menú  
**GitHub:** `github.com/jopohl/urh`

---

### 6.3 Inspectrum

**¿Qué es?** Inspectrum es el "osciloscopio" del SDR — una herramienta de análisis visual de archivos IQ capturados. No hace decodificación automática ni análisis de protocolo: te muestra la señal de la forma más cruda y precisa posible para que vos la analices. Trabaja offline con archivos de captura (`.cfile`, `.cf32`, `.cs8`, etc.), soporta archivos de más de 100 GB sin problema, y tiene herramientas de medición precisas para extraer parámetros de la señal.

**¿Qué podés hacer con él?**
- Ver el espectrograma de una señal capturada con zoom y pan infinito — ideal para señales cortas y ráfagas (bursts)
- Graficar amplitud, frecuencia, fase y muestras IQ en el tiempo
- Medir el period y symbol rate exactos con cursores interactivos
- Extraer símbolos de la señal visualmente — definís los umbrales de decisión a mano
- Exportar segmentos de tiempo, muestras filtradas, y datos demodulados
- Identificar modulación desconocida observando el patrón visual de la señal

**Diferencia con URH:** Inspectrum es más preciso y visual para análisis fino de señales individuales, pero no tiene la suite completa de análisis de protocolo que URH incluye. Son complementarios: Inspectrum para entender la señal a fondo, URH para descifrar el protocolo. Inspectrum también soporta archivos mucho más grandes y consume menos recursos.

**El flujo típico:**
1. Capturás con GQRX o SDR++ → archivo IQ
2. Inspectrum para ver visualmente la señal, medir bit rate, identificar modulación
3. URH para hacer el análisis del protocolo en capas más altas

**Puerto en DragonOS:** disponible en menú / `inspectrum` en terminal  
**GitHub:** `github.com/miek/inspectrum`

---

### 6.4 SigDigger

**¿Qué es?** SigDigger es un analizador de señales digitales que ocupa el espacio entre GQRX (frontend general) y las herramientas especializadas como URH. Es más avanzado que GQRX para análisis técnico de señales pero más accesible que GNU Radio para tareas rápidas. Clave: **no está basado en GNU Radio** — usa su propia librería DSP (sigutils/Suscan) optimizada para multi-core. Desarrollado por BatchDrake (Gonzalo Carracedo), investigador español de seguridad RF.

**¿Qué podés hacer con él?**
- Análisis de señales en tiempo real Y replay desde archivos IQ
- **Estimación ciega de parámetros**: detecta automáticamente modulación, symbol rate, y otros parámetros de señales desconocidas sin que vos los conozcas de antemano
- Demodulación configurable de FSK, PSK, ASK con visualización de constelación
- Inspección de bursts individuales con detección automática de inicio/fin
- Grabación de banda base (baseband) — captura todo el ancho de banda a archivo
- **Panoramic spectrum**: navegar por el espectro histórico como si fuera una línea de tiempo
- Análisis Doppler y de transición para señales en movimiento
- Demodulación de TV analógica (prueba de concepto histórica interesante)
- Stream de datos demodulados via red — alimentar otras herramientas

**Diferencia clave con GQRX:** SigDigger está orientado a análisis técnico profundo, no a escuchar. No vas a SigDigger para oír una emisora — vas cuando necesitás entender una señal desconocida a nivel técnico.

**Puerto en DragonOS:** disponible en menú (con actualización via `/usr/src/blsd`)  
**GitHub:** `github.com/BatchDrake/SigDigger`

---

### 6.5 GNU Radio

**¿Qué es?** GNU Radio es el **framework** que hace posible casi todo el ecosistema SDR en DragonOS. No es una herramienta para un caso de uso específico — es el motor de procesamiento de señales sobre el que se construyen cientos de aplicaciones. Si SDRTrunk, SatDump, gr-gsm, OP25, QRadioLink, GNSS-SDR, y docenas de otras herramientas existen, es porque GNU Radio les da la infraestructura.

**¿Qué es en términos técnicos?** Un toolkit de procesamiento de señales digitales (DSP) que opera con el modelo de **flowgraph**: un grafo dirigido donde cada nodo es un bloque de procesamiento (filtro, demodulador, decoder, fuente de hardware, sink de audio) y las flechas son flujos de datos IQ. GNU Radio Companion (GRC) es el editor visual de flowgraphs — arrastrás bloques y los conectás como si armaras un circuito.

**¿Qué podés hacer directamente con él?**
- Construir receptores y transmisores completamente customizados desde cero
- Prototipado rápido de algoritmos de comunicación (nuevas modulaciones, nuevos protocolos)
- Procesar señales capturadas offline con cadenas de procesamiento arbitrarias
- Crear bloques propios en Python o C++ e integrarlos al ecosistema
- Visualizar señales con sinks: waterfall, FFT, constelación, osciloscopio, histograma

**La curva de aprendizaje:** GNU Radio no es una herramienta que se usa casualmente. Requiere entender DSP (filtros, muestreo, modulaciones) y tiene una curva inicial empinada. Pero dominar GNU Radio es dominar el SDR a nivel fundamental — todo lo demás son abstracciones sobre él.

**Versión en DragonOS:** GNU Radio 3.10 en DragonOS Noble. Muchos módulos out-of-tree (OOT) instalados: gr-gsm, gr-iridium, gr-osmosdr, gr-lora, gr-limesdr, entre otros.

**Web:** `gnuradio.org`

---

### 6.6 Signal Server

**¿Qué es?** Signal Server es una herramienta de modelado de propagación de RF basada en datos topográficos. No analiza señales reales — **predice** cómo se va a propagar una señal de radio en el espacio geográfico dado el terreno, la frecuencia, la potencia del transmisor y la altura de las antenas. Genera mapas de cobertura sobre cartografía real. Es la misma tecnología que usan operadores de telecomunicaciones para planificar la ubicación de antenas.

**¿Qué podés hacer con él?**
- Generar mapas de cobertura de RF para cualquier frecuencia y ubicación
- Predecir qué áreas van a recibir señal de un transmisor en una ubicación dada
- Planificar ubicaciones óptimas para antenas (repetidores, nodos Meshtastic, APRS, etc.)
- Modelar zonas de sombra de RF en terreno montañoso
- Calcular enlace punto a punto entre dos ubicaciones

**Modelos de propagación incluidos:** ITM (Longley-Rice), ITWOM v3, EPE, Irregular Terrain — cada uno adecuado para distintos rangos de frecuencia y tipo de terreno.

**Caso de uso práctico para vos:** planificar dónde poner un nodo Meshtastic/LoRa en Guanacaste para maximizar cobertura desde tu propiedad, o calcular si hay enlace visual de RF entre Atenas y tu propiedad en Guanacaste para un radioenlace VHF.

**En DragonOS incluye:**
- Signal Server (el motor de cálculo, CLI)
- Signal Server GUI (interfaz gráfica con Python3 venv)
- Signal Server N90ZB (versión con interfaz web, mantenida por Dr. Bill Walker)

**Datos topográficos:** usa datos SRTM (NASA Shuttle Radar Topography Mission) que cubren la práctica totalidad de la Tierra incluyendo Costa Rica. Descargables gratis de USGS.

**Puerto en DragonOS:** Signal Server GUI en menú / Signal Server CLI en `/usr/src/`

---

## 7. Transmisión y herramientas ofensivas

> *Herramientas que permiten transmitir señales, simular dispositivos, o realizar pruebas de seguridad RF. Requieren licencia de radioaficionado o entorno controlado. Hardware mínimo: HackRF One.*

### 7.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **SDRAngel** | Transmisión de señales moduladas con visualización en tiempo real |
| **GNU Radio Companion** | Diseño visual de flujos de transmisión personalizados |
| **SDR++** | Verificación de lo transmitido (con segundo hardware en RX) |

### 7.2 HackTV

**¿Qué es?** HackTV convierte un HackRF en un transmisor de televisión analógica completo. Genera señales PAL, NTSC, SECAM y otros estándares históricos a partir de cualquier archivo de video compatible con ffmpeg (incluyendo streams de red). Es la herramienta que resucita televisores vintage, que fue usada en demostraciones en CTF competitions, y que permitió a Cemaxecuter transmitir video NTSC desde un dron en el contexto del DragonDrone.

**¿Qué podés hacer con él?**
- Transmitir cualquier video a un televisor analógico en PAL (Europa/CR) o NTSC (EE.UU.)
- Soporta estándares históricos: 819 líneas (Francia 1948), 405 líneas (UK 1936), 240 y 30 líneas (TV experimental de los años 30), y el formato de video de las misiones Apollo de la NASA
- Teletext: transmitir páginas de teletext embebidas en la señal PAL
- Audio NICAM estéreo digital
- Videocrypt I/II/S: el sistema de encriptación de Sky TV en los años 90 — investigación histórica de seguridad
- Transmitir la pantalla de tu PC en tiempo real (`ffmpeg::0` como fuente)
- Generar archivos IQ de señal TV para análisis posterior

**Hardware:** HackRF One obligatorio (necesita TX). El RTL-SDR no sirve — es solo RX.  
**Antena:** para transmisión TV UHF (~470–862 MHz en PAL) se usa cualquier dipole o antena UHF.

**Caso de uso práctico:** dar vida a un televisor Sony Trinitron de los años 80 o un Betamax conectado a un viejo TV. También usado en drones para transmisión FPV analógica NTSC sobre frecuencias no licenciadas (con potencia muy baja).

**Nota técnica importante:** HackRF está limitado a 20 MHz de ancho de banda — para transmisión satelital analógica de alta fidelidad (ASTRA FM, 26 MHz) se necesita LimeSDR. Para TV terrestre estándar el HackRF es suficiente.

**Codeberg:** `codeberg.org/fsphil/hacktv` (repositorio activo, GitHub archivado)

---

### 7.3 GPS-Simulator (gps-sdr-sim)

**¿Qué es?** `gps-sdr-sim` genera un stream de datos IQ que simula las señales de la constelación GPS en L1 (1575.42 MHz) para una ubicación geográfica arbitraria. Este stream se transmite con `hackrf_transfer` y cualquier receptor GPS cercano que reciba la señal (no solo el GPS objetivo — todos los que estén dentro del alcance) interpretará estar en la ubicación ficticia. Es la herramienta de investigación en GPS spoofing más usada en el mundo académico y de seguridad.

**¿Qué podés hacer con él?**
- Generar una señal GPS falsa para una coordenada estática cualquier lugar del mundo
- Generar rutas dinámicas (archivos NMEA o ECEF) — el receptor GPS "se mueve" siguiendo la ruta
- Testear firmware de drones, aplicaciones de navegación, y sistemas que dependen de GPS en entorno controlado
- Investigar vulnerabilidades de sistemas GPS: drones, vehículos autónomos, timing de infraestructura
- Reproducir ataques históricos: el autopilot de Tesla redirigido, cheating en Pokémon Go, spoofing de yates en el Mar Negro

**Cómo funciona:**
1. Descargás el archivo de efemérides GNSS del día (NASA CDDIS — gratis)
2. `gps-sdr-sim -e brdc0220.25n -l 9.934739,-84.087502,1200` (coordenadas de San José, CR)
3. Genera `gpssim.bin` (~1.5 GB por minuto de simulación)
4. `hackrf_transfer -t gpssim.bin -f 1575420000 -s 2600000 -a 1 -x 0`

**Hardware:** HackRF One obligatorio para transmisión. Se recomienda atenuador de 50-60 dB entre HackRF y el receptor GPS bajo prueba — la señal es extremadamente fuerte comparada con la real.

**Nota legal crítica:** transmitir señales GPS falsas es ilegal en prácticamente todas las jurisdicciones — incluyendo Costa Rica (SUTEL, Ley 8642). Uso legítimo exclusivamente en: jaula de Faraday, laboratorio apantallado, o con autorización explícita de la autoridad regulatoria. La misma tecnología es usada por Rusia para proteger instalaciones sensibles y por fuerzas militares para proteger VIPs de drones.

**GitHub:** `github.com/osqzss/gps-sdr-sim`

---

### 7.4 hackrf_transfer / hackrf_sweep

**¿Qué son?** Son las herramientas de línea de comando fundamentales del HackRF One — las que vienen con el firmware oficial de Great Scott Gadgets. Todo lo demás (HackTV, GPS-sim, URH, etc.) se construye sobre estas primitivas o las usa internamente.

**hackrf_transfer:**
- Transmite cualquier archivo IQ raw a cualquier frecuencia: `hackrf_transfer -t signal.bin -f 433920000 -s 2000000`
- Recibe y guarda a archivo IQ raw: `hackrf_transfer -r capture.bin -f 915000000 -s 2000000`
- Parámetros: frecuencia (`-f`), sample rate (`-s`), ganancia TX VGA (`-x`), amplificador RF (`-a`), ganancia RX LNA (`-l`), ganancia RX VGA (`-g`)
- Es la "cañería" de datos del HackRF — nada más, nada menos

**hackrf_sweep:**
- Barre el espectro de frecuencias a alta velocidad — mucho más rápido que un SDR normal
- Puede escanear decenas de GHz en segundos: `hackrf_sweep -f 88:108 -N 1 -B` (FM broadcast)
- Exporta a CSV para graficar con gnuplot, Python, o herramientas de análisis
- Usado en Sparrow-WiFi para el overlay de espectro de 2.4/5 GHz
- Útil para detección de interferencia, mapeo de espectro, y auditoría de ocupación de bandas

**Herramientas adicionales del paquete hackrf:**
- `hackrf_info` — info del dispositivo conectado
- `hackrf_spiflash` — flashea el firmware del HackRF
- `hackrf_debug` — debug de hardware

**Puerto en DragonOS:** `/usr/bin/` (instalados globalmente)

---

### 7.5 Portapack Mayhem

**¿Qué es?** PortaPack es un módulo que se conecta al HackRF One y le agrega: pantalla LCD a color, tarjeta SD, batería integrada, y botones de navegación — convirtiéndolo en un dispositivo standalone completamente portátil sin necesidad de PC. Mayhem es el firmware comunitario (fork del original Havoc) con más de 100 aplicaciones, actualizaciones nightly, y es el estándar de facto para el PortaPack. DragonOS incluye los archivos de firmware de Mayhem en `/usr/src/firmware/mayhem/`.

**Variantes de hardware:**
- H1 — original, básico
- H2 / H2+ — más común, con batería integrada
- H4M — el más nuevo (2024), pantalla más grande, micrófono integrado, todo en un solo caso

**¿Qué podés hacer con Mayhem?**

*Recepción y análisis:*
- Spectrum analyzer, waterfall display
- ADS-B — mapa de aviones en pantalla
- ACARS, POCSAG, TPMS (presión de neumáticos), AIS (barcos)
- FM/AM/SSB receiver
- Bluetooth sniffer básico
- Sondas de radio — detección de radiosondas meteorológicas

*Transmisión y replay:*
- Signal replay: capturás cualquier señal a la SD y la retransmitís
- Flipper TX: transmite archivos `.sub` del Flipper Zero directamente
- Key fob tools: captura y replay de mandos de garaje, controles remoto, etc.
- GPS spoofer integrado
- FM transmitter

*Utilidades:*
- File manager en pantalla
- MayhemHub: interfaz web cuando está conectado a PC por USB
- Scripting (.ppsc scripts)
- Actualización de firmware over-the-air desde la web

**La comparación con Flipper Zero:** Mayhem+PortaPack cubre frecuencias de 1 MHz a 6 GHz (vs Flipper que solo llega a ~1 GHz) con mayor potencia y capacidad analítica. Flipper es más fácil de usar y más portátil; PortaPack H4M es más potente y capaz pero más grande y costoso (~$300-400 el combo).

**Puerto en DragonOS:** `/usr/src/firmware/mayhem/` (archivos .bin para flashear)  
**GitHub:** `github.com/portapack-mayhem/mayhem-firmware`

---

## 8. Decodificadores misceláneos

> *Protocolos específicos con decodificadores dedicados. Cada uno resuelve un problema concreto: ubicación de barcos, mensajes de buscapersonas, imágenes por radio, telemetría de globos meteorológicos.*

### 8.1 Frontends / visualización aplicable

| Herramienta | Rol en esta categoría |
|---|---|
| **GQRX** | Recepción base para la mayoría de estos protocolos en VHF/UHF |
| **SDR++** | Plugins integrados para AIS, POCSAG, APRS |
| **SDRAngel** | Demoduladores para múltiples protocolos misceláneos |
| **OpenWebRX+** | Decodificación web de FT8, WSPR, APRS, AIS, POCSAG entre otros |

### 8.2 APRS — Direwolf / multimon-ng

**¿Qué es?** APRS (Automatic Packet Reporting System) es una red de telemetría y mensajería en tiempo real sobre radio AX.25 VHF. Los nodos transmiten paquetes que contienen posición GPS, mensajes de texto, telemetría de sensores, datos meteorológicos, y alertas de emergencia. La red es global y descentralizada — cualquier radio con APRS puede comunicarse con el sistema completo. Se ve en `aprs.fi`.

**Direwolf** es el TNC (Terminal Node Controller) por software de referencia — reemplaza el hardware TNC dedicado usando la placa de sonido de la PC. Con un RTL-SDR como fuente de audio puede funcionar completamente en modo recepción.

**¿Qué podés hacer con él?**
- **IGate (pasarela internet):** recibir paquetes APRS de radio local y subirlos a la red APRS-IS (aprs.fi) — contribuís a la cobertura global
- **Digipeater:** retransmitir paquetes para extender alcance — útil en áreas con poca infraestructura
- **Tracker:** si tenés licencia y un radio TX, transmitir tu posición GPS en tiempo real
- **Monitoreo:** ver actividad de estaciones locales, objetos meteorológicos, vehículos de emergencia que usen APRS
- Integración con aplicaciones externas via KISS o AGWPE (Xastir, APRSIS32, etc.)

**multimon-ng** es el decodificador multiprotocolo más básico — toma audio y decodifica APRS/AFSK1200 entre muchos otros protocolos (POCSAG, FLEX, EAS, DTMF). Más simple que Direwolf pero sin capacidad de transmisión ni TNC completo.

**Flujo SDR:** `RTL-SDR` → `rtl_fm -f 144.390M -s 22050` → pipe de audio → `direwolf` (decodificación en tiempo real)

**Frecuencia en Costa Rica:** 144.390 MHz (IARU Región 2 — Américas). La actividad en CR es limitada pero existe, especialmente durante emergencias y eventos de la Sección Costa Rica del ARRL.

**Hardware:** RTL-SDR + antena VHF. Para IGate completo 24/7: Raspberry Pi + RTL-SDR.

**GitHub Direwolf:** `github.com/wb2osz/direwolf`  
**multimon-ng:** disponible en paquetes Debian

---

### 8.3 AIS — barcos

**¿Qué es?** AIS (Automatic Identification System) es el equivalente marino de ADS-B. Todo barco comercial de más de 300 toneladas bruta está obligado por SOLAS (IMO) a transmitir continuamente su identidad MMSI, nombre, posición GPS, velocidad, rumbo, tipo de carga, y destino. Transmite en dos canales VHF marítimos simultáneamente usando modulación GMSK.

**¿Qué podés hacer con él?**
- Ver en tiempo real todos los barcos comerciales en el Océano Pacífico y Mar Caribe cercanos a Costa Rica
- Obtener identidad, destino, tipo de barco y ETA de cualquier vessel en rango
- Alimentar plataformas como MarineTraffic o VesselFinder con datos propios
- Detectar barcos que desactivan AIS deliberadamente (actividad sospechosa)
- Monitorear tráfico en el Canal de Panamá desde tierra costarricense (con buena antena y elevación)

**Frecuencias:** 161.975 MHz (Canal 87B) y 162.025 MHz (Canal 88B) — ambos simultáneos.

**Hardware:** RTL-SDR + antena VHF omnidireccional. Desde Guanacaste con antena en altura, podés recibir barcos a 50-70 km mar adentro.

**Decodificadores en DragonOS:**
- OpenWebRX+ — AIS integrado, mapa en browser
- SDR++ — plugin AIS nativo
- `rtl-ais` — decodificador CLI específico para AIS
- `multimon-ng` — también decodifica AIS básico

**Dato:** AIS Clase B (barcos pequeños, yates) también existe pero es opcional. Ferries y transbordadores costarricenses deberían transmitir AIS Clase A.

---

### 8.4 POCSAG / FLEX — mensajería de buscapersonas

**¿Qué es?** POCSAG (Post Office Code Standardisation Advisory Group) y FLEX (Motorola) son los protocolos de mensajería digital unidireccional usados en buscapersonas (pagers). A pesar de que los pagers parecen tecnología del siglo pasado, siguen activos en hospitales, servicios de emergencias, y utilities — principalmente porque son más confiables que los celulares en escenarios de desastre y tienen mayor penetración en edificios.

**¿Qué podés hacer con él?**
- Recibir mensajes de texto transmitidos a buscapersonas en el rango de frecuencias local
- En contextos hospitalarios: mensajes de nurse call, alertas de código, llamados de guardia
- En emergencias: mensajes de coordinación de Cruz Roja, bomberos, servicios de salud
- Análisis de protocolos: cómo funciona un sistema de paging, estructura de mensajes POCSAG

**Frecuencias:** variable por país y operador, generalmente 152–158 MHz (América), también 929/931 MHz en EE.UU. En CR dependé de los sistemas activos locales — requiere barrido previo.

**Hardware:** RTL-SDR + antena VHF/UHF según la frecuencia local.

**Decodificador:** `multimon-ng` es el estándar en Linux:
```bash
rtl_fm -f 152.230M -s 22050 | multimon-ng -t raw -a POCSAG512 -a POCSAG1200 -a FLEX -
```

**Nota ética:** los mensajes de pager viajan sin encriptación. En muchos países recibir es legal (recepción pasiva), pero usar o publicar el contenido de mensajes ajenos puede violar leyes de privacidad. En hospitales especialmente el contenido puede ser sensible (datos de pacientes). Actividad de investigación y monitoreo propio.

---

### 8.5 SSTV — imágenes por radio

**¿Qué es?** SSTV (Slow Scan Television) es un método para transmitir imágenes estáticas por radio. Es extremadamente lento comparado con TV digital — una imagen tarda entre 8 y 138 segundos en transmitirse, dependiendo del modo. Se usa principalmente en radio amateur HF y VHF como actividad recreativa, y la ISS (Estación Espacial Internacional) transmite SSTV periódicamente como actividad educativa en 145.800 MHz.

**¿Qué podés hacer con él?**
- Recibir imágenes transmitidas por radioaficionados en HF (14.230 MHz, 21.340 MHz)
- Capturar las transmisiones SSTV de la ISS cuando pasa sobre Costa Rica (~cada 90 min)
- Enviar imágenes propias si tenés licencia y radio TX (con software como QSSTV)

**Modos más comunes:** Martin M1 (alta calidad, 114s), Scottie S1 (alta calidad, 110s), Robot 36 (color rápido, 36s), PD180 (alta resolución, 187s)

**Flujo SDR:** RTL-SDR (modo direct sampling para HF, o antena VHF para ISS) → GQRX/SDR++ → audio USB demodulado → QSSTV (Linux) para decodificación y visualización de imagen.

**La ISS y SSTV:** cada pocos meses la ISS activa SSTV durante 1-2 días. Podés recibirla desde cualquier lugar con vista al cielo con un RTL-SDR y una antena VHF simple. El timing se publica en amsat.org. Desde Atenas la ISS pasa con buena elevación varias veces al día.

**Decodificador en DragonOS:** QSSTV (menú de aplicaciones)

---

### 8.6 Radiosondas — globos meteorológicos

**¿Qué es?** Las radiosondas son pequeños paquetes de sensores que los servicios meteorológicos atan a globos de helio y lanzan dos veces al día (00 UTC y 12 UTC) para medir el perfil atmosférico vertical: temperatura, humedad, presión, viento y posición GPS en tiempo real hasta ~35 km de altitud. Transmiten en la banda meteorológica 400-406 MHz y se descartan después del vuelo — solo una fracción es recuperada.

**¿Qué podés hacer con él?**
- Recibir datos meteorológicos en tiempo real de globos lanzados por el IMN u otros servicios
- Ver la trayectoria en tiempo real y predecir dónde va a caer el globo (para caza)
- Contribuir datos a la red global sondehub.org
- Recuperar la radiosonda caída — contiene electrónica interesante (GPS Ublox, MCU STM32)

**Lanzamientos relevantes para CR:**
- IMN lanza desde Aeropuerto Juan Santamaría (~2 por día)
- También podés recibir sondas de Colombia, Panamá, o incluso Cuba en condiciones favorables de propagación

**Tipos de sonda más comunes:**
- RS41 (Vaisala) — el más común globalmente, protocolos bien documentados
- M10/M20 (Meteomodem) — usados en Francia y algunas estaciones americanas
- DFM09/DFM17 (GRAW) — más raro

**`radiosonde_auto_rx`:** el sistema automático completo — escanea 400-406 MHz, identifica sondas automáticamente, las decodifica, y sube datos a sondehub.org/radiosondy.info en tiempo real. Corriendo 24/7 en una Raspberry Pi con RTL-SDR constituye una estación meteorológica amateur.

**Hardware:** RTL-SDR + antena VHF omnidireccional o yagi 403 MHz.

**GitHub:** `github.com/projecthorus/radiosonde_auto_rx`

---

### 8.7 NRSC5 — radio HD

**¿Qué es?** NRSC-5 es el estándar de radio digital implementado en EE.UU. bajo el nombre comercial "HD Radio". Transmite datos digitales comprimidos en dos bandas laterales adyacentes a la señal FM analógica existente — el receptor puede escuchar la versión digital de mayor calidad cuando la señal es buena. El protocolo estuvo cerrado por años hasta que fue reverse-engineered en 2017.

**¿Qué podés hacer con él?**
- Decodificar audio digital HD Radio de emisoras que transmitan en EE.UU.
- Recibir metadatos de programación: nombre del artista, canción, álbum, y en algunas emisoras imágenes de portada
- Recibir datos de tráfico y clima embebidos en la señal HD Radio

**Relevancia para Costa Rica:** prácticamente nula — HD Radio solo existe en EE.UU. y algunos mercados donde Ibiquity tiene licencia. Las emisoras costarricenses usan FM analógico estándar. Sin embargo, es un caso de estudio interesante de reverse engineering de protocolo propietario.

**Hardware:** RTL-SDR + antena FM. Funciona directamente en las frecuencias FM (87.5–108 MHz).

**En DragonOS:** `nrsc5` en `/usr/src/` — incluye GUI y versión CLI.

**GitHub:** `github.com/theori-io/nrsc5`

---

### 8.8 JS8Call / WSJT-X — modos débiles HF

**¿Qué es?** WSJT-X implementa los modos de radio de señal extremadamente débil diseñados por Joe Taylor (K1JT, Nobel de Física 1993). FT8 es el modo estrella — permite comunicación digital confiable con señales 20 dB por debajo del nivel de ruido del receptor, usando ciclos estrictos de 15 segundos. JS8Call extiende FT8 para mensajería completa tipo SMS sobre radio HF, haciendo posible comunicación de texto en condiciones donde ninguna otra modalidad funcionaría.

**WSJT-X — modos disponibles:**
- **FT8** — el más popular: ciclos de 15s, -20 dB SNR, intercambia callsign y grid locator
- **FT4** — más rápido (7.5s), menos sensible, para concursos
- **JT65** — precursor de FT8, 60s por ciclo, más lento
- **WSPR** — Weak Signal Propagation Reporter, beacons de muy baja potencia para mapear propagación HF
- **MSK144** — para comunicaciones por scatter meteórico (rebote en estelas de meteoros)

**JS8Call:**
- Basado en FT8 pero permite mensajes de texto completos
- No requiere licencia para RECIBIR — solo para transmitir
- Ideal para redes de emergencia HF digitales
- Compatible con APRS via JS8Call gateway

**¿Qué podés hacer con SDR?**
- Recibir FT8/FT4/WSPR en cualquier banda HF — monitorear la actividad de radioaficionados en todo el mundo
- Ver en tiempo real en PSKReporter qué estaciones están siendo recibidas desde tu ubicación
- Analizar propagación HF — cuántas bandas abiertas hay y hacia qué partes del mundo
- Para TRANSMITIR se requiere licencia de radioaficionado (LIC en CR) y radio con TX

**Frecuencias principales FT8 (bandas más activas):**
- 14.074 MHz (20m) — la banda más activa, abierta la mayor parte del día
- 7.074 MHz (40m) — activo de noche, propagación regional/continental
- 21.074 MHz (15m) — activo en picos solares, DX intercontinental
- 144.174 MHz (2m) — FT8 en VHF, comunicaciones locales/ES

**Flujo SDR → WSJT-X:** RTL-SDR en modo direct sampling → GQRX (sintonizado en banda HF, demodulación USB) → audio pipe → WSJT-X (decodificación FT8)

**Hardware:** RTL-SDR v3 en direct sampling Q para HF (sin upconverter, con limitaciones de sensibilidad). Para mejor recepción: SDRplay RSP1A o Airspy HF+ con antena de hilo largo.

**Puerto en DragonOS:** WSJT-X y JS8Call en menú de aplicaciones

---

## Apéndice A — Hardware compatible

| Hardware | Rango de frecuencia | Tx/Rx | Casos de uso ideales |
|---|---|---|---|
| RTL-SDR v3/v4 | 500 kHz – 1766 MHz | Solo Rx | ADS-B, ACARS, APRS, AIS, NOAA, FM |
| Airspy R2 | 24 – 1800 MHz | Solo Rx | Todo lo de RTL con mejor calidad |
| Airspy HF+ | 0.5 kHz – 31 MHz / 60 – 260 MHz | Solo Rx | HF-DL, HF, shortwave |
| HackRF One | 1 MHz – 6 GHz | Tx+Rx (half duplex) | Transmisión, replay, análisis |
| LimeSDR | 100 kHz – 3.8 GHz | Tx+Rx (full duplex) | LTE, GSM, 5G research |
| BladeRF | 300 MHz – 3.8 GHz | Tx+Rx (full duplex) | Telecom research |

---

## Apéndice B — Frecuencias de referencia rápida

| Señal | Frecuencia | Protocolo | Herramienta |
|---|---|---|---|
| ADS-B (aviones posición) | 1090 MHz | ADS-B | dump1090 |
| ACARS (mensajes avión-tierra) | 129.125 / 130.025 / 131.550 MHz | ACARS | acarsdec |
| VDL Mode 2 | 136.900 MHz | VDL2 | dumpvdl2 |
| L-Band Inmarsat | 1545 / 1555 MHz | ACARS/voice | Jaero |
| Iridium | 1616 – 1626.5 MHz | Iridium | iridium-toolkit |
| APRS | 144.390 MHz (IARU R2) | AX.25 | Direwolf |
| AIS (barcos) | 161.975 / 162.025 MHz | AIS | varios |
| NOAA weather sat | 137.100 / 137.620 / 137.912 MHz | APT | SatDump |
| POCSAG / pager | 152 – 158 MHz (varía por país) | POCSAG | multimon-ng |
| FM broadcast | 87.5 – 108 MHz | WFM | GQRX / SDR++ |
| GSM 850 (downlink CR) | 869 – 894 MHz | GSM | gr-gsm |
| LTE B4 (downlink CR) | 2110 – 2155 MHz | LTE | srsRAN |

---

## Apéndice C — Glosario de siglas

| Sigla | Significado | Contexto |
|---|---|---|
| **ACARS** | Aircraft Communications Addressing and Reporting System | Mensajería digital avión-tierra, VHF y satélite |
| **ADS-B** | Automatic Dependent Surveillance–Broadcast | Posición GPS que transmite cada avión en 1090 MHz |
| **AIS** | Automatic Identification System | Identificación y posición de embarcaciones marítimas |
| **AM** | Amplitude Modulation | Modulación en amplitud, usada en HF y aviación |
| **APRS** | Automatic Packet Reporting System | Red de telemetría/mensajería sobre radio AX.25 |
| **APT** | Automatic Picture Transmission | Protocolo de imagen de satélites NOAA meteorológicos |
| **AX.25** | Amateur X.25 | Protocolo de enlace de datos usado en radio amateur |
| **BTS** | Base Transceiver Station | Antena/estación base de red celular |
| **C-Band** | Frequency band ~4–8 GHz | Banda usada por satélites geoestacionarios de comunicación |
| **CCIR** | Comité Consultatif International des Radiocommunications | Organismo predecesor del ITU-R, define estándares de radio |
| **DSP** | Digital Signal Processing | Procesamiento digital de señales |
| **DSD** | Digital Speech Decoder | Decodificador de voz digital P25, DMR, NXDN, etc. |
| **FM** | Frequency Modulation | Modulación en frecuencia (radio FM broadcast, VHF/UHF) |
| **FLEX** | — | Protocolo de mensajería de buscapersonas de alta velocidad |
| **FSK** | Frequency Shift Keying | Modulación digital por desplazamiento de frecuencia |
| **GNSS** | Global Navigation Satellite System | Sistema global de navegación por satélite (GPS, GLONASS, Galileo) |
| **GPS** | Global Positioning System | Sistema de posicionamiento satelital de EE.UU. |
| **GSM** | Global System for Mobile Communications | Estándar de red celular 2G |
| **HF** | High Frequency | Banda de 3–30 MHz (shortwave / onda corta) |
| **HF-DL** | HF Data Link | Protocolo ACARS sobre HF para vuelos transoceánicos |
| **HRPT** | High Resolution Picture Transmission | Imagen de alta resolución de satélites meteorológicos |
| **IQ** | In-phase / Quadrature | Representación de señales RF en SDR (parte real e imaginaria) |
| **ITU** | International Telecommunication Union | Organismo de la ONU que regula el espectro radioeléctrico |
| **L-Band** | Frequency band ~1–2 GHz | Banda usada por Inmarsat, Iridium, GPS, aviación satcom |
| **LEO** | Low Earth Orbit | Órbita terrestre baja (~200–2000 km), usada por Iridium, NOAA |
| **LNA** | Low Noise Amplifier | Amplificador de bajo ruido, mejora sensibilidad del receptor |
| **LoRa** | Long Range | Modulación de radio de largo alcance, usada en IoT y Meshtastic |
| **LTE** | Long Term Evolution | Estándar de red celular 4G |
| **MPEG** | Moving Picture Experts Group | Estándar de compresión de audio/video |
| **NFM** | Narrow FM | FM angosta, usada en radio VHF/UHF de comunicaciones |
| **NRSC-5** | National Radio Systems Committee–5 | Estándar de radio HD (digital) en banda FM de EE.UU. |
| **NXDN** | — | Protocolo de radio digital de Icom/Kenwood |
| **OsmocomBB** | Open Source Mobile Communications Baseband | Firmware open source para chipsets de celulares 2G |
| **P25** | Project 25 / APCO-25 | Estándar de radio digital para servicios de emergencia de EE.UU. |
| **POCSAG** | Post Office Code Standardisation Advisory Group | Protocolo de mensajería de buscapersonas unidireccional |
| **PSK** | Phase Shift Keying | Modulación digital por desplazamiento de fase |
| **QAM** | Quadrature Amplitude Modulation | Modulación de amplitud en cuadratura, usada en LTE/cable |
| **RF** | Radio Frequency | Frecuencias de radio (generalmente 3 kHz – 300 GHz) |
| **RTL-SDR** | Realtek SDR | Dongle SDR basado en chip Realtek RTL2832U, muy económico |
| **RX** | Receive | Modo recepción |
| **S-Band** | Frequency band ~2–4 GHz | Banda usada por satélites meteorológicos de alta resolución |
| **SDR** | Software Defined Radio | Radio definida por software — el hardware es mínimo, la lógica es código |
| **SIGINT** | Signals Intelligence | Inteligencia de señales, interceptación y análisis de comunicaciones RF |
| **SNR** | Signal to Noise Ratio | Relación señal a ruido, métrica de calidad de recepción |
| **SSTV** | Slow Scan Television | Transmisión de imágenes estáticas por radio, muy lenta |
| **SSB** | Single Sideband | Modulación de banda lateral única, eficiente para HF de voz |
| **TCP** | Transmission Control Protocol | Protocolo de red orientado a conexión |
| **TETRA** | Terrestrial Trunked Radio | Estándar de radio digital troncal europeo (policía, emergencias) |
| **TX** | Transmit | Modo transmisión |
| **UHF** | Ultra High Frequency | Banda de 300 MHz – 3 GHz |
| **USRP** | Universal Software Radio Peripheral | Hardware SDR de alta gama de Ettus Research / NI |
| **VDL** | VHF Digital Link | Enlace digital VHF para aviación, versión digital de ACARS |
| **VHF** | Very High Frequency | Banda de 30–300 MHz |
| **VFO** | Variable Frequency Oscillator | Sintonizador de frecuencia variable, término heredado de radio analógica |
| **WFM** | Wide FM | FM ancha, usada en radio FM broadcast (87.5–108 MHz) |
| **WSJT-X** | Weak Signal Joe Taylor | Software para modos de señal débil HF (FT8, FT4, JT65, etc.) |

---

*Este documento es una guía viva. Cada sección se actualiza a medida que se investiga cada herramienta en profundidad.*
