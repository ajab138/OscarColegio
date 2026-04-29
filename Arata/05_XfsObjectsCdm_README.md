# XfsObjectsCdm

Biblioteca estática C++ que proporciona los **tipos de datos CDM** (Cash Dispenser Module) del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsObjectsCdm` define todas las clases C++ que representan estructuras de datos del estándar XFS CDM (versión 3.40):

- **Objetos de comandos** `getInfo` y `execute`: parámetros de entrada y resultados de salida para cada comando CDM.
- **Objetos de eventos**: estructuras empleadas en los eventos execute, service y user del servicio CDM.
- **Tipos enumerados** (`TCdm::*`): mapeo tipado y sin dependencias de los `#define` de `xfscdm.h` a `enum class` de C++11.
- **Serialización** bidireccional vía Protobuf (`proto::cdm::*`) e interfaz `IXfsSerializable`.

Es consumida por `XfsBaseCdm` y por las implementaciones concretas de dispensadores de efectivo.

## Arquitectura

Todos los objetos de datos residen en el namespace `obj::cdm` e implementan la interfaz `IXfsSerializable`:

```
IXfsSerializable      (XfsCoreSupport)
        │
        ▼
obj::cdm::<Clase>  ←──── proto::cdm::<Proto>   (XfsProtos / Protobuf)
```

Cada clase proporciona:

- Constructor por defecto, constructor de copia y operador de asignación.
- Operadores `==` y `!=`.
- `toString(stream, indent)` — representación textual para trazas/log.
- `serialize(buffer)` / `deserialize(buffer)` — serialización binaria vía Protobuf.
- `toProto()` / `fromProto()` / `setProto(proto)` — acceso directo al mensaje Protobuf subyacente.

## Clases de objetos (`obj::cdm::*`)

### Objetos de resultado getInfo

| Clase | Comando CDM | Descripción |
|---|---|---|
| `CdmStatus` | `Status` (301) | Estado del dispositivo: dispositivo, safe door, dispensador, stacker, posiciones, guid lights, posición del dispositivo. |
| `CdmCaps` | `Capabilities` (302) | Capacidades del dispositivo: tipo, obturador, áreas de retracción, posiciones, modos de exchange, etc. |
| `CdmCuInfo` | `CashUnitInfo` (303) | Colección de unidades de efectivo lógicas (`CdmCashUnit`) del dispensador. |
| `CdmCurrencyExp` | `CurrencyExp` (306) | Exponentes de moneda para el cálculo de importes. |
| `CdmMixType` | `MixTypes` (307) | Tipos de mezcla de billetes disponibles. |
| `CdmMixTable` | `MixTable` (308) | Tabla de mezcla: conjunto de filas `CdmMixRow`. |
| `CdmPresentStatus` | `PresentStatus` (309) | Estado de la presentación de billetes en la posición de salida. |
| `CdmBlacklist` | `GetBlacklist` (311) | Lista negra de billetes: colección de `CdmBlacklistElement`. |
| `CdmClassificationList` | `GetClassificationList` (313) | Lista de clasificación: colección de `CdmClassificationElement`. |
| `CdmTellerDetails` | `TellerInfo` (304) | Detalles y totales de cajero: colección de `CdmTellerTotals`. |
| `CdmItemInfo` | `GetItemInfo` (310) | Información de un billete individual. |
| `CdmAllItemsInfo` | `GetAllItemsInfo` (312) | Información de todos los billetes: colección de `CdmItemInfoAll`. |

### Parámetros de entrada getInfo

| Clase | Descripción |
|---|---|
| `CdmTellerInfo` | Parámetro de entrada para `TellerInfo`: identificador de cajero y moneda. |
| `CdmGetItemInfo` | Parámetro de entrada para `GetItemInfo`: número de ítem. |
| `CdmGetAllItemsInfo` | Parámetro de entrada para `GetAllItemsInfo`: número de ítems y tipo de información requerida. |

