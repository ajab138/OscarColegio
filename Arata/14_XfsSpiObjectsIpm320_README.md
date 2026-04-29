# XfsSpiObjectsIpm320

Biblioteca estática C++ que proporciona los **objetos SPI para procesadores de imagen (IPM) en formato XFS 3.20** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsSpiObjectsIpm320` contiene adaptadores SPI que envuelven los objetos de dominio IPM (`obj::ipm::*`) añadiendo dos capas de funcionalidad:

- **Serialización / deserialización** binaria y representación en texto (`toString`, `serialize`, `deserialize`).
- **Marshaling SPI** entre la representación interna del framework y las estructuras C de la API XFS 3.20 (`initialize`, `createStruct`, `createStructFromObject`).

Cada clase hereda del correspondiente objeto de dominio en `XfsObjectsIpm` y de `IXfsSpiObject`, siguiendo el patrón uniforme del framework para todos los dispositivos XFS.

## Arquitectura

```
obj::ipm::IpmXxx   (XfsObjectsIpm)        IXfsSpiObject   (xfsCoreSupport)
        │                                         │
        └──────────────┬───────────────────────────┘
                       ▼
           objSPI::ipm320::IpmXxx320
           (serialización + marshaling SPI 3.20)
```

Todos los objetos residen en el namespace `objSPI::ipm320` e implementan los contratos `IXfsSerializable` e `IXfsSPI`.

## Clases publicadas

### Consulta (`getInfo`)

| Clase | Objeto de dominio base | Descripción |
|---|---|---|
| `IpmCaps320` | `obj::ipm::IpmCaps` | Capacidades del dispositivo IPM. |
| `IpmStatus320` | `obj::ipm::IpmStatus` | Estado actual del dispositivo. |
| `IpmMediaBin320` | `obj::ipm::IpmMediaBin` | Descripción de un contenedor de medios. |
| `IpmMediaBinInfo320` | `obj::ipm::IpmMediaBinInfo` | Información detallada del contenedor de medios. |
| `IpmMediaStatus320` | `obj::ipm::IpmMediaStatus` | Estado del medio presente en el dispositivo. |
| `IpmTransStatus320` | `obj::ipm::IpmTransStatus` | Estado de la transacción en curso. |

### Comandos `execute` (entradas)

| Clase | Objeto de dominio base | Descripción |
|---|---|---|
| `IpmMediaIn320` | `obj::ipm::IpmMediaIn` | Parámetros para iniciar la entrada de medios. |
| `IpmMediaInEnd320` | `obj::ipm::IpmMediaInEnd` | Datos para finalizar la entrada de medios. |
| `IpmAcceptItem320` | `obj::ipm::IpmAcceptItem` | Aceptación de un ítem individual. |
| `IpmPresentMedia320` | `obj::ipm::IpmPresentMedia` | Presentación de medios al cliente. |
| `IpmRetractMedia320` | `obj::ipm::IpmRetractMedia` | Retracción de medios al interior del dispositivo. |
| `IpmReset320` | `obj::ipm::IpmReset` | Reset del dispositivo. |
| `IpmPrintText320` | `obj::ipm::IpmPrintText` | Impresión de texto sobre el medio. |
| `IpmReadImageIn320` | `obj::ipm::IpmReadImageIn` | Solicitud de captura de imagen. |
| `IpmSetMode320` | `obj::ipm::IpmSetMode` | Configuración del modo operativo. |
| `IpmSetDestination320` | `obj::ipm::IpmSetDestination` | Configuración del destino de los medios. |
| `IpmSetGuidLight320` | `obj::ipm::IpmSetGuidLight` | Control de la luz de guía. |
| `IpmScannerThreshold320` | `obj::ipm::IpmScannerThreshold` | Umbral del escáner óptico. |
| `IpmPowerSaveControl320` | `obj::ipm::IpmPowerSaveControl` | Control del modo de ahorro de energía. |

### Objetos de salida / resultado

| Clase | Objeto de dominio base | Descripción |
|---|---|---|
| `IpmImageData320` | `obj::ipm::IpmImageData` | Datos de imagen capturada. |
| `IpmMediaData320` | `obj::ipm::IpmMediaData` | Datos del medio procesado. |
| `IpmImageRequest320` | `obj::ipm::IpmImageRequest` | Solicitud de imagen asociada a un medio. |
| `IpmCodelineMapping320` | `obj::ipm::IpmCodelineMapping` | Mapeado de línea de código (entrada). |
| `IpmCodelineMappingOut320` | `obj::ipm::IpmCodelineMappingOut` | Mapeado de línea de código (salida). |
| `IpmRetractMediaOut320` | `obj::ipm::IpmRetractMediaOut` | Resultado de una operación de retracción. |
| `IpmNextItemOut320` | `obj::ipm::IpmNextItemOut` | Información del siguiente ítem a procesar. |
| `IpmGetImageAfterPrint320` | `obj::ipm::IpmGetImageAfterPrint` | Imagen obtenida tras operación de impresión. |
| `IpmMediaInRequest320` | `obj::ipm::IpmMediaInRequest` | Solicitud de entrada de medios pendiente. |

### Objetos de evento

| Clase | Objeto de dominio base | Descripción |
|---|---|---|
| `IpmMediaDetected320` | `obj::ipm::IpmMediaDetected` | Medio detectado en el dispositivo. |
| `IpmMediaPresented320` | `obj::ipm::IpmMediaPresented` | Medio presentado en la ranura de salida. |
| `IpmMediaRefused320` | `obj::ipm::IpmMediaRefused` | Medio rechazado durante la entrada. |
| `IpmMediaRejected320` | `obj::ipm::IpmMediaRejected` | Medio rechazado tras procesamiento. |
| `IpmMbError320` | `obj::ipm::IpmMbError` | Error producido en un contenedor de medios. |
| `IpmSupplyReplen320` | `obj::ipm::IpmSupplyReplen` | Notificación de reposición de suministros. |
| `IpmThreshold320` | `obj::ipm::IpmThreshold` | Umbral de nivel de medios alcanzado. |
| `IpmPowerSaveChange320` | `obj::ipm::IpmPowerSaveChange` | Cambio en el modo de ahorro de energía. |
| `IpmDevicePosition320` | `obj::ipm::IpmDevicePosition` | Cambio de posición del dispositivo. |

### Posición y geometría

| Clase | Objeto de dominio base | Descripción |
|---|---|---|
| `IpmPos320` | `obj::ipm::IpmPos` | Posición lógica dentro del dispositivo. |
| `IpmPosCaps320` | `obj::ipm::IpmPosCaps` | Capacidades de posición del dispositivo. |
| `IpmPosition320` | `obj::ipm::IpmPosition` | Posición física del dispositivo. |
| `IpmMediaSize320` | `obj::ipm::IpmMediaSize` | Dimensiones del medio procesado. |
| `IpmPrintSize320` | `obj::ipm::IpmPrintSize` | Dimensiones del área de impresión. |

### Datos extendidos

| Clase | Objeto de dominio base | Descripción |
|---|---|---|
| `IpmXData320` | `obj::ipm::IpmXData` | Datos extendidos (XData) de uso específico por dispositivo. |

## Interfaz común de cada objeto SPI

Todas las clases implementan los métodos de `IXfsSerializable` e `IXfsSPI`:

```cpp
// Serialización
virtual void toString(std::stringstream& stream, uint8_t indent = 0) override final;
virtual void serialize(std::vector<uint8_t>& buffer) override final;
virtual void deserialize(const std::vector<uint8_t>& buffer) override final;

