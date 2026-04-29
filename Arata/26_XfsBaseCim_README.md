# XfsBaseCim

Biblioteca estática C++ que actúa como **capa base específica para receptores de efectivo (CIM)** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsBaseCim` implementa el patrón **Template Method** para gestionar la lógica común del servicio CIM:

- **Despacho genérico** de comandos `getInfo` y `execute` de la clase CIM, con soporte de caché thread-safe para las categorías cacheables.
- **Delegación** de la ejecución concreta de cada comando en la interfaz `IXfsExecutorCim`, que deben implementar las clases de dispositivo concretas.
- **Publicación de eventos** CIM (execute, service y user) hacia el framework XFS.
- **Sistema de checks** por comando, extensible desde las clases concretas mediante `checkList` virtuales.

Es la base de la que heredan las implementaciones concretas de receptores de efectivo (`SpCim`, etc.).

## Arquitectura

```
IXfsClassDevice          (XfsCoreExe)
       │
       ▼
CXfsBaseCim  ◄──── IXfsExecutorCim  (implementada por cada clase concreta de CIM)
       │
       ├── CXfsCacheCim            →  IXfsCacheCim
       ├── CXfsEventsExeeCim       →  IXfsEventsExeeCim
       └── CXfsEventsSrveUsreCim   →  IXfsEventsSrveUsreCim
