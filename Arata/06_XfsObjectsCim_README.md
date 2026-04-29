# XfsObjectsCim

Biblioteca estática C++ que proporciona el **modelo de objetos tipado para aceptadores de efectivo (CIM)** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsObjectsCim` define las clases C++ que encapsulan todas las estructuras de datos del estándar XFS CIM (Cash-In Module, clase de servicio 13, versión 3.40):

- **Wrappers tipados** de todas las estructuras `WFS_CIM_*`, expuestos en el espacio de nombres `obj::cim::`.
- **Enumeraciones fuertemente tipadas** (`enum class`) que reemplazan los `#define` del header XFS, agrupadas en el espacio de nombres `TCim::`.
- **Serialización binaria** mediante `IXfsSerializable` (`serialize` / `deserialize`) para transporte entre procesos.
- **Integración con Protocol Buffers**: cada objeto expone `toProto()`, `fromProto()`, `setProto()` y `getProto()` para interoperar con la capa gRPC del framework.

Es la biblioteca de tipos que consumen `XfsObjectsCim`-dependientes como `XfsBaseCim` y las implementaciones concretas de aceptadores.

## Arquitectura

```
IXfsSerializable          (XfsCoreSupport)
       │
       ▼
obj::cim::Cim<Xxx>   ◄──── proto::cim::<XXX>   (XfsProtos / protobuf)
       │
       └── TCim::<EnumClass>    (constantes y enumeraciones del estándar XFS CIM)
```

Cada clase del espacio de nombres `obj::cim` implementa `IXfsSerializable` de forma independiente. Los objetos compuestos agregan otros objetos `obj::cim` mediante composición o `std::vector<>`.

## Cabeceras publicadas

### `XfsObjectsCim.h`

Cabecera agregadora: incluye todos los tipos de la biblioteca. Es el único include necesario para los consumidores.

### `TCim.h`

Define todos los tipos enumerados del estándar XFS CIM como `enum class` C++11.

| Tipo | Descripción |
|---|---|
| `TCim::CimInfoCmd` | Comandos getInfo CIM (1301–1320) |
| `TCim::CimExeCmd` | Comandos execute CIM (1301–1331) |
| `TCim::CimEvents` | Eventos CIM (service, user y execute) |
| `TCim::CimDeviceStatus` | Estado del dispositivo |
| `TCim::CimAcceptor` | Estado del aceptador |
| `TCim::CimIntermediateStacker` | Estado del stacker intermedio |
| `TCim::CimCashUnitType` | Tipo de unidad de efectivo |
| `TCim::CimCashUnitStatus` | Estado de unidad de efectivo |
| `TCim::CimItemType` | Tipo de elemento (billete, moneda, etc.) |
| `TCim::CimPosition` | Posiciones soportadas |
| `TCim::CimRetractArea` | Áreas de retracción |
| `TCim::CimRetractAction` | Acciones de retracción al transport/stacker |
| `TCim::CimExchangeType` | Tipos de intercambio soportados |
| `TCim::CimCountAction` | Acciones de conteo |
| `TCim::CimCashInLimit` | Tipos de límite de cash-in |
| `TCim::CimMixedModeCaps` / `CimMixedModeStatus` | Modo mixto aceptador/dispensador |
| Otros | Enumeraciones para P6, clasificación, bloqueo, ahorro energía, etc. |

Todas las enumeraciones incluyen funciones auxiliares `toString(e)`, `getNumber(e)` y `getCim<X>(uint32_t)`.

## Catálogo de tipos `obj::cim`

### Estado y capacidades

| Clase | Descripción |
|---|---|
| `CimStatus` | Estado completo del dispositivo (device, safe door, acceptor, stacker, posiciones, guid lights, etc.) |
| `CimCaps` | Capacidades del dispositivo (tipo, posiciones, áreas retracción, P6, clasificación, etc.) |
| `CimCashInStatus` | Estado de la operación cash-in en curso |
| `CimCashCapabilities` | Capacidades de una posición de efectivo |
| `CimPosCapabilities` | Capacidades de una posición de entrada/salida |
| `CimPosCaps` | Estructura auxiliar de capacidades de posición |

