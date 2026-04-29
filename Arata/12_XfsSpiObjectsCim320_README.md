# XfsSpiObjectsCim320

Biblioteca estática C++ que proporciona los **objetos SPI para módulos de aceptación de efectivo (CIM) en la versión XFS 3.20** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsSpiObjectsCim320` implementa la capa de serialización/deserialización y conversión de estructuras XFS 3.20 para los tipos CIM. Cada clase del namespace `objSPI::cim320`:

- **Hereda** del tipo de dominio correspondiente de `obj::cim::*` (lógica de negocio CIM independiente del protocolo de transporte).
- **Implementa `IXfsSpiObject`**, que agrupa `IXfsSerializable` e `IXfsSPI`, para cubrir la conversión entre objetos C++ tipados y los buffers de bytes / estructuras C del protocolo XFS 3.20.

Es consumida por el SPI de CIM (`SpCim`) para inicializar objetos desde las estructuras XFS recibidas del framework y para crear las estructuras XFS de respuesta.

## Arquitectura

```
obj::cim::CimXxx          (XfsObjectsCim)
        │
        ▼
objSPI::cim320::CimXxx320  ◄── IXfsSpiObject
                                    │
                                    ├── IXfsSerializable
                                    │     ├── toString(stream)
                                    │     ├── serialize(buffer)
                                    │     └── deserialize(buffer)
                                    └── IXfsSPI
                                          ├── initialize(LPVOID lpData)
                                          ├── createStruct(LPVOID lpWFSResult)
                                          └── createStructFromObject(lpWFSResult, lpObjectBase, lpXfsStruct)
```

Todas las clases son `final`. El punto de entrada para incluir la biblioteca completa es `XfsSpiObjectsCim320.h`.

## Clases publicadas

Todas las clases residen en el namespace `objSPI::cim320`.

### Estado y capacidades

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimStatus320` | `CimStatus` | `WFSCIMSTATUS` |
| `CimCaps320` | `CimCaps` | `WFSCIMCAPS` |
| `CimCashCapabilities320` | `CimCashCapabilities` | `WFSCIMCASHCAPABILITIES` |
| `CimCashInfo320` | `CimCashInfo` | `WFSCIMCASHINFO` |
| `CimCurrencyExp320` | `CimCurrencyExp` | `WFSCIMCURRENCYEXP` |

### Flujo de aceptación de efectivo

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimCashIn320` | `CimCashIn` | `WFSCIMCASHIN` |
| `CimCashInLimit320` | `CimCashInLimit` | `WFSCIMCASHINLIMIT` |
| `CimCashInStart320` | `CimCashInStart` | `WFSCIMCASHINSTART` |
| `CimCashInStatus320` | `CimCashInStatus` | `WFSCIMCASHINSTATUSEX` |
| `CimCashInType320` | `CimCashInType` | `WFSCIMCASHINTYPE` |
| `CimStartEx320` | `CimStartEx` | `WFSCIMSTARTEX` |
| `CimOutput320` | `CimOutput` | `WFSCIMOUTPUT` |
| `CimPresent320` | `CimPresent` | `WFSCIMINPOS` |
| `CimRetract320` | `CimRetract` | `WFSCIMRETRACT` |
| `CimAmountLimit320` | `CimAmountLimit` | `WFSCIMAMOUNTLIMIT` |

### Unidades físicas de efectivo

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimCount320` | `CimCount` | `WFSCIMCOUNT` |
| `CimCashUnitCapabilities320` | `CimCashUnitCapabilities` | `WFSCIMCASHUNITCAPABILITIES` |
| `CimCashUnitLock320` | `CimCashUnitLock` | `WFSCIMCASHUNITLOCK` |
| `CimPhCuCapabilities320` | `CimPhCuCapabilities` | `WFSCIMPHCUCAPABILITIES` |
| `CimPhysicalUnit320` | `CimPhysicalUnit` | `WFSCIMPHYSICALUNIT` |
| `CimUnitLockControl320` | `CimUnitLockControl` | `WFSCIMUNITLOCKCONTROL` |
| `CimDeviceLockControl320` | `CimDeviceLockControl` | `WFSCIMDEVICELOCKCONTROL` |
| `CimDeviceLockStatus320` | `CimDeviceLockStatus` | `WFSCIMDEVICELOCKSTATUS` |

### Billetes y tipos de nota

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimNoteNumber320` | `CimNoteNumber` | `WFSCIMNOTENUM` |
| `CimNoteNumberList320` | `CimNoteNumberList` | `WFSCIMNOTENUMLIST` |
| `CimNoteType320` | `CimNoteType` | `WFSCIMINOTETYPELIST` |
| `CimNoteTypeList320` | `CimNoteTypeList` | `WFSCIMNOTETYPELIST` |
| `CimConfigureNoteReader320` | `CimConfigureNoteReader` | `WFSCIMCONFIGURENOTEREADER` |
| `CimConfigureNoteReaderOut320` | `CimConfigureNoteReaderOut` | `WFSCIMCONFIGURENOTEREADEROUT` |

### Posiciones e información de ítems

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimInPos320` | `CimInPos` | `WFSCIMINPOS` |
| `CimItemInfo320` | `CimItemInfo` | `WFSCIMITEMINFO` |
| `CimItemInfoSummary320` | `CimItemInfoSummary` | `WFSCIMITEMINFOSUMMARY` |
| `CimItemPosition320` | `CimItemPosition` | `WFSCIMITEMPOSITION` |
| `CimGetItemInfo320` | `CimGetItemInfo` | `WFSCIMGETITEMINFO` |
| `CimDevicePosition320` | `CimDevicePosition` | `WFSCIMDEVICEPOSITION` |
| `CimPositionInfo320` | `CimPositionInfo` | `WFSCIMPOSITIONINFO` |
| `CimPosCapabilities320` | `CimPosCapabilities` | `WFSCIMPOSCAPABILITIES` |
| `CimPosCaps320` | `CimPosCaps` | `WFSCIMPOSCAPS` |