```

`CXfsBaseCim` implementa `IXfsClassDevice` de forma `final` para `processGetInfo` y `processExecute`, gestionando internamente la caché y el despacho de eventos. Las clases concretas solo necesitan implementar los métodos de `IXfsExecutorCim` correspondientes a los comandos que soportan.

## Interfaces publicadas

### `IXfsBaseCim.h`

| Interfaz | Descripción |
|---|---|
| `IXfsCacheCim` | Contrato para acceder y actualizar la caché de información CIM. |
| `IXfsEventsExeeCim` | Publica eventos de ejecución CIM hacia el framework. |
| `IXfsEventsSrveUsreCim` | Publica eventos de servicio y usuario CIM hacia el framework. |
| `IXfsExecutorCim` | Contrato que toda clase concreta CIM debe implementar. |

#### `IXfsCacheCim`

Almacena las categorías de `getInfo` que pueden cachearse. Acceso protegido mediante `mutex`.

| Método | Tipo |
|---|---|
| `setCacheStatus` / `getCacheStatus` | `obj::cim::CimStatus` |
| `setCacheCapabilities` / `getCacheCapabilities` | `obj::cim::CimCaps` |
| `setCacheCashInfo` / `getCacheCashInfo` | `obj::cim::CimCashInfo` |
| `setCacheCurrencyExp` / `getCacheCurrencyExp` | `obj::cim::CimCurrencyExp` |
| `setCacheBanknoteTypes` / `getCacheBanknoteTypes` | `obj::cim::CimNoteTypeList` |
| `setCacheCashInStatus` / `getCacheCashInStatus` | `obj::cim::CimCashInStatus` |
| `setCacheP6Info` / `getCacheP6Info` | `obj::cim::CimP6Info` |
| `setCachePosCapabilities` / `getCachePosCapabilities` | `obj::cim::CimPosCapabilities` |
| `setCacheDeviceLockStatus` / `getCacheDeviceLockStatus` | `obj::cim::CimDeviceLockStatus` |
| `setCacheCashUnitCapabilities` / `getCacheCashUnitCapabilities` | `obj::cim::CimCashUnitCapabilities` |
| `setCacheBlacklist` / `getCacheBlacklist` | `obj::cim::CimBlacklist` |
| `setCacheClassificationList` / `getCacheClassificationList` | `obj::cim::CimClassificationList` |
| `setCacheCashUnitCountStatus` / `getCacheCashUnitCountStatus` | `obj::cim::CimCashUnitCountStatus` |
| `setCachePresentStatus` / `getCachePresentStatus` | `obj::cim::CimPresentStatus` |

#### `IXfsEventsExeeCim`

```cpp
virtual void fireExeeCashUnitError(obj::cim::CimCuError& event) const = 0;
virtual void fireExeeInputRefuse(TCim::CimInputRefuseReason& event) const = 0;
virtual void fireExeeNoteError(TCim::CimNoteErrorReason& event) const = 0;
virtual void fireExeeInfoAvailable(obj::cim::CimItemInfoSummary& event) const = 0;
virtual void fireExeeInsertItems() const = 0;
virtual void firedExeeInputP6(obj::cim::CimP6Info& event) const = 0;
virtual void fireExeeSubCashIn(obj::cim::CimNoteNumberList& event) const = 0;
virtual void fireExeeIncompleteReplenish(obj::cim::CimIncompleteReplenish& event) const = 0;
virtual void fireExeeIncompleteDeplete(obj::cim::CimIncompleteDeplete& event) const = 0;
virtual void fireExeeIncompleteRetract(obj::cim::CimIncompleteRetract& event) const = 0;
```

#### `IXfsEventsSrveUsreCim`

```cpp
virtual void fireSrveSafeDoorOpen() = 0;
virtual void fireSrveSafeDoorClosed() = 0;
virtual void fireUsreCashUnitThreshold(obj::cim::CimCashIn& event) const = 0;
virtual void fireSrveCashUnitInfoChanged(obj::cim::CimCashIn& event) const = 0;
virtual void fireSrveTellerInfoChanged(obj::xfs::ObjNumber& event) const = 0;
virtual void fireSrveItemsTaken(obj::cim::CimPositionInfo& event) const = 0;
virtual void fireSrveCountsChanged(obj::cim::CimCountsChanged& event) const = 0;
virtual void fireSrveItemsPresented(obj::cim::CimPositionInfo& event) const = 0;
virtual void fireSrveItemsInserted(obj::cim::CimPositionInfo& event) const = 0;
virtual void fireSrveMediaDetected(obj::cim::CimItemPosition& event) const = 0;
virtual void fireSrveDevicePosition(obj::cim::CimDevicePosition& event) const = 0;
virtual void fireSrvePowerSaveChange(obj::cim::CimPowerSaveChange& event) const = 0;
virtual void fireSrveShutterStatusChanged(obj::cim::CimShutterStatusChanged& event) const = 0;
virtual void fireSrveCountAccuarcyChanged(obj::cim::CimCashUnitCountStatus& event) const = 0;
```

#### `IXfsExecutorCim` — comandos execute

| Método | Descripción |
|---|---|
| `executeCashInStart` | Iniciar una sesión de recepción de efectivo. |
| `executeCashIn` | Recibir e identificar billetes. |
| `executeCashInEnd` | Finalizar la sesión de recepción (aceptar). |
| `executeCashInRollback` | Revertir la sesión de recepción (devolver billetes). |
| `executeRetract` | Retraer billetes al interior del receptor. |
| `executeOpenShutter` / `executeCloseShutter` | Controlar el obturador de entrada. |
| `executeSetTellerInfo` | Configurar información de cajero. |
| `executeSetCashUnitInfo` | Actualizar información de unidades de efectivo. |
| `executeStartExchange` / `executeEndExchange` | Gestionar ciclo de reposición de efectivo. |
| `executeOpenSafeDoor` | Abrir la puerta del safe. |
| `executeReset` | Resetear el dispositivo. |
| `executeConfigureCashInUnits` | Configurar unidades de recepción de efectivo. |
| `executeConfigureNotetypes` | Configurar tipos de billete aceptados. |
| `executeCreateP6Signature` | Crear firma P6 de los billetes. |
| `executeSetGuidanceLight` | Controlar luz de guía. |
| `executeConfigureNoteReader` | Configurar el lector de billetes. |
| `executeCompareP6Signature` | Comparar firmas P6 de billetes. |
| `executePowerSaveControl` | Gestionar modo de ahorro de energía. |
| `executeReplenish` | Reponer efectivo entre unidades. |
| `executeSetCashInLimit` | Establecer límite de recepción de efectivo. |
| `executeCashUnitCount` | Contar billetes en unidades físicas. |
| `executeDeviceLockControl` | Controlar el bloqueo del dispositivo. |
| `executeSetMode` | Establecer el modo de operación. |
| `executePresentMedia` | Presentar medios en la posición de salida. |
| `executeDeplete` | Vaciar efectivo entre unidades. |
| `executeSetBlacklist` | Configurar lista negra de billetes. |
| `executeSynchronizeCommand` | Sincronizar comandos. |
| `executeClassificationList` | Configurar lista de clasificación de billetes. |
| `executePreparePresent` | Preparar la presentación de medios. |

#### `IXfsExecutorCim` — comandos getInfo no cacheados

```cpp
virtual std::variant<TXfs::XFSError, TCim::CimError> infoTellerInfo(
    const obj::cim::CimTellerInfo& in, obj::cim::CimTellerDetails& out) = 0;
