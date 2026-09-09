# nodeIO_Hardware — Nodo IO LoRa (Aysafi)

Placa portadora (*carrier*) de **E/S remota industrial** para el módulo
**Heltec WiFi LoRa 32 V3** (SoC **ESP32‑S3FN8** + radio **Semtech SX1262**).
Este repositorio contiene el **diseño electrónico** (esquema, PCB, BOM y salidas de
fabricación) del producto `NODE-IO-2405`.

| | |
|---|---|
| **Versión de hardware** | V1.0 |
| **Esquema** | `SCH_Schematic1_2026-09-09.pdf` (creado 2026‑08‑29, actualizado 2026‑09‑08) |
| **PCB / Gerbers** | `PCB1_2026-08-30` · 89,845 × 55,795 mm · 2 capas |
| **EDA** | EasyEDA Pro / LCEDA Pro (proyecto `Remote_nodeIO_Hardware.eprj`) |
| **Firmware** | [`nodeIO`](../../nodeIO) (nodo) · [`nodeIO_master`](../../nodeIO_master) (pasarela LoRa↔Modbus) |
| **Manual electrónico** | [`MANUAL_ELECTRONICO_nodeIO.md`](MANUAL_ELECTRONICO_nodeIO.md) — descripción bloque por bloque, cálculos, bring‑up y **22 mejoras priorizadas** |

---

## 1. Qué hace la placa

Un nodo gobernado por radio **LoRa 915 MHz** que atiende a un maestro y expone:

- **4 entradas digitales** opto‑aisladas (contacto seco / 24 V de campo).
- **4 entradas analógicas** de lazo **4–20 mA** (aisladas por opto).
- **4 salidas de relé** (nivel lógico 3,3 V a un módulo de relés **externo**).
- **1 puerto RS‑485** (MAX485, semidúplex) — usado por `nodeIO_master` para Modbus RTU.
- **Fuente aislada** 24 V → 5 V (RECOM RI3‑2405S) + entrada alternativa de 5 V.
- HMI local: OLED del módulo, 2 pulsadores de usuario, pulsador PRG, LED.

La configuración (dirección LoRa, canal, relés) se hace por **portal cautivo WiFi** del
firmware; no requiere tocar el hardware.

---

## 2. Estructura del repositorio

| Archivo / carpeta | Contenido | Editar con |
|---|---|---|
| `Remote_nodeIO_Hardware.eprj` | **Proyecto EasyEDA Pro** (base de datos SQLite: esquema + PCB + librerías). Fuente de verdad. | EasyEDA Pro / LCEDA Pro — **no editar a mano ni hacer *merge* de git** |
| `Remote_nodeIO_Hardware_backup/` | Instantáneas `.zip` del proyecto (`v11`…`v93`), exportadas por la propia EDA | — (histórico) |
| `SCH_Schematic1_2026-09-09.pdf` | Esquema exportado (revisión legible) | regenerar al cambiar el esquema |
| `PCB_PCB1_2026-08-30.pdf` | PCB exportado (capas, taladros, dimensiones) | regenerar al re‑rutar |
| `Gerber_PCB1_2026-08-30_NODEIO_AYSAFI.zip` | **Gerbers + Excellon** para fabricación | regenerar al re‑rutar |
| `BOM_Board1_Schematic1_2026-09-08.xlsx` | **BOM** desde el esquema (lista maestra) | regenerar al cambiar componentes |
| `BOM_Board1_PCB1_2026-08-30.xlsx` | BOM desde el PCB (debe cuadrar con la del esquema) | regenerar |
| `PickAndPlace_PCB1_2026-08-30.xlsx` | Centroides para **ensamblado SMT** | regenerar al mover componentes |
| `3D_PCB1_2026-08-30.step` / `*.png` | Modelo 3D y renders para revisión mecánica / encaje en caja | regenerar |
| `Info_PCB_PCB1_2026-08-30.txt` | Resumen de PCB (tamaño, capas, nets, vías, longitud de pista) | regenerar |
| `heltecLora.png` | Huella / referencia del módulo Heltec | — |
| `MANUAL_ELECTRONICO_nodeIO.md` | Manual de ingeniería + hoja de mejoras | mantener junto con el diseño |
| `README.md` | Este documento | — |