### Unidades de efectivo

| Clase | Descripción |
|---|---|
| `CimCashIn` | Unidad lógica de efectivo (tipo, divisa, conteos, estado, unidades físicas asociadas) |
| `CimCashInfo` | Colección de unidades lógicas (`vector<CimCashIn>`) |
| `CimPhysicalUnit` | Unidad física de efectivo |
| `CimCashUnitCapabilities` | Capacidades de una unidad lógica |
| `CimCashUnitCountStatus` | Estado de conteo de unidad lógica |
| `CimPhCuCapabilities` | Capacidades de una unidad física |
| `CimPhCuCountStatus` | Estado de conteo de unidad física |
| `CimCashUnitLock` | Control de bloqueo de unidad lógica |
| `CimUnitLockControl` | Control de bloqueo de unidades (comando execute) |

### Operaciones cash-in

| Clase | Descripción |
|---|---|
| `CimCashInStart` | Parámetros para iniciar una operación cash-in |
| `CimCashInType` | Tipo de operación cash-in |
| `CimCashInLimit` | Límite de importe/conteo para la operación cash-in |
| `CimAmountLimit` | Límite de importe por divisa |
| `CimCount` | Conteo de billetes por tipo |
| `CimStartEx` | Parámetros para inicio de intercambio |
| `CimMoveItems` | Movimiento de elementos entre posiciones |

### Posicionamiento y presentación

| Clase | Descripción |
|---|---|
| `CimInPos` | Información de posición de entrada |
| `CimOutput` | Posición de salida (transport/stacker/refuse) |
| `CimItemPosition` | Posición de un elemento detectado |
| `CimDevicePosition` | Posición física del dispositivo |
| `CimPositionInfo` | Información de posición de efectivo |
| `CimPresent` | Parámetros de presentación de medios |
| `CimPresentStatus` | Estado de la presentación activa |

### Retracción

| Clase | Descripción |
|---|---|
| `CimRetract` | Parámetros de retracción de billetes |
| `CimIncompleteRetract` | Resultado de retracción incompleta |

### Reposición (Replenish)

| Clase | Descripción |
|---|---|
| `CimRep` | Parámetros de la operación de reposición |
| `CimRepInfo` | Solicitud de información de objetivos de reposición |
| `CimRepInfoTarget` | Objetivo de reposición |
| `CimRepInfoRes` | Resultado de consulta de objetivos |
| `CimRepTarget` | Objetivo durante ejecución |
| `CimRepTargetRes` | Resultado de objetivo de reposición |
| `CimRepRes` | Resultado global de la reposición |
| `CimIncompleteReplenish` | Resultado de reposición incompleta |

### Vaciado (Deplete)

| Clase | Descripción |
|---|---|
| `CimDep` | Parámetros de la operación de vaciado |
| `CimDepInfo` | Solicitud de información de fuentes de vaciado |
| `CimDepInfoSource` | Fuente de vaciado |
| `CimDepInfoRes` | Resultado de consulta de fuentes |
| `CimDepSource` | Fuente durante ejecución |
| `CimDepSourceRes` | Resultado de fuente de vaciado |
| `CimDepRes` | Resultado global del vaciado |
| `CimIncompleteDeplete` | Resultado de vaciado incompleto |

### Tipos de billetes y clasificación

| Clase | Descripción |
|---|---|
| `CimNoteType` | Definición de un tipo de billete |
| `CimNoteTypeList` | Lista de tipos de billete configurados |
| `CimNoteNumber` | Par (tipo de billete, cantidad) |
| `CimNoteNumberList` | Lista de `CimNoteNumber` |
| `CimClassificationElement` | Elemento de la lista de clasificación |
| `CimClassificationList` | Lista de clasificación de billetes |
| `CimBlacklistElement` | Elemento de la lista negra |
| `CimBlacklist` | Lista negra de billetes |
| `CimCurrencyExp` | Exponentes de divisas |

