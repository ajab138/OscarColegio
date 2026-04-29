# XfsFactoryCim

Biblioteca estática C++ que actúa como **fábrica de objetos SPI para módulos de recepción de efectivo (CIM)** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0


## Propósito

`XfsFactoryCim` implementa la interfaz `IXfsFactory` para la clase de servicio CIM, encargándose de:

- **Instanciar los objetos SPI** (`IXfsSpiObject`) adecuados para cada comando `execute`, categoría `getInfo` y evento CIM, en función de la versión XFS negociada.
- **Despacho por versión**: enruta la creación a `CXfsFactoryCim320` (XFS v3.20) o `CXfsFactoryCim330` (XFS v3.30).
- **Publicar la clase de servicio** (`SERVICE_CLASS_CIM`) y el rango de versiones soportado (`SRVC_VERSION_RANGE = 0x14031E03`).

Es la fábrica que el núcleo XFS usa para construir los contenedores de datos de entrada/salida de cada operación CIM antes de invocar al SPI del dispositivo.

## Arquitectura

```
IXfsFactory
      │
      ▼
CXfsFactoryCim
      │
      ├── CXfsFactoryCim320   ←  objetos SPI versión XFS 3.20
      └── CXfsFactoryCim330   ←  objetos SPI versión XFS 3.30
```

`CXfsFactoryCim` implementa `IXfsFactory` de forma `final`. En cada método fábrica, resuelve la versión XFS a partir de `wVersion` (vía `TXfs::getXfsVersion`) y delega la construcción del objeto SPI concreto a la clase de versión correspondiente. Las clases `CXfsFactoryCim320` y `CXfsFactoryCim330` solo ofrecen métodos estáticos de creación; no tienen estado propio.

## Interfaces y clases publicadas

### `XfsFactoryCim.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryCim` | Implementación `final` de `IXfsFactory` para la clase CIM. Expone `versionRange()` y `getServiceClass()` además de los cinco métodos fábrica. |

#### Métodos de `CXfsFactoryCim`

| Método | Descripción |
|---|---|
| `newGetInfoInput(dwCategory, wVersion)` | Crea el objeto de entrada para una categoría `getInfo`. |
| `newGetInfoOutput(dwCategory, wVersion)` | Crea el objeto de salida para una categoría `getInfo`. |
| `newExecuteInput(dwCommand, wVersion)` | Crea el objeto de entrada para un comando `execute`. |
| `newExecuteOutput(dwCommand, wVersion)` | Crea el objeto de salida para un comando `execute`. |
| `newEventOutput(dwEventID, wVersion)` | Crea el objeto de salida para un evento CIM. |
| `versionRange()` | Devuelve `0x14031E03` (versiones XFS 3.20–3.30). |
| `getServiceClass()` | Devuelve `TCim::SERVICE_CLASS_CIM`. |

### `XfsFactoryCim320.h` / `XfsFactoryCim330.h`

Clases auxiliares con métodos estáticos para cada versión:

```cpp
static std::unique_ptr<IXfsSpiObject> newExecuteInput(TCim::CimExeCmd);
static std::unique_ptr<IXfsSpiObject> newExecuteOutput(TCim::CimExeCmd);
static std::unique_ptr<IXfsSpiObject> newGetInfoInput(TCim::CimInfoCmd);
static std::unique_ptr<IXfsSpiObject> newGetInfoOutput(TCim::CimInfoCmd);
static std::unique_ptr<IXfsSpiObject> newEventOutput(TCim::CimEvents);
```

## Cobertura de comandos y eventos

### Categorías `getInfo`

| Categoría | v3.20 | v3.30 |
|---|:---:|:---:|
| `Status` | ✓ | ✓ |
| `Capabilities` | ✓ | ✓ |
| `CashUnitInfo` | ✓ | ✓ |
| `TellerInfo` | ✓ | ✓ |
| `CurrencyExp` | ✓ | ✓ |
| `BanknoteTypes` | ✓ | ✓ |
| `CashInStatus` | ✓ | ✓ |
| `GetP6Info` | ✓ | ✓ |
| `GetP6Signature` | ✓ | ✓ |
| `GetItemInfo` | ✓ | ✓ |
| `PositionCapabilities` | ✓ | ✓ |
| `ReplenishTarget` | ✓ | ✓ |
| `DeviceLockStatus` | ✓ | ✓ |
| `CashUnitCapabilities` | ✓ | ✓ |
| `DepleteSource` | — | ✓ |
| `GetAllItemsInfo` | — | ✓ |
| `GetBlacklist` | — | ✓ |

### Comandos `execute`

