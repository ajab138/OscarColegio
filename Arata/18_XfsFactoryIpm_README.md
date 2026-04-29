# XfsFactoryIpm

Biblioteca estática C++ que actúa como **fábrica de objetos SPI específica para procesadores de imagen (IPM)** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsFactoryIpm` implementa el patrón **Factory** para instanciar los objetos SPI (`IXfsSpiObject`) correspondientes a los comandos, categorías y eventos de la clase IPM:

- **Despacho por versión XFS:** enruta la creación de objetos a la sub-fábrica correcta (`CXfsFactoryIpm320` o `CXfsFactoryIpm330`) en función de la versión negociada.
- **Creación de objetos de entrada y salida** para los comandos `execute` y las categorías `getInfo`.
- **Creación de objetos de salida** para todos los eventos IPM (execute, service y user).

Es consumida por el core XFS (`xfsCoreDll`) para construir los objetos de serialización/deserialización de mensajes SPI sin conocer los tipos concretos de cada clase de dispositivo.

## Arquitectura

```
IXfsFactory          (XfsSpiObjects)
       │
       ▼
CXfsFactoryIpm
       │
       ├── CXfsFactoryIpm320  →  objSPI::ipm320::*   (XFS 3.20)
       └── CXfsFactoryIpm330  →  objSPI::ipm330::*   (XFS 3.30)
```

`CXfsFactoryIpm` implementa `IXfsFactory` e inspecciona la versión XFS negociada (`wVersion`) para delegar en la sub-fábrica adecuada. Si la versión no está soportada, los métodos devuelven `nullptr`.

## Clases publicadas

### `XfsFactoryIpm.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryIpm` | Implementación de `IXfsFactory` para IPM. Rango de versión soportado: `0x14031E03` (3.20–3.30). Clase de servicio: `TIpm::SERVICE_CLASS_IPM`. |

### `XfsFactoryIpm320.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryIpm320` | Sub-fábrica estática para XFS 3.20. Todos sus métodos son `static`. |

### `XfsFactoryIpm330.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryIpm330` | Sub-fábrica estática para XFS 3.30. Todos sus métodos son `static`. |

## Comandos y categorías soportados

### GetInfo — categorías de consulta

| Categoría | v3.20 | v3.30 |
|---|:---:|:---:|
| `Status` | ✔ | ✔ |
| `Capabilities` | ✔ | ✔ |
| `CodelineMapping` | ✔ | ✔ |
| `MediaBinInfo` | ✔ | ✔ |
| `TransactionStatus` | ✔ | ✔ |
| `MediaBinCapabilities` | — | ✔ |

### Execute — comandos de ejecución

| Comando | Descripción |
|---|---|
| `MediaIn` | Iniciar entrada de medios. |
| `MediaInEnd` | Finalizar entrada de medios. |
| `MediaInRollback` | Revertir la entrada de medios. |
| `ReadImage` | Capturar imagen del ítem. |
| `SetDestination` | Configurar destino del ítem. |
| `PresentMedia` | Presentar medios al cliente. |
| `RetractMedia` | Retraer medios al interior. |
| `PrintText` | Imprimir texto en el ítem. |
| `SetMediaBinInfo` | Actualizar información de bandejas. |
| `Reset` | Resetear el dispositivo. |
| `SetGuidanceLight` | Controlar luz de guía. |
| `GetNextItem` | Obtener el siguiente ítem de la transacción. |
| `ActionItem` | Aplicar acción sobre el ítem actual. |
| `ExpelMedia` | Expulsar medios. |
| `GetImageAfterPrint` | Capturar imagen del ítem tras la impresión. |
| `AcceptItem` | Aceptar el ítem en la bandeja de destino. |
| `SupplyReplenish` | Reponer suministros (tóner, tinta…). |
| `PowerSaveControl` | Gestionar modo de ahorro de energía. |
| `SetMode` | Configurar modo de operación del dispositivo. |
| `SynchronizeCommand` *(v3.30)* | Sincronizar comandos entre aplicaciones. |

### Eventos IPM

| Evento | Tipo | v3.20 | v3.30 |
|---|---|:---:|:---:|
| `ExeeNoMedia` | Execute | ✔ | ✔ |
| `ExeeMediaInserted` | Execute | ✔ | ✔ |
| `ExeeMediaBinError` | Execute | ✔ | ✔ |
| `ExeeMediaPresented` | Execute | ✔ | ✔ |
| `ExeeMediaRefused` | Execute | ✔ | ✔ |
| `ExeeMediaData` | Execute | ✔ | ✔ |
| `ExeeMediaRejected` | Execute | ✔ | ✔ |
| `SrveMediaBinInfoChanged` | Service | ✔ | ✔ |
| `SrveMediaTaken` | Service | ✔ | ✔ |
| `SrveMediaDetected` | Service | ✔ | ✔ |
| `SrveDevicePosition` | Service | ✔ | ✔ |
| `SrvePowerSaveChange` | Service | ✔ | ✔ |
| `SrveShutterStatusChanged` | Service *(v3.30)* | — | ✔ |
| `UsreMediaBinThreshold` | User | ✔ | ✔ |
| `UsreTonerThreshold` | User | ✔ | ✔ |
| `UsreScannerThreshold` | User | ✔ | ✔ |
| `UsreInkThreshold` | User | ✔ | ✔ |
| `UsreMicrThreshold` | User | ✔ | ✔ |

Los eventos no reconocidos como IPM se intentan interpretar como eventos de sistema (`TXfs::SystemEventId`) mediante `newSystemEvent`.

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | `IXfsFactory`, interfaces DLL del core XFS |
| `xfsObjects` | `../../externs/xfsObjects/inc` | Tipos XFS genéricos (`TXfs::*`) |
| `xfsObjectsIpm` | `../../externs/xfsObjectsIpm/inc` | Tipos IPM (`TIpm::IpmInfoCmd`, `TIpm::IpmExecCmd`, `TIpm::IpmEvent`) |
| `XfsSpiObjects` | `../../externs/XfsSpiObjects/inc` | `IXfsSpiObject`, `objSPI::xfs::ObjVoid` |
| `XfsSpiObjectsIpm320` | `../../externs/XfsSpiObjectsIpm320/inc` | Objetos SPI concretos para IPM v3.20 |
| `XfsSpiObjectsIpm330` | `../../externs/XfsSpiObjectsIpm330/inc` | Objetos SPI concretos para IPM v3.30 |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |

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
../../externs/XfsFactoryIpm/inc/                         ← cabeceras (.h)
../../externs/XfsFactoryIpm/lib/<Plat>/<Config>/         ← XfsFactoryIpm.lib
```

Esto permite que los proyectos ejecutables IPM consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsFactoryIpm/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsFactoryIpm/lib/$(Platform)/$(Configuration)/XfsFactoryIpm.lib`.
3. Registrar la instancia de `CXfsFactoryIpm` en el core XFS como fábrica del servicio IPM.

```cpp
#include <XfsFactoryIpm.h>

// Ejemplo de registro en el arranque del servicio IPM
IXfsFactory* pFactory = new CXfsFactoryIpm();
xfsCoreRegisterFactory(pFactory);
```

## Posición en el árbol de compilación (EXE)

```
xfsCoreSupport      ──┐
xfsCoreDll          ──┤
xfsObjects          ──┤
xfsObjectsIpm       ──┤──► XfsFactoryIpm ──► SpIpm.exe
XfsSpiObjects       ──┤
XfsSpiObjectsIpm320 ──┤
XfsSpiObjectsIpm330 ──┘
```