virtual std::variant<TXfs::XFSError, TCim::CimError> infoReplenishTarget(
    const obj::cim::CimRepInfo& in, obj::cim::CimRepInfoRes& out) = 0;
virtual TXfs::XFSError infoGetP6Signature(
    const obj::cim::CimGetP6Signature& in, obj::cim::CimP6Signature& out) = 0;
virtual TXfs::XFSError infoGetItemInfo(
    const obj::cim::CimGetItemInfo& in, obj::cim::CimItemInfo& out) = 0;
virtual TXfs::XFSError infoDepInfSource(
    const obj::cim::CimDepInfoSource& in, obj::cim::CimDepInfoRes& out) = 0;
virtual TXfs::XFSError infoGetAllItemsInfo(
    const obj::cim::CimGetAllItemsInfo& in, obj::cim::CimAllItemsInfo& out) = 0;
```

### `XfsBaseCim.h`

| Clase | Descripción |
|---|---|
| `CXfsBaseCim` | Implementación base concreta de `IXfsClassDevice` para CIM. Instancia internamente `CXfsCacheCim`, `CXfsEventsExeeCim` y `CXfsEventsSrveUsreCim`. Todos los comandos de `IXfsExecutorCim` tienen implementación base que devuelve `UnsuppCommand` / `UnsuppCategory`. |
| `CheckIdCim` | Enumeración de identificadores de check predefinidos (`Operative`, `Specific1`–`Specific6`). |
| `CheckCim` | Estructura que asocia un `CheckIdCim` con la función de check correspondiente. |

### `XfsEventsCim.h`

| Clase | Descripción |
|---|---|
| `CXfsEventsExeeCim` | Implementación de `IXfsEventsExeeCim`. Serializa los objetos de evento y los eleva al framework mediante `notifyExecuteEvent`. |
| `CXfsEventsSrveUsreCim` | Implementación de `IXfsEventsSrveUsreCim`. Eleva eventos de servicio (`notifyServiceEvent`) y usuario (`notifyUserEvent`). |

### `XfsCacheCim.h`

| Clase | Descripción |
|---|---|
| `CXfsCacheCim` | Implementación `final` de `IXfsCacheCim`. Thread-safe mediante `std::mutex` en cada acceso. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreExe` | `../../externs/XfsCoreExe/inc` | `IXfsClassDevice`, `IXfsCommonDevice`, gestión de eventos al framework |
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `XfsObjectsCim` | `../../externs/XfsObjectsCim/inc` | Tipos CIM (`obj::cim::*`, `TCim::*`) |
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


## Uso en proyectos consumidores

1. Añadir `../../externs/XfsBaseCim/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsBaseCim/lib/$(Platform)/$(Configuration)/XfsBaseCim.lib`.
3. Heredar `CXfsBaseCim` e implementar únicamente los métodos de `IXfsExecutorCim` que el dispositivo soporta.

```cpp
class MiReceptor : public CXfsBaseCim
{
protected:
    std::variant<TXfs::XFSError, TCim::CimError>
    executeCashInStart(const obj::cim::CimCashInStart& in) override;

    std::variant<TXfs::XFSError, TCim::CimError>
    executeCashIn(obj::cim::CimNoteNumberList& out,
                  const IXfsEventsExeeCim& ExeeEvents) override;

    std::variant<TXfs::XFSError, TCim::CimError>
    executeCashInEnd(obj::cim::CimCashInfo& out,
                     const IXfsEventsExeeCim& ExeeEvents) override;
    // ... otros comandos soportados
};
```

La caché y los eventos son accesibles desde la clase concreta mediante `getCache()` y `getEventsSrveUsre()`.

## Ventajas

- **Caché thread-safe:** `CXfsCacheCim` protege cada acceso con `std::mutex`, siendo seguro actualizarla desde el hilo de polling o desde suscripciones a eventos de la DevApi.
- **Implementación opcional por comando:** Los comandos no soportados por el hardware devuelven `UnsuppCommand` automáticamente sin necesidad de código en la clase concreta.
- **Sistema de checks extensible:** Cada comando expone una `checkList` virtual que la clase concreta puede sobreescribir para añadir validaciones previas a la ejecución (p.ej. `CheckIdCim::Operative`).
- **Separación de tipos de evento:** Los eventos execute, service y user se gestionan por implementaciones independientes, respetando el protocolo XFS.

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport  ──┐
XfsCoreExe      ──┤
XfsObjectsCim   ──┤──► XfsBaseCim ──► SpCim.exe
XfsObjects      ──┤
Log             ──┘
```
