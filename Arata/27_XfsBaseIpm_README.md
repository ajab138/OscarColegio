# XfsBaseIpm

Biblioteca estática C++ que actúa como **capa base específica para dispositivos de imagen y proceso de medios (IPM)** dentro del framework XFS ARATA (eXtensions for Financial Services) en el ecosistema `SpG600_XfsCore`.

## Propósito

`XfsBaseIpm` implementa el patrón **Template Method** para gestionar la lógica común del servicio IPM:

- **Despacho genérico** de comandos `getInfo` y `execute` de la clase IPM, con soporte de caché thread-safe para las categorías cacheables.
- **Delegación** de la ejecución concreta de cada comando en la interfaz `IXfsExecutorIpm`, que deben implementar las clases de dispositivo concretas.
- **Publicación de eventos** IPM (execute, service y user) hacia el framework XFS.

Es la base de la que heredan las implementaciones concretas de dispositivos de imagen y proceso de medios (`SpIpm`, etc.).

## Arquitectura

```
IXfsClassDevice          (XfsCoreExe)
       │
       ▼
CXfsBaseIpm  ◄──── IXfsExecutorIpm  (implementada por cada clase concreta de IPM)
       │
       ├── CXfsCacheIpm          →  IXfsCacheIpm
       ├── CXfsEventsExeeIpm     →  IXfsEventsExeeIpm
       └── CXfsEventsSrveUsreIpm →  IXfsEventsSrveUsreIpm
```

`CXfsBaseIpm` implementa `IXfsClassDevice` de forma `final` para `processGetInfo` y `processExecute`, gestionando internamente la caché y el despacho de eventos. Las clases concretas solo necesitan implementar los métodos de `IXfsExecutorIpm` correspondientes a los comandos que soportan.

## Interfaces publicadas

### `IXfsBaseIpm.h`

| Interfaz | Descripción |
|---|---|
| `IXfsCacheIpm` | Contrato para acceder y actualizar la caché de información IPM. |
| `IXfsEventsExeeIpm` | Publica eventos de ejecución IPM hacia el framework. |
| `IXfsEventsSrveUsreIpm` | Publica eventos de servicio y usuario IPM hacia el framework. |
| `IXfsExecutorIpm` | Contrato que toda clase concreta IPM debe implementar. |

#### `IXfsCacheIpm`

Almacena las categorías de `getInfo` que pueden cachearse. Acceso protegido mediante `mutex`.

| Método | Tipo |
|---|---|
| `setCacheStatus` / `getCacheStatus` | `obj::ipm::IpmStatus` |
| `setCacheCapabilities` / `getCacheCapabilities` | `obj::ipm::IpmCaps` |
| `setCacheMediaBinInfo` / `getCacheMediaBinInfo` | `obj::ipm::IpmMediaBinInfo` |
| `setTransactionStatus` / `getCacheTransactionStatus` | `obj::ipm::IpmTransStatus` |
| `setMediaBinCapabilities` / `getCacheMediaBinCapabilities` | `obj::ipm::IpmMediaBinCaps` |

#### `IXfsEventsExeeIpm`

```cpp
virtual void fireExeeNoMedia() const = 0;
virtual void fireExeeMediaInserted() const = 0;
virtual void fireExeeMediaBinError(obj::ipm::IpmMbError& event) const = 0;
virtual void fireExeeMediaPresented(obj::ipm::IpmMediaPresented& event) const = 0;
virtual void fireExeeMediaRefused(obj::ipm::IpmMediaRefused& event) const = 0;
virtual void fireExeeMediaData(obj::ipm::IpmMediaData& event) const = 0;
virtual void fireExeeMediaRejected(obj::ipm::IpmMediaRejected& event) const = 0;
```

#### `IXfsEventsSrveUsreIpm`

```cpp
virtual void fireUsreMediaBinThreshold(obj::ipm::IpmMediaBin& event) const = 0;
virtual void fireSrveMediaBinInfoChanged(obj::ipm::IpmMediaBin& event) const = 0;
virtual void fireSrveMediaTaken(obj::ipm::IpmPosition& event) const = 0;
virtual void fireUsreTonerThreshold(obj::ipm::IpmThreshold& event) const = 0;
virtual void fireUsreScannerThreshold(obj::ipm::IpmScannerThreshold& event) const = 0;
virtual void fireUsreIpmInkThreshold(obj::ipm::IpmThreshold& event) const = 0;
virtual void fireSrveIpmMediaDetected(obj::ipm::IpmMediaDetected& event) const = 0;
virtual void fireUsreIpmMicrThreshold(obj::ipm::IpmThreshold& event) const = 0;
virtual void fireSrveDevicePosition(obj::ipm::IpmDevicePosition& event) const = 0;
virtual void fireSrvePowerSaveChange(obj::ipm::IpmPowerSaveChange& event) const = 0;
virtual void fireSrveShutterStatusChanged(obj::ipm::IpmShutterStatusChanged& event) const = 0;
```

#### `IXfsExecutorIpm` — comandos execute