### Objetos de comandos execute

| Clase | Comando CDM | Descripción |
|---|---|---|
| `CdmDenominate` | `Denominate` (301) | Parámetros para calcular una denominación: tellerID, número de mezcla, denominación deseada. |
| `CdmDispense` | `Dispense` (302) | Parámetros para dispensar: tellerID, número de mezcla, posición, flag de presentación, denominación. |
| `CdmDenomination` | — | Denominación CDM: moneda, importe, conteo, valores por posición lógica, caja fuerte. Usado como parámetro y resultado en `Denominate` y `Dispense`. |
| `CdmCount` | `Count` (323) | Parámetros para contar billetes: unidades físicas a contar y posición de retracción. |
| `CdmRetract` | `Retract` (305) | Parámetros para retraer billetes: posición de salida y área/índice de retracción. |
| `CdmSetGuidLight` | `SetGuidanceLight` (324) | Control de luz de guía: número de luz y valor. |
| `CdmPowerSaveControl` | `PowerSaveControl` (325) | Control de ahorro de energía: tiempo máximo de espera. |
| `CdmPrepareDispense` | `PrepareDispense` (326) | Preparación del dispensador: acción a ejecutar. |
| `CdmCalibrate` | `CalibrateCashUnit` (315) | Parámetros de calibración de unidades de efectivo. |
| `CdmStartEx` | `StartExchange` (311) | Parámetros de inicio de ciclo de reposición. |
| `CdmSynchronizeCommand` | `SynchronizeCommand` (328) | Parámetros para sincronizar comandos. |
| `CdmTellerUpdate` | `SetTellerInfo` (309) | Actualización de información de cajero. |

### Objetos de eventos

| Clase | Evento CDM | Descripción |
|---|---|---|
| `CdmCuError` | `ExeeCashUnitError` (308) | Error en unidad de efectivo durante ejecución. |
| `CdmCountsChanged` | `SrveCountsChanged` (314) | Cambio de contadores en unidades de efectivo. |
| `CdmDevicePosition` | `SrveDevicePosition` (319) | Cambio de posición física del dispositivo. |
| `CdmIncompleteRetract` | `ExeeIncompleteRetract` (322) | Retracción incompleta: unidades físicas con billetes retenidos. |
| `CdmPowerSaveChange` | `SrvePowerSaveChange` (320) | Cambio de estado del modo ahorro de energía. |
| `CdmShutterStatusChanged` | `SrveShutterStatusChanged` (323) | Cambio de estado del obturador. |
| `CdmItemPosition` | `SrveMediaDetected` (317) | Posición detectada de billetes en el dispositivo. |
| `CdmItemInfoSummary` | `ExeeInfoAvailable` (321) | Resumen de información de ítem disponible. |

### Objetos auxiliares

| Clase | Descripción |
|---|---|
| `CdmCashUnit` | Unidad de efectivo lógica: tipo, moneda, contadores (inicial, actual, rechazado, dispensado, presentado, retraído), estado, unidades físicas. |
| `CdmPhCu` | Referencia abreviada a unidad física de efectivo: nombre, ID, contadores básicos. |
| `CdmPhysicalCu` | Unidad física de efectivo completa: nombre, ID, contadores extendidos, estado. |
| `CdmCountedPhysCu` | Unidad física contada durante un comando `Count`. |
| `CdmOutPos` | Posición de salida: tipo de posición y estado del obturador/transporte/stacker. |
| `CdmMixRow` | Fila de tabla de mezcla: importe, cantidades por denominación. |
| `CdmBlacklistElement` | Elemento de lista negra: moneda, valor, acción y datos de firma. |
| `CdmClassificationElement` | Elemento de lista de clasificación: moneda, valor y nivel. |
| `CdmSignature` | Firma de billete: moneda, valor y datos de firma. |
| `CdmTellerTotals` | Totales por moneda de un cajero: dispensado, presentado, retraído. |
| `CdmItemNumber` | Número de serie de ítem (billete). |
| `CdmItemNumberList` | Lista de números de serie. |
| `CdmItemInfoAll` | Información completa de un ítem: número, moneda, valor, orientación, estado, firma. |