| Comando | v3.20 | v3.30 |
|---|:---:|:---:|
| `CashInStart` | ✓ | ✓ |
| `CashIn` | ✓ | ✓ |
| `CashInEnd` | ✓ | ✓ |
| `CashInRollback` | ✓ | ✓ |
| `Retract` | ✓ | ✓ |
| `OpenShutter` / `CloseShutter` | ✓ | ✓ |
| `SetTellerInfo` | ✓ | ✓ |
| `SetCashUnitInfo` | ✓ | ✓ |
| `StartExchange` / `EndExchange` | ✓ | ✓ |
| `OpenSafeDoor` | ✓ | ✓ |
| `Reset` | ✓ | ✓ |
| `ConfigureCashInUnits` | ✓ | ✓ |
| `ConfigureNoteTypes` | ✓ | ✓ |
| `CreateP6Signature` | ✓ | ✓ |
| `SetGuidanceLight` | ✓ | ✓ |
| `ConfigureNoteReader` | ✓ | ✓ |
| `CompareP6Signature` | ✓ | ✓ |
| `PowerSaveControl` | ✓ | ✓ |
| `Replenish` | ✓ | ✓ |
| `SetCashInLimit` | ✓ | ✓ |
| `CashUnitCount` | ✓ | ✓ |
| `DeviceLockControl` | ✓ | ✓ |
| `SetMode` | ✓ | ✓ |
| `PresentMedia` | ✓ | ✓ |
| `Deplete` | — | ✓ |
| `SetBlacklist` | — | ✓ |

### Eventos

| Evento | Tipo | v3.20 | v3.30 |
|---|---|:---:|:---:|
| `SrveSafeDoorOpen` | Servicio | ✓ | ✓ |
| `SrveSafeDoorClosed` | Servicio | ✓ | ✓ |
| `UsreCashUnitThreshold` | Usuario | ✓ | ✓ |
| `SrveCashUnitInfoChanged` | Servicio | ✓ | ✓ |
| `SrveTellerInfoChanged` | Servicio | ✓ | ✓ |
| `ExeeCashUnitError` | Execute | ✓ | ✓ |
| `SrveItemsTaken` | Servicio | ✓ | ✓ |
| `SrveCountsChanged` | Servicio | ✓ | ✓ |
| `ExeeInputRefuse` | Execute | ✓ | ✓ |
| `SrveItemsPresented` | Servicio | ✓ | ✓ |
| `SrveItemsInserted` | Servicio | ✓ | ✓ |
| `ExeeNoteError` | Execute | ✓ | ✓ |
| `ExeeSubCashIn` | Execute | ✓ | ✓ |
| `SrveMediaDetected` | Servicio | ✓ | ✓ |
| `ExeeInputP6` | Execute | ✓ | ✓ |
| `ExeeInfoAvailable` | Execute | ✓ | ✓ |
| `ExeeInsertItems` | Execute | ✓ | ✓ |
| `SrveDevicePosition` | Servicio | ✓ | ✓ |
| `SrvePowerSaveChange` | Servicio | ✓ | ✓ |
| `ExeeIncompleteReplenish` | Execute | ✓ | ✓ |
| Eventos de sistema (`SystemEventId`) | Sistema | ✓ (fallback) | ✓ (fallback) |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS (`TXfs::*`) |
| `XfsCoreDll` | `../../externs/xfsCoreDll/inc` | `IXfsFactory`, acceso al núcleo XFS |
| `XfsObjects` | `../../externs/xfsObjects/inc` | Tipos XFS genéricos (`objSPI::xfs::*`) |
| `XfsObjectsCim` | `../../externs/xfsObjectsCim/inc` | Tipos CIM (`TCim::*`) |
| `XfsSpiObjects` | `../../externs/XfsSpiObjects/inc` | Interfaz base `IXfsSpiObject` |
| `XfsSpiObjectsCim320` | `../../externs/XfsSpiObjectsCim320/inc` | Tipos SPI CIM versión 3.20 (`objSPI::cim320::*`) |
| `XfsSpiObjectsCim330` | `../../externs/XfsSpiObjectsCim330/inc` | Tipos SPI CIM versión 3.30 (`objSPI::cim330::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |




## Uso en proyectos consumidores

1. Añadir `../../externs/XfsFactoryCim/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsFactoryCim/lib/$(Platform)/$(Configuration)/XfsFactoryCim.lib`.
3. Registrar una instancia de `CXfsFactoryCim` en el núcleo XFS como fábrica para la clase de servicio CIM.

```cpp
#include <XfsFactoryCim.h>

// El núcleo XFS llama a la fábrica para obtener los objetos SPI de cada operación CIM
std::unique_ptr<IXfsFactory> pFactory = std::make_unique<CXfsFactoryCim>();
```

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport      ──┐
XfsCoreDll          ──┤
XfsObjects          ──┤
XfsObjectsCim       ──┤──► XfsFactoryCim ──► SpCim.exe
XfsSpiObjects       ──┤
XfsSpiObjectsCim320 ──┤
XfsSpiObjectsCim330 ──┘
```
