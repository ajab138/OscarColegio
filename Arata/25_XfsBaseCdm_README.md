# XfsBaseCdm

Biblioteca estática C++ que actúa como **capa base específica para dispensadores de efectivo (CDM)** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsBaseCdm` implementa el patrón **Template Method** para gestionar la lógica común del servicio CDM:

- **Despacho genérico** de comandos `getInfo` y `execute` de la clase CDM, con soporte de caché thread-safe para las categorías cacheables.
- **Delegación** de la ejecución concreta de cada comando en la interfaz `IXfsExecutorCdm`, que deben implementar las clases de dispositivo concretas.
- **Publicación de eventos** CDM (execute, service y user) hacia el framework XFS.

Es la base de la que heredan las implementaciones concretas de dispensadores de efectivo (`SpCdm`, etc.).

## Arquitectura

```
IXfsClassDevice          (XfsCoreExe)
       │
       ▼
CXfsBaseCdm  ◄──── IXfsExecutorCdm  (implementada por cada clase concreta de CDM)
       │
       ├── CXfsCacheCdm        →  IXfsCacheCdm
       ├── CXfsEventsExeeCdm   →  IXfsEventsExeeCdm
       └── CXfsEventsSrveUsreCdm → IXfsEventsSrveUsreCdm
```

`CXfsBaseCdm` implementa `IXfsClassDevice` de forma `final` para `processGetInfo` y `processExecute`, gestionando internamente la caché y el despacho de eventos. Las clases concretas solo necesitan implementar los métodos de `IXfsExecutorCdm` correspondientes a los comandos que soportan.

## Interfaces publicadas

### `IXfsBaseCdm.h`

| Interfaz | Descripción |
|---|---|
| `IXfsCacheCdm` | Contrato para acceder y actualizar la caché de información CDM. |
| `IXfsEventsExeeCdm` | Publica eventos de ejecución CDM hacia el framework. |
| `IXfsEventsSrveUsreCdm` | Publica eventos de servicio y usuario CDM hacia el framework. |
| `IXfsExecutorCdm` | Contrato que toda clase concreta CDM debe implementar. |

#### `IXfsCacheCdm`

Almacena las categorías de `getInfo` que pueden cachearse. Acceso protegido mediante `mutex`.

| Método | Tipo |
|---|---|
| `setCacheStatus` / `getCacheStatus` | `obj::cdm::CdmStatus` |
| `setCacheCapabilities` / `getCacheCapabilities` | `obj::cdm::CdmCaps` |
| `setCacheCuInfo` / `getCacheCuInfo` | `obj::cdm::CdmCuInfo` |
| `setCacheCurrencyExp` / `getCacheCurrencyExp` | `obj::cdm::CdmCurrencyExp` |
| `setCacheMixTypes` / `getCacheMixTypes` | `obj::cdm::CdmMixType` |
| `setCacheMixTable` / `getCacheMixTable` | `obj::cdm::CdmMixTable` |
| `setCachePresentStatus` / `getCachePresentStatus` | `obj::cdm::CdmPresentStatus` |
| `setCacheBlacklist` / `getCacheBlacklist` | `obj::cdm::CdmBlacklist` |
| `setCacheClassificationList` / `getCacheClassificationList` | `obj::cdm::CdmClassificationList` |

#### `IXfsEventsExeeCdm`

```cpp
virtual void fireExeeDelayedDispense(obj::xfs::ObjNumber& event) const = 0;
virtual void fireExeeStartDispense(obj::xfs::ObjNumber& event) const = 0;
virtual void fireExeeCashUnitError(obj::cdm::CdmCuError& event) const = 0;
virtual void fireExeePartialDispense(obj::xfs::ObjNumber& event) const = 0;
virtual void fireExeeSubDispenseOk(obj::cdm::CdmDenomination& event) const = 0;
virtual void fireExeeIncompleteDispense(obj::cdm::CdmDenomination& event) const = 0;
virtual void fireExeeNoteError(TCdm::CdmNoteErrorReason& event) const = 0;
virtual void fireExeeInputP6() const = 0;
virtual void fireExeeInfoAvailable(obj::cdm::CdmItemInfoSummary& event) const = 0;
virtual void fireIncompleteRetract(const obj::cdm::CdmIncompleteRetract& event) const = 0;
```

#### `IXfsEventsSrveUsreCdm`

```cpp
virtual void fireSrveSafeDoorOpen() = 0;
virtual void fireSrveSafeDoorClosed() = 0;
virtual void fireUsreCashUnitThreshold(obj::cdm::CdmCashUnit& event) const = 0;
virtual void fireSrveCashUnitInfoChanged(obj::cdm::CdmCashUnit& event) const = 0;
virtual void fireSrveTellerInfoChanged(obj::xfs::ObjNumber& event) const = 0;
virtual void fireSrveItemsTaken(TCdm::CdmOutPosition&& event) const = 0;
virtual void fireSrveCountsChanged(obj::cdm::CdmCountsChanged& event) const = 0;
virtual void fireSrveItemsPresented() const = 0;
virtual void fireSrveMediaDetected(obj::cdm::CdmItemPosition& event) const = 0;
virtual void fireSrveDevicePosition(obj::cdm::CdmItemPosition& event) const = 0;
virtual void fireSrvePowerSaveChange(obj::cdm::CdmPowerSaveChange& event) const = 0;
virtual void fireSrveShutterStatusChanged(obj::cdm::CdmShutterStatusChanged& event) const = 0;
virtual void fireSrveItemsInserted(TCdm::CdmOutPosition& event) const = 0;
```

#### `IXfsExecutorCdm` — comandos execute

| Método | Descripción |
|---|---|
| `executeDenominate` | Calcular denominación para un importe dado. |
| `executeDispense` | Dispensar efectivo. |
| `executeCount` | Contar billetes en unidades físicas. |
| `executePresent` | Presentar billetes en la posición de salida. |
| `executeReject` | Rechazar billetes presentados. |
| `executeRetract` | Retraer billetes al interior del dispensador. |
| `executeOpenShutter` / `executeCloseShutter` | Controlar el obturador de salida. |
| `executeSetTellerInfo` | Configurar información de cajero. |
| `executeSetCashUnitInfo` | Actualizar información de unidades de efectivo. |
| `executeStartExchange` / `executeEndExchange` | Gestionar ciclo de reposición de efectivo. |
| `executeOpenSafeDoor` | Abrir la puerta del safe. |
| `executeCalibrateCashUnit` | Calibrar unidades de efectivo. |
| `executeSetMixTable` | Configurar tabla de mezcla de billetes. |
| `executeReset` | Resetear el dispositivo. |
| `executeTestCashUnits` | Testear unidades de efectivo. |
| `executeSetGuidanceLight` | Controlar luz de guía. |
| `executePowerSaveControl` | Gestionar modo de ahorro de energía. |
| `executePrepareDispense` | Preparar el dispensador para una dispensación. |
| `executeSetBlacklist` | Configurar lista negra de billetes. |
| `executeSynchronizeCommand` | Sincronizar comandos. |
| `executeClassificationList` | Configurar lista de clasificación de billetes. |

#### `IXfsExecutorCdm` — comandos getInfo no cacheados

```cpp
virtual std::variant<TXfs::XFSError, TCdm::CdmError> infoTellerInfo(
    const obj::cdm::CdmTellerInfo& in, obj::cdm::CdmTellerDetails& out) = 0;