// Marshaling SPI ↔ estructuras XFS 3.20
virtual void initialize(LPVOID lpData) override final;
virtual void createStruct(LPVOID lpWFSResult) override final;
virtual void createStructFromObject(LPVOID lpWFSResult, const void* lpObjectBase, LPVOID lpXfsStruct) override final;
```

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IXfsSpiObject`, `IXfsSerializable`, `IXfsSPI`, tipos de soporte del core |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | Tipos y definiciones de la DLL del core XFS |
| `XfsObjectsIpm` | `../../externs/XfsObjectsIpm/inc` | Objetos de dominio IPM (`obj::ipm::*`) |
| `xfs320` | `../../externs/xfs320/inc` | Estructuras C de la API XFS 3.20 para IPM |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
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
../../externs/XfsSpiObjectsIpm320/inc/                     ← cabeceras (.h)
../../externs/XfsSpiObjectsIpm320/lib/<Plat>/<Config>/     ← XfsSpiObjectsIpm320.lib
```

Esto permite que los proyectos consumidores (SPI IPM) enlacen la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjectsIpm320/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjectsIpm320/lib/$(Platform)/$(Configuration)/XfsSpiObjectsIpm320.lib`.
3. Incluir la cabecera agregadora `XfsSpiObjectsIpm320.h` para acceder a todos los objetos SPI en un solo `#include`.

```cpp
#include "XfsSpiObjectsIpm320.h"

// Crear y poblar un objeto SPI a partir de una estructura XFS 3.20
objSPI::ipm320::IpmStatus320 status;
status.initialize(lpWFSResult->lpBuffer);

// Serializar para envío entre procesos
std::vector<uint8_t> buffer;
status.serialize(buffer);

// Reconstruir en el receptor
objSPI::ipm320::IpmStatus320 statusRemote;
statusRemote.deserialize(buffer);

// Generar estructura XFS 3.20 de vuelta
statusRemote.createStruct(lpOutput);
```

## Posición en el árbol de compilación (SPI)

```
xfsCoreSupport  ──┐
xfsCoreDll      ──┤
XfsObjectsIpm   ──┤──► XfsSpiObjectsIpm320 ──► SpIpm.dll
xfs320          ──┤
Log             ──┘
```
