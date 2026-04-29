# XfsSpiObjectsCim330

Biblioteca estática C++ que proporciona los **objetos SPI para Cash-In Module (CIM) versión 3.30** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsSpiObjectsCim330` implementa la capa de serialización/deserialización y de traducción entre estructuras XFS nativas (`WFSCIM*`) y los objetos de dominio CIM del framework. Cada clase de este proyecto:

- **Hereda** de la clase base de dominio correspondiente en `obj::cim::*` (definida en `XfsObjectsCim`) y de `IXfsSpiObject`.
- **Implementa `IXfsSerializable`:** serialización binaria (`serialize`/`deserialize`) y volcado legible (`toString`).
- **Implementa `IXfsSPI`:** inicialización desde un puntero XFS nativo (`initialize`), construcción de la estructura XFS desde el objeto (`createStruct`) y construcción desde objeto base (`createStructFromObject`).

Todas las clases residen en el espacio de nombres `objSPI::cim330`.

## Arquitectura

```
obj::cim::Cim*          (XfsObjectsCim)
       │
       ▼
CimXxx330  ──── IXfsSpiObject
                    ├── IXfsSerializable  →  toString / serialize / deserialize
                    └── IXfsSPI           →  initialize / createStruct / createStructFromObject
```

El fichero cabecera de entrada es `XfsSpiObjectsCim330.h`, que incluye todas las clases del proyecto.

## Clases publicadas

### Estado y capacidades

| Clase | Estructura XFS |
|---|---|
| `CimStatus330` | `WFSCIMSTATUS` |
| `CimCaps330` | `WFSCIMCAPS` |
| `CimCashCapabilities330` | `WFSCIMCASHCAPABILITIES` |
| `CimCashInfo330` | `WFSCIMCASHINFO` |
| `CimCashInStatus330` | `WFSCIMCASHINSTATUS` |
| `CimCurrencyExp330` | `WFSCIMCURRENCYEXP` |
| `CimPositionInfo330` | `WFSCIMPOSITIONINFO` |
| `CimPosCapabilities330` | `WFSCIMPOSCAPABILITIES` |
| `CimPosCaps330` | `WFSCIMPOSCAPS` |
| `CimDeviceLockStatus330` | `WFSCIMDEVICELOCKSTATUS` |
| `CimDevicePosition330` | `WFSCIMDEVICEPOSITION` |

### Unidades y tipos de notas

| Clase | Estructura XFS |
|---|---|
| `CimNoteType330` | `WFSCIMNOTETYPE` |
| `CimNoteTypeList330` | `WFSCIMNOTYPELIST` |
| `CimNoteNumber330` | `WFSCIMNOTENUMBER` |
| `CimNoteNumberList330` | `WFSCIMNOTENUMBERLIST` |
| `CimCashUnitCapabilities330` | `WFSCIMCASHUNITCAPABILITIES` |
| `CimPhCuCapabilities330` | `WFSCIMPHCUCAPABILITIES` |
| `CimPhysicalUnit330` | `WFSCIMPHYSICALUNIT` |

### Información de ítems y firma P6

| Clase | Estructura XFS |
|---|---|
| `CimGetItemInfo330` | `WFSCIMGETITEMINFO` |
| `CimGetAllItemsInfo330` | `WFSCIMGETALLITEMSINFO` |
| `CimItemInfo330` | `WFSCIMITEMINFO` |
| `CimItemInfoAll330` | `WFSCIMITEMINFOALL` |
| `CimItemInfoSummary330` | `WFSCIMITEMINFOSUMMARY` |
| `CimAllItemsInfo330` | `WFSCIMALLITEMSINFO` |
| `CimItemPosition330` | `WFSCIMITEMPOSITION` |
| `CimInPos330` | `WFSCIMINPOS` |
| `CimP6Info330` | `WFSCIMP6INFO` |
| `CimP6Signature330` | `WFSCIMP6SIGNATURE` |
| `CimP6SignaturesIndex330` | `WFSCIMP6SIGNATURESINDEX` |
| `CimP6CompareResult330` | `WFSCIMP6COMPARERESULT` |
| `CimP6CompareSignature330` | `WFSCIMP6COMPARESIGNATURE` |
| `CimGetP6Signature330` | `WFSCIMGETP6SIGNATURE` |

### Comandos CashIn

| Clase | Estructura XFS |
|---|---|
| `CimCashIn330` | `WFSCIMCASHIN` |
| `CimCashInStart330` | `WFSCIMCASHINSTART` |
| `CimCashInLimit330` | `WFSCIMCASHINLIMIT` |
| `CimCashInType330` | `WFSCIMCASHINTYPE` |
| `CimAmountLimit330` | `WFSCIMAMOUNTLIMIT` |
| `CimOutput330` | `WFSCIMOUTPUT` |
| `CimCount330` | `WFSCIMCOUNT` |

### Comandos Replenish (reposición)

| Clase | Estructura XFS |
|---|---|
| `CimRep330` | `WFSCIMREP` |
| `CimRepInfo330` | `WFSCIMREPINFO` |
| `CimRepInfoTarget330` | `WFSCIMREPINFOTARGET` |
| `CimRepInfoRes330` | `WFSCIMREPINFORES` |
| `CimRepTarget330` | `WFSCIMREPTARGET` |
| `CimRepTargetRes330` | `WFSCIMREPTARGETRES` |
| `CimRepRes330` | `WFSCIMREPRES` |
| `CimIncompleteReplenish330` | `WFSCIMINCOMPLETEREPL` |

### Comandos Deplete (descarga entre unidades)

| Clase | Estructura XFS |
|---|---|
| `CimDep330` | `WFSCIMDEP` |
| `CimDepInfo330` | `WFSCIMDEPINFO` |
| `CimDepInfoSource330` | `WFSCIMDEPINFOSOURCE` |
| `CimDepInfoRes330` | `WFSCIMDEPINFORES` |
| `CimDepSource330` | `WFSCIMDEPSOURCE` |
| `CimDepSourceRes330` | `WFSCIMDEPSOURCERES` |
| `CimDepRes330` | `WFSCIMDEPRES` |
| `CimIncompleteDeplete330` | `WFSCIMINCOMPLETEDEPLETE` |

### Gestión del dispositivo

| Clase | Estructura XFS |
|---|---|
| `CimRetract330` | `WFSCIMRETRACT` |
| `CimPresent330` | `WFSCIMINPOS` |
| `CimCashUnitLock330` | `WFSCIMCASHUNITLOCK` |
| `CimUnitLockControl330` | `WFSCIMUNITLOCKCONTROL` |
| `CimDeviceLockControl330` | `WFSCIMDEVICELOCKCONTROL` |
| `CimConfigureNoteReader330` | `WFSCIMCONFIGURENOTEREADER` |
| `CimConfigureNoteReaderOut330` | `WFSCIMCONFIGURENOTEREADEROUT` |
| `CimSetGuidLight330` | `WFSCIMSETGUIDLIGHT` |
| `CimSetMode330` | `WFSCIMSETMODE` |
| `CimPowerSaveControl330` | `WFSCIMSAVEPOWERCONTROL` |
| `CimStartEx330` | `WFSCIMSTARTEX` |
| `CimSynchronizeCommand330` | `WFSCIMSYNCHRONIZECOMMAND` |
| `CimBlacklist330` | `WFSCIMBLACKLIST` |
| `CimBlacklistElement330` | `WFSCIMBLACKLISTELEMENT` |
| `CimCuError330` | `WFSCIMCUERROR` |

### Información de cajero (Teller)

| Clase | Estructura XFS |
|---|---|
| `CimTellerInfo330` | `WFSCIMTELLERINFO` |
| `CimTellerDetails330` | `WFSCIMTELLERDETAILS` |
| `CimTellerTotals330` | `WFSCIMTELLERTOTALS` |
| `CimTellerUpdate330` | `WFSCIMTELLERUPDATE` |

### Eventos

| Clase | Estructura XFS |
|---|---|
| `CimCountsChanged330` | `WFSCIMCOUNTSCHANGED` |
| `CimShutterStatusChanged330` | `WFSCIMSHUTTERSTATUSCHANGED` |
| `CimPowerSaveChange330` | `WFSCIMSAVEPOWERCHANGE` |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | `IXfsSpiObject`, `IXfsSPI`, `IXfsSerializable` |
| `XfsObjectsCim` | `../../externs/XfsObjectsCim/inc` | Clases base de dominio CIM (`obj::cim::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `xfs330` | `../../externs/xfs330/inc` | Estructuras nativas XFS 3.30 (`WFSCIM*`) |
| STL | — | `<sstream>`, `<vector>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0

### Post-build automático

Tras cada compilación, los artefactos se copian automáticamente a:

```
../../externs/XfsSpiObjectsCim330/inc/                         ← cabeceras (.h)
../../externs/XfsSpiObjectsCim330/lib/<Plat>/<Config>/         ← XfsSpiObjectsCim330.lib
```

Esto permite que los proyectos consumidores de la SPI CIM 3.30 consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjectsCim330/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjectsCim330/lib/$(Platform)/$(Configuration)/XfsSpiObjectsCim330.lib`.
3. Incluir `XfsSpiObjectsCim330.h` para acceder a todas las clases SPI CIM 3.30.

```cpp
#include "XfsSpiObjectsCim330.h"

// Inicializar un objeto desde datos XFS nativos
objSPI::cim330::CimStatus330 status;
status.initialize(lpWFSResult->lpBuffer);

// Serializar para transporte
std::vector<uint8_t> buffer;
status.serialize(buffer);

// Construir estructura XFS nativa desde el objeto
status.createStruct(lpWFSResult);
```

## Posición en el árbol de compilación (SPI)

```
xfsCoreSupport    ──┐
xfsCoreDll        ──┤
XfsObjectsCim     ──┤──► XfsSpiObjectsCim330 ──► SpCim.dll (SPI)
xfs330            ──┤
Log               ──┘
```