### Firma P6

| Clase | Descripción |
|---|---|
| `CimP6Info` | Información del banco de firmas P6 |
| `CimP6Signature` | Firma P6 de un billete |
| `CimP6SignaturesIndex` | Índice del banco de firmas |
| `CimGetP6Signature` | Parámetros para solicitar una firma P6 |
| `CimP6CompareSignature` | Parámetros para comparar firma P6 |
| `CimP6CompareResult` | Resultado de comparación de firma P6 |

### Información de elementos

| Clase | Descripción |
|---|---|
| `CimGetItemInfo` | Parámetros para solicitar info de un elemento |
| `CimGetAllItemsInfo` | Parámetros para solicitar info de todos los elementos |
| `CimItemInfo` | Información detallada de un elemento |
| `CimItemInfoAll` | Colección de información de todos los elementos |
| `CimItemInfoSummary` | Resumen de información de elemento (evento) |
| `CimAllItemsInfo` | Resultado completo de info de todos los elementos |

### Cajeros

| Clase | Descripción |
|---|---|
| `CimTellerInfo` | Parámetros de consulta de cajero |
| `CimTellerDetails` | Detalles de un cajero |
| `CimTellerTotals` | Totales acumulados de un cajero |
| `CimTellerUpdate` | Actualización de información de cajero |

### Control del dispositivo y configuración

| Clase | Descripción |
|---|---|
| `CimDeviceLockControl` | Parámetros de control de bloqueo del dispositivo |
| `CimDeviceLockStatus` | Estado de bloqueo del dispositivo |
| `CimSetGuidLight` | Control de luces de guía |
| `CimSetMode` | Configuración del modo de operación |
| `CimPowerSaveControl` | Control de modo de ahorro de energía |
| `CimSynchronizeCommand` | Parámetros de sincronización de comandos |
| `CimConfigureNoteReader` | Parámetros de configuración del lector de billetes |
| `CimConfigureNoteReaderOut` | Resultado de configuración del lector |

### Eventos

| Clase | Descripción |
|---|---|
| `CimCountsChanged` | Notificación de cambio de conteos de unidades |
| `CimShutterStatusChanged` | Notificación de cambio de estado del obturador |
| `CimPowerSaveChange` | Notificación de cambio de modo de ahorro de energía |
| `CimCuError` | Error en una unidad de efectivo |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | `IXfsSerializable`, `Flags<T>`, tipos de soporte |
| `XfsProtos` | `../../externs/XfsProtos/inc` | Definiciones protobuf generadas para CIM |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| vcpkg | `C:\Projects\vcpkg\installed\<plat>-windows-static\include` | Abseil / protobuf (dependencias de XfsProtos) |
| STL | — | `<memory>`, `<vector>`, `<map>`, `<array>`, `<cstdint>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0


## Uso en proyectos consumidores

1. Añadir `../../externs/XfsObjectsCim/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsObjectsCim/lib/$(Platform)/$(Configuration)/XfsObjectsCim.lib`.
3. Incluir `XfsObjectsCim.h` (o las cabeceras individuales necesarias).

```cpp
#include "XfsObjectsCim.h"   // todos los tipos CIM

// Usar tipos directamente:
obj::cim::CimStatus  status;
obj::cim::CimCashInfo cashInfo;

// Serializar para transporte entre procesos:
std::vector<uint8_t> buffer;
status.serialize(buffer);

// Deserializar en el receptor:
obj::cim::CimStatus statusRecv;
statusRecv.deserialize(buffer);

// Acceder a enumeraciones tipadas:
TCim::CimInfoCmd cmd = TCim::CimInfoCmd::Status;
```

## Posición en el árbol de compilación

```
XfsCoreSupport  ──┐
XfsProtos       ──┤──► XfsObjectsCim ──► XfsBaseCim ──► SpCim.exe
Log             ──┘
```