> **Sobre `.eprj`:** es un fichero binario (SQLite). Git no puede fusionarlo. Trabaja
> siempre desde una única copia, y antes de un cambio grande exporta una instantánea a
> `Remote_nodeIO_Hardware_backup/` (menú *Archivo → Copia de seguridad* de la EDA).

---

## 3. Herramienta de diseño

- **EasyEDA Pro** (escritorio) o **LCEDA Pro**. Abrir `Archivo → Abrir proyecto →
  Remote_nodeIO_Hardware.eprj`.
- Librerías de componentes: las estándar de LCSC/JLCPCB (los `Manufacturer Part` y
  `LCSC` van en la BOM). El módulo Heltec se representa como **zócalo de 2×18 pines**
  (`H1`, `H2`), no se suelda directamente.

---

## 4. Especificaciones actuales (V1.0)

| Parámetro | Valor | Notas |
|---|---|---|
| Dimensiones PCB | 89,845 × 55,795 mm | 4 taladros de montaje |
| Capas | 2 (cobre) | 30 vías · 1 458 mm de pista |
| Componentes | 38 (32 top / 6 bottom) | 11 huellas distintas |
| Alimentación principal | **24 V DC ±10 %** (borne U5) → RI3‑2405S → 5 V | ⚠️ la serigrafía dice “12–32 V”; el RI3‑2405S sólo admite **21,6–26,4 V** y **no es regulado** — ver mejora §9.1 del manual |
| Alimentación alternativa | 5 V DC (borne U35) | no aislada; sin ORing con la salida del DC‑DC |
| Aislamiento galvánico | ~3 kVDC (sólo a través de U31) | campo ↔ electrónica |
| Entradas digitales | 4 × opto LTV‑247, R serie 2K2 | activo a masa tras opto; `GPIO38–41` |
| Entradas analógicas | 4 × 4–20 mA, opto LTV‑247, *burden* 165 Ω | 20 mA → 3,30 V (fondo de escala exacto); `GPIO2–5`, ADC1 |
| Salidas de relé | 4 × nivel lógico 3,3 V a conector JST‑XH 6P (U7) | **sin driver on‑board**; `GPIO33/34/45/46` (45/46 son *strapping*) |
| RS‑485 | MAX485ED, terminación fija 120 Ω (R16) | datos en `GPIO43/44` = **UART0 compartido con el CP2102 USB** |
| Radio | SX1262 en el módulo · 915 MHz / BW125 / SF9 / CR4/5 / sync 0x34 / 14 dBm | parámetros por defecto (portal los reescribe) |
| Conectores de campo | bornes enchufables DB125 2,54 mm | relés en JST‑XH 2,54 (revisar, §9.13) |
| Puntos de prueba | 13 (`TP1`…`TP17`) | |

El **mapa de pines completo** y los **cálculos** están en
[`MANUAL_ELECTRONICO_nodeIO.md`](MANUAL_ELECTRONICO_nodeIO.md) §7 y §3–§6.

---

## 5. Regenerar los entregables de fabricación

Tras **cualquier** cambio en el esquema o el PCB, regenera **todo** el juego de salidas
desde EasyEDA Pro y renómbralas con la fecha (`AAAA-MM-DD`):

| Salida | Menú EasyEDA Pro | Archivo destino |
|---|---|---|
| Esquema PDF | Esquema → `Archivo → Exportar → PDF` | `SCH_Schematic1_<fecha>.pdf` |
| PCB PDF | PCB → `Archivo → Exportar → PDF` | `PCB_PCB1_<fecha>.pdf` |
| Gerbers + taladros | PCB → `Fabricación → Salida de Gerber` | `Gerber_PCB1_<fecha>_NODEIO_AYSAFI.zip` |
| Pick & Place | PCB → `Fabricación → Posición de componentes` | `PickAndPlace_PCB1_<fecha>.xlsx` |
| BOM (esquema) | Esquema → `Fabricación → BOM` | `BOM_Board1_Schematic1_<fecha>.xlsx` |
| BOM (PCB) | PCB → `Fabricación → BOM` | `BOM_Board1_PCB1_<fecha>.xlsx` |
| Modelo 3D | PCB → `Ver → 3D → Exportar STEP` | `3D_PCB1_<fecha>.step` + PNG |
| Info de PCB | PCB → `Ver → Información del PCB` | `Info_PCB_PCB1_<fecha>.txt` |