| Método | Descripción |
|---|---|
| `executeMediaInfo` | Iniciar sesión de entrada de medios. |
| `executeMediaInEnd` | Finalizar sesión de entrada de medios. |
| `executeMediaInRollback` | Revertir la sesión de entrada de medios en curso. |
| `executeReadImage` | Leer la imagen capturada de un medio. |
| `executeSetDestination` | Establecer el destino de los medios procesados. |
| `executePresentMedia` | Presentar medios en la posición de salida. |
| `executeRetractMedia` | Retraer medios al interior del dispositivo. |
| `executePrintText` | Imprimir texto sobre el medio. |
| `executeSetMediaBinInfo` | Actualizar información de los contenedores de medios. |
| `executeResetCount` | Reiniciar contadores del dispositivo. |
| `executeSetGuidanceLight` | Controlar la luz de guía del dispositivo. |
| `executeGetNextItem` | Obtener el siguiente ítem en la cola de procesamiento. |
| `executeActionItem` | Ejecutar la acción definida sobre el ítem actual. |
| `executeExpelMedia` | Expulsar el medio del dispositivo. |
| `executeGetImageAfterPrint` | Obtener la imagen del medio tras la impresión. |
| `executeAcceptItem` | Aceptar el ítem procesado. |
| `executeSupplyReplenish` | Reposición de suministros (tóner, tinta, etc.). |
| `executePowerSaveControl` | Gestionar modo de ahorro de energía. |
| `executeSetMode` | Configurar el modo de operación del dispositivo. |
| `executeSynchronizeCommand` | Sincronizar comandos. |

#### `IXfsExecutorIpm` — comandos getInfo no cacheados

```cpp
virtual TXfs::XFSError infoCodelineMapping(
    const obj::ipm::IpmCodelineMapping& in, obj::ipm::IpmCodelineMappingOut& out) = 0;
```

### `XfsBaseIpm.h`

| Clase | Descripción |
|---|---|
| `CXfsBaseIpm` | Implementación base concreta de `IXfsClassDevice` para IPM. Instancia internamente `CXfsCacheIpm`, `CXfsEventsExeeIpm` y `CXfsEventsSrveUsreIpm`. Todos los comandos de `IXfsExecutorIpm` tienen implementación base que devuelve `UnsuppCommand` / `UnsuppCategory`. |

### `XfsEventsIpm.h`

| Clase | Descripción |
|---|---|
| `CXfsEventsExeeIpm` | Implementación de `IXfsEventsExeeIpm`. Serializa los objetos de evento y los eleva al framework mediante `notifyExecuteEvent`. |
| `CXfsEventsSrveUsreIpm` | Implementación de `IXfsEventsSrveUsreIpm`. Eleva eventos de servicio (`notifyServiceEvent`) y usuario (`notifyUserEvent`). |

### `XfsCacheIpm.h`

| Clase | Descripción |
|---|---|
| `CXfsCacheIpm` | Implementación `final` de `IXfsCacheIpm`. Thread-safe mediante `std::mutex` en cada acceso. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreExe` | `../../externs/XfsCoreExe/inc` | `IXfsClassDevice`, `IXfsCommonDevice`, gestión de eventos al framework |
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `XfsObjectsIpm` | `../../externs/XfsObjectsIpm/inc` | Tipos IPM (`obj::ipm::*`, `TIpm::*`) |
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
../../externs/XfsBaseIpm/inc/                          ← cabeceras (.h)
../../externs/XfsBaseIpm/lib/<Plat>/<Config>/          ← XfsBaseIpm.lib
```

Esto permite que los proyectos ejecutables de IPM consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsBaseIpm/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsBaseIpm/lib/$(Platform)/$(Configuration)/XfsBaseIpm.lib`.
3. Heredar `CXfsBaseIpm` e implementar únicamente los métodos de `IXfsExecutorIpm` que el dispositivo soporta.

```cpp
class MiDispositivoIpm : public CXfsBaseIpm
{
protected:
    std::variant<TXfs::XFSError, TIpm::IpmError>
    executeMediaInfo(const obj::ipm::IpmMediaInRequest& in,
                     obj::ipm::IpmMediaBin out,
                     const IXfsEventsExeeIpm& exeeEvents) override;

    std::variant<TXfs::XFSError, TIpm::IpmError>
    executeMediaInEnd(obj::ipm::IpmMediaInEnd& out,
                      const IXfsEventsExeeIpm& exeeEvents) override;
    // ... otros comandos soportados
};
```

La caché y los eventos son accesibles desde la clase concreta mediante `getCache()` y `getEventsSrveUsre()`.

## Ventajas

- **Caché thread-safe:** `CXfsCacheIpm` protege cada acceso con `std::mutex`, siendo seguro actualizarla desde el hilo de polling o desde suscripciones a eventos de la DevApi.
- **Implementación opcional por comando:** Los comandos no soportados por el hardware devuelven `UnsuppCommand` automáticamente sin necesidad de código en la clase concreta.
- **Separación de tipos de evento:** Los eventos execute, service y user se gestionan por implementaciones independientes, respetando el protocolo XFS.

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport  ──┐
XfsCoreExe      ──┤
XfsObjectsIpm   ──┤──► XfsBaseIpm ──► SpIpm.exe
XfsObjects      ──┤
Log             ──┘
```