virtual TXfs::XFSError infoGetItemInfo(
    const obj::cdm::CdmGetItemInfo& in, obj::cdm::CdmItemInfo& out) = 0;
virtual TXfs::XFSError infoGetAllItemsInfo(
    const obj::cdm::CdmGetAllItemsInfo& in, obj::cdm::CdmAllItemsInfo& out) = 0;
```

### `XfsBaseCdm.h`

| Clase | Descripción |
|---|---|
| `CXfsBaseCdm` | Implementación base concreta de `IXfsClassDevice` para CDM. Instancia internamente `CXfsCacheCdm`, `CXfsEventsExeeCdm` y `CXfsEventsSrveUsreCdm`. Todos los comandos de `IXfsExecutorCdm` tienen implementación base que devuelve `UnsuppCommand` / `UnsuppCategory`. |

### `XfsEventsCdm.h`

| Clase | Descripción |
|---|---|
| `CXfsEventsExeeCdm` | Implementación de `IXfsEventsExeeCdm`. Serializa los objetos de evento y los eleva al framework mediante `notifyExecuteEvent`. |
| `CXfsEventsSrveUsreCdm` | Implementación de `IXfsEventsSrveUsreCdm`. Eleva eventos de servicio (`notifyServiceEvent`) y usuario (`notifyUserEvent`). |

### `XfsCacheCdm.h`

| Clase | Descripción |
|---|---|
| `CXfsCacheCdm` | Implementación `final` de `IXfsCacheCdm`. Thread-safe mediante `std::mutex` en cada acceso. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreExe` | `../../externs/XfsCoreExe/inc` | `IXfsClassDevice`, `IXfsCommonDevice`, gestión de eventos al framework |
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `XfsObjectsCdm` | `../../externs/XfsObjectsCdm/inc` | Tipos CDM (`obj::cdm::*`, `TCdm::*`) |
| `XfsObjects` | `../../externs/XfsObjects/inc` | Tipos XFS genéricos (`obj::xfs::*`, `TXfs::XFSError`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| STL | — | `<mutex>`, `<vector>`, `<variant>` |

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
../../externs/XfsBaseCdm/inc/                          ← cabeceras (.h)
../../externs/XfsBaseCdm/lib/<Plat>/<Config>/          ← XfsBaseCdm.lib
```

Esto permite que los proyectos ejecutables de CDM consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsBaseCdm/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsBaseCdm/lib/$(Platform)/$(Configuration)/XfsBaseCdm.lib`.
3. Heredar `CXfsBaseCdm` e implementar únicamente los métodos de `IXfsExecutorCdm` que el dispositivo soporta.

```cpp
class MiDispensador : public CXfsBaseCdm
{
protected:
    std::variant<TXfs::XFSError, TCdm::CdmError>
    executeDispense(const obj::cdm::CdmDispense& in,
                    obj::cdm::CdmDenomination& out,
                    const IXfsEventsExeeCdm& ExeeEvents) override;

    std::variant<TXfs::XFSError, TCdm::CdmError>
    executeDenominate(const obj::cdm::CdmDenominate& in,
                      obj::cdm::CdmDenomination& out,
                      const IXfsEventsExeeCdm& ExeeEvents) override;
    // ... otros comandos soportados
};
```

La caché y los eventos son accesibles desde la clase concreta mediante `getCache()` y `getEventsSrveUsre()`.

## Ventajas

- **Caché thread-safe:** `CXfsCacheCdm` protege cada acceso con `std::mutex`, siendo seguro actualizarla desde el hilo de polling o desde suscripciones a eventos de la DevApi.
- **Implementación opcional por comando:** Los comandos no soportados por el hardware devuelven `UnsuppCommand` automáticamente sin necesidad de código en la clase concreta.
- **Separación de tipos de evento:** Los eventos execute, service y user se gestionan por implementaciones independientes, respetando el protocolo XFS.

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport  ──┐
XfsCoreExe      ──┤
XfsObjectsCdm   ──┤──► XfsBaseCdm ──► SpCdm.exe
XfsObjects      ──┤
Log             ──┘
```