Verifica que la **BOM del esquema y la del PCB cuadran** (misma cantidad y referencias).

---

## 6. Fabricación y ensamblado

- **PCB:** 2 capas, acabado y espesor estándar JLCPCB. `Info_PCB` lista tamaño, vías y
  longitud de pista.
- **SMT:** 32 componentes en cara superior, 6 en inferior. Usar `PickAndPlace_*.xlsx` +
  `BOM_*_PCB1_*.xlsx`. Partes con referencia `LCSC` en la BOM.
- **THT / manual:** bornes DB125, conector JST‑XH (U7), zócalos del módulo (`H1`/`H2`),
  pulsadores, DC‑DC `U31`.
- **Módulo Heltec:** se **inserta en el zócalo** tras el ensamblado; no va al horno.
- **Antena:** conectar la antena de 915 MHz **antes** de energizar (proteger el PA del
  SX1262). Mantenerla alejada del DC‑DC y del metal de la caja.

---

## 7. Relación con el firmware

| Repo | Rol | Comparte con el hardware |
|---|---|---|
| [`nodeIO`](../../nodeIO) | Nodo/responder LoRa. Portal cautivo, protocolo `RD/WR/WP/PING`. | `src/io.{h,cpp}` = mapa de pines de esta placa |
| [`nodeIO_master`](../../nodeIO_master) | Pasarela **LoRa ↔ Modbus** (TCP :502 / RTU RS‑485). | mismo `src/io.{h,cpp}` **byte a byte** |
| `ORCHESTRATION/REGISTER_MAP.md` | Contrato del mapa Modbus (MAPA A) que publica el master | — |

**Contrato hardware ↔ firmware:** cualquier re‑mapa de pines en este diseño **debe**
reflejarse en `nodeIO/src/io.h` **y** `nodeIO_master/src/io.h` (se mantienen idénticos a
mano). Hoy existen inconsistencias documentadas en el manual (§6 y §9.4): los pines de
RS‑485 del esquema no coinciden con lo que describe el README del master.

---

## 8. Estado actual y limitaciones conocidas

La V1.0 es **funcional para banco / piloto**. Limitaciones que conviene conocer antes de
desplegar en planta (detalle y solución en el manual, §9):

| # | Limitación | Impacto |
|---|---|---|
| 9.1 | Rango de entrada mal rotulado; RI3‑2405S sólo 21,6–26,4 V y no regulado | No arranca fuera de 24 V ±10 %; 5 V pobre con picos de WiFi |
| 9.2 | Relés 3 y 4 en `GPIO45/46` (*strapping*) | Una placa de relés externa puede impedir arranque/flasheo |
| 9.3 | Sin driver de relé ni *pull‑down* | Salidas flotantes en reset → posible “castañeteo” |
| 9.4 | RS‑485 en UART0 (compartido con el CP2102 USB) | La consola y el bus se interfieren |
| 9.5 | RS‑485 sin *bias* de reposo; terminación fija | Bus multipunto poco fiable |
| 9.6 | “Aislamiento” analógico con opto de fototransistor (LTV‑247) | Medida 4–20 mA no lineal ni repetible |
| 9.10 | Sin fusible, anti‑inversión ni TVS en la entrada | Vulnerable en entorno 24 V industrial |

---

## 9. Hoja de ruta de hardware

Agrupación de las **22 mejoras** del manual en entregas. Cada punto enlaza a su sección
en [`MANUAL_ELECTRONICO_nodeIO.md`](MANUAL_ELECTRONICO_nodeIO.md).