### Firma P6

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimGetP6Signature320` | `CimGetP6Signature` | `WFSCIMGETP6SIGNATURE` |
| `CimP6CompareResult320` | `CimP6CompareResult` | `WFSCIMP6COMPARERESULT` |
| `CimP6CompareSignature320` | `CimP6CompareSignature` | `WFSCIMP6COMPARESIGNATURE` |
| `CimP6Info320` | `CimP6Info` | `WFSCIMP6INFO` |
| `CimP6Signature320` | `CimP6Signature` | `WFSCIMP6SIGNATURE` |
| `CimP6SignaturesIndex320` | `CimP6SignaturesIndex` | `WFSCIMP6SIGNATURESINDEX` |

### Reposición (Replenishment)

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimRep320` | `CimRep` | `WFSCIMREP` |
| `CimRepInfo320` | `CimRepInfo` | `WFSCIMREPINFO` |
| `CimRepInfoRes320` | `CimRepInfoRes` | `WFSCIMREPINFORES` |
| `CimRepInfoTarget320` | `CimRepInfoTarget` | `WFSCIMREPINFOTARGET` |
| `CimRepRes320` | `CimRepRes` | `WFSCIMREPRES` |
| `CimRepTarget320` | `CimRepTarget` | `WFSCIMREPTARGET` |
| `CimRepTargetRes320` | `CimRepTargetRes` | `WFSCIMREPTARGETRES` |
| `CimIncompleteReplenish320` | `CimIncompleteReplenish` | `WFSCIMINCOMPLETEREPLENISH` |

### Cajeros (Teller)

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimTellerDetails320` | `CimTellerDetails` | `WFSCIMTELLERDETAILS` |
| `CimTellerInfo320` | `CimTellerInfo` | `WFSCIMTELLERINFO` |
| `CimTellerTotals320` | `CimTellerTotals` | `WFSCIMTELLERTOTALS` |
| `CimTellerUpdate320` | `CimTellerUpdate` | `WFSCIMTELLERUPDATE` |

### Control y eventos

| Clase | Base `obj::cim::*` | Estructura XFS 3.20 |
|---|---|---|
| `CimCountsChanged320` | `CimCountsChanged` | `WFSCIMCOUNTSCHANGED` |
| `CimCuError320` | `CimCuError` | `WFSCIMCUERROR` |
| `CimPowerSaveChange320` | `CimPowerSaveChange` | `WFSCIMPOWERSAVECHANGE` |
| `CimPowerSaveControl320` | `CimPowerSaveControl` | `WFSCIMPOWERSAVECONTROL` |
| `CimSetGuidLight320` | `CimSetGuidLight` | `WFSCIMSETGUIDLIGHT` |
| `CimSetMode320` | `CimSetMode` | `WFSCIMSETMODE` |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IXfsSpiObject`, `IXfsSerializable`, `IXfsSPI` |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | Tipos y constantes del protocolo XFS DLL |
| `XfsObjectsCim` | `../../externs/XfsObjectsCim/inc` | Tipos de dominio CIM (`obj::cim::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `xfs320` | `../../externs/xfs320/inc` | Estructuras C del estándar CEN/XFS versión 3.20 |
| STL | — | `<sstream>`, `<vector>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Excepción SEH:** habilitada (`/EHa`)

### Post-build automático

Tras cada compilación, los artefactos se copian automáticamente a:

```
../../externs/XfsSpiObjectsCim320/inc/                     ← cabeceras (.h)
../../externs/XfsSpiObjectsCim320/lib/<Plat>/<Config>/     ← XfsSpiObjectsCim320.lib
```

Esto permite que el SPI de CIM y otros proyectos consumidores integren la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjectsCim320/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjectsCim320/lib/$(Platform)/$(Configuration)/XfsSpiObjectsCim320.lib`.
3. Incluir `XfsSpiObjectsCim320.h` para acceder a todas las clases del namespace `objSPI::cim320`.

```cpp
#include "XfsSpiObjectsCim320.h"

// Inicializar desde estructura XFS recibida del framework
objSPI::cim320::CimStatus320 status;
status.initialize(lpWFSResult->u.lpBuffer);

// Serializar para transporte
std::vector<uint8_t> buffer;
status.serialize(buffer);

// Crear estructura XFS de respuesta
status.createStruct(lpWFSResult);
```

## Posición en el árbol de compilación (SPI)

```
xfsCoreSupport     ──┐
xfsCoreDll         ──┤
XfsObjectsCim      ──┤──► XfsSpiObjectsCim320 ──► SpCim.dll
xfs320             ──┤
Log                ──┘
```