## Tipos enumerados (`TCdm::*`)

Definidos en `TCdm.h`, generados a partir de `xfscdm.h` versión 3.40. Sin dependencia de cabeceras XFS o Windows.

| Enum class | Descripción |
|---|---|
| `CdmInfoCmd` | Comandos getInfo CDM (301–313). |
| `CdmExecCmd` | Comandos execute CDM (301–329). |
| `CdmEvents` | Eventos CDM execute/service/user (301–324). |
| `CdmError` | Códigos de error específicos CDM (-300 a -343). |
| `CdmDeviceStatus` | Estado del hardware del dispensador. |
| `CdmSafeDoor` | Estado de la puerta del safe. |
| `CdmDispenser` | Estado del módulo dispensador. |
| `CdmIntermediateStacker` | Estado del stacker intermedio. |
| `CdmOutPosition` | Posición de salida lógica. |
| `CdmCashUnitType` | Tipo de unidad de efectivo. |
| `CdmCashUnitStatus` | Estado de la unidad de efectivo. |
| `CdmRetractArea` | Área de retracción de billetes. |
| `CdmRetractAction` | Acción de retracción. |
| `CdmMoveItems` | Movimiento de ítems soportado. |
| `CdmExchangeType` | Tipo de ciclo de reposición. |
| `CdmGuidLightValue` | Valor de luz de guía. |
| `CdmItemInfoType` | Tipo de información de ítem. |
| `CdmDevicePosition` | Posición física del dispositivo. |
| `CdmNoteErrorReason` | Razón de error de billete. |
| `CdmAntiFraudModule` | Estado del módulo antifraude. |

Cada enum dispone de funciones helper: `getNumber(e)`, `getCdm<Enum>(v)` y en algunos casos `toString(e)`.

## Cabecera de inclusión única

`XfsObjectsCdm.h` incluye todas las cabeceras de objeto. `TCdm.h` incluye todas las cabeceras de enums CDM.

```cpp
#include "XfsObjectsCdm.h"   // todos los objetos obj::cdm::*
#include "TCdm.h"            // todos los enums TCdm::*
```

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | `IXfsSerializable`, `Flags<T>`, tipos de soporte del core XFS |
| `XfsProtos` | `../../externs/XfsProtos/inc` | Mensajes Protobuf generados (`proto::cdm::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| vcpkg (Protobuf) | `C:\Projects\vcpkg\installed\<plat>-windows-static\include` | Runtime de serialización Protobuf |
| STL | — | `<memory>`, `<vector>`, `<string>`, `<array>`, `<map>`, `<cstdint>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0


## Uso en proyectos consumidores

1. Añadir `../../externs/XfsObjectsCdm/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsObjectsCdm/lib/$(Platform)/$(Configuration)/XfsObjectsCdm.lib`.

```cpp
#include "XfsObjectsCdm.h"   // o incluir sólo las cabeceras necesarias

// Construir un objeto de dispensación
obj::cdm::CdmDispense dispense;
dispense.usTellerID   = 0;
dispense.usMixNumber  = 1;
dispense.fwPosition   = TCdm::CdmOutPosition::Default;
dispense.bPresent     = true;
dispense.lpDenomination.cCurrencyID = "EUR";
dispense.lpDenomination.ulAmount    = 20000;

// Serializar para transporte
std::vector<uint8_t> buffer;
dispense.serialize(buffer);
```

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport  ──┐
XfsProtos       ──┤──► XfsObjectsCdm ──┐
Log             ──┘                    │
                                       ▼
XfsCoreExe      ─────────────────► XfsBaseCdm ──► SpCdm.exe
XfsObjects      ─────────────────►
```