### Rev B — bloqueantes de arranque y de campo
- **§9.1** DC‑DC aislado de **entrada ancha 9–36 V y regulado** (≥ 5 W).
- **§9.2** Salidas de relé fuera de los pines de *strapping* → registro `TPIC6B595` / `74HC595`.
- **§9.3** Driver de relé **on‑board** (ULN2803A / NMOS + diodo volante + LED) y *pull‑down* 100 k.
- **§9.4** RS‑485 a **GPIO6 / 7 / 26**; consola de depuración en UART0 dedicado o USB‑CDC.
- **§9.5** RS‑485: *bias* 680 Ω, terminación **por jumper**, TVS en A/B.
- **§9.10** Entrada: PTC/fusible + protección de inversión (P‑FET) + TVS `SMBJ33A` + caps del DC‑DC.
- **§9.18** Corregir serigrafía de `U4`/`U6` y marcar polaridad de AI.

### Rev C — exactitud y robustez
- **§9.6** Aislamiento analógico **lineal** (`ISO224`/`AMC1200`) o **ADC I²C aislado** (`ADS1115` + `ISO1540`).
- **§9.7** *Burden* 150 Ω + detección **NAMUR NE43** (rotura de lazo / sobre‑rango) en firmware.
- **§9.8** ADC externo de 16 bits (`ADS1115`) → libera `GPIO2–5`.
- **§9.9** Frontal DI de **corriente constante** (IEC 61131‑2 Tipo 1/2/3) + protección de inversión + RC.
- **§9.11** ORing de las dos entradas de alimentación (diodo ideal `LM66100`).
- **§9.12** PCB a **4 capas** con plano de masa continuo; keep‑out de antena.
- **§9.13** Bornes de relé aptos para campo + *creepage/clearance* (o declarar SELV).
- **§9.14** TVS en todos los bornes de campo + terminal de tierra de chasis.

### Backlog
- **§9.15** Más puntos de prueba + LED de estado bicolor.
- **§9.16** Filtro EMC de entrada (choque de modo común + X/Y) para marcado CE/FCC.
- **§9.17** Riel de 5 V con margen (DC‑DC 5–6 W, más *bulk*).
- **§9.19** *Flag* de inversión de relé en configuración (firmware).
- **§9.20** Fijar el número de parte del módulo (ESP32‑S3 **sin PSRAM**) para no chocar con `GPIO33–37`.
- **§9.21** Reasignar `SW2` fuera de `GPIO48` (LED integrado del módulo).
- **§9.22** Documentar que no se conecte batería al cargador del módulo.

Ver también la **propuesta de re‑mapa de pines para la revisión B** en el manual, §9.24.

---

## 10. Flujo de mantenimiento del diseño

Al modificar el hardware:

1. **Editar** el esquema/PCB en EasyEDA Pro sobre `Remote_nodeIO_Hardware.eprj`.
2. **Copia de seguridad**: exportar instantánea `.zip` a `Remote_nodeIO_Hardware_backup/`.
3. **Regenerar** todas las salidas (sección 5) con la fecha nueva; borrar las obsoletas.
4. **Actualizar** [`MANUAL_ELECTRONICO_nodeIO.md`](MANUAL_ELECTRONICO_nodeIO.md): mapa de
   pines, cálculos, BOM resumido, bring‑up y el estado de las mejoras.
5. **Sincronizar** si cambió algún pin: `nodeIO/src/io.h` **y** `nodeIO_master/src/io.h`.
6. **Subir la versión** de hardware (V1.0 → V1.1 / V2.0) en el esquema, en este README y
   en el manual. Añadir entrada al *changelog*.
7. **Commit** con mensaje `hw:` o `docs:` describiendo el cambio y su motivo.
8. Para una revisión de PCB nueva, etiquetar: `git tag hw-v1.1 && git push --tags`.

---

## 11. Changelog

| Fecha | Hito |
|---|---|
| 2026‑09‑09 | Manual electrónico + hoja de 22 mejoras (`MANUAL_ELECTRONICO_nodeIO.md`); este README |
| 2026‑09‑08 | Esquema actualizado; BOM `Schematic1` regenerada |
| 2026‑08‑30 | PCB1 rutado; Gerbers, Pick&Place, 3D y BOM de PCB |
| 2026‑08‑29 | Primer esquema completo; instantáneas `v11`…`v76` |

---

## 12. Licencia y contacto

Diseño propiedad de **Aysafi — Ingeniería y Tecnología**. Uso interno / bajo acuerdo.
Contacto: <asdrubal@aysafi.com>.
