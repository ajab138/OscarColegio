# XfsSpiObjectsIpm330

Biblioteca estática C++ que implementa los **objetos SPI de IPM para la versión XFS 3.30** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsSpiObjectsIpm330` proporciona las clases concretas que traducen entre los objetos de dominio IPM genéricos (`obj::ipm::*`) y las estructuras binarias del protocolo XFS 3.30 (`WFSIPM*`):

- **Serialización / deserialización** de cada tipo IPM hacia/desde el buffer binario del SPI.
- **Inicialización** de objetos de dominio a partir de punteros de estructura XFS (`LPVOID lpData`).
- **Creación de estructuras XFS** (`LPWFSRESULT`) a partir de objetos de dominio, con soporte de herencia de puntero base.
- **Trazabilidad** mediante `toString` en formato legible para depuración y logging.

Todas las clases residen en el espacio de nombres `objSPI::ipm330` y son la contraparte de versión específica de los tipos neutros de `XfsObjectsIpm`.

## Arquitectura

```
obj::ipm::IpmXxx          (XfsObjectsIpm — objeto de dominio genérico)
       │
       ▼
IpmXxx330  ──── IXfsSpiObject
       │              │
       │              ├── IXfsSerializable  (serialize / deserialize / toString)
       │              └── IXfsSPI           (initialize / createStruct / createStructFromObject)
       │
       └── [implementación final, namespace objSPI::ipm330]
```

Cada clase hereda en doble vía: el **modelo de datos** del objeto IPM base y el **contrato SPI** de `IXfsSpiObject`. El método `initialize` rellena el objeto desde la estructura XFS 3.30; `createStruct` / `createStructFromObject` realizan la conversión inversa.

## Clases publicadas

### Cabecera de inclusión única: `XfsSpiObjectsIpm330.h`

Incluye automáticamente todas las clases de la biblioteca.

---

### Estado e información del dispositivo

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmStatus330` | `obj::ipm::IpmStatus` | Estado operativo del módulo IPM. |
| `IpmCaps330` | `obj::ipm::IpmCaps` | Capacidades del hardware IPM. |
| `IpmTransStatus330` | `obj::ipm::IpmTransStatus` | Estado de la transacción en curso. |

### Bandejas de documentos (Media Bins)

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmMediaBin330` | `obj::ipm::IpmMediaBin` | Bandeja lógica de documentos. |
| `IpmMediaBinCaps330` | `obj::ipm::IpmMediaBinCaps` | Capacidades de la bandeja de documentos. |
| `IpmMediaBinInfo330` | `obj::ipm::IpmMediaBinInfo` | Información detallada de la bandeja. |
| `IpmBinCaps330` | `obj::ipm::IpmBinCaps` | Capacidades de bin físico. |
| `IpmMbError330` | `obj::ipm::IpmMbError` | Detalle de error asociado a una bandeja. |

### Posiciones

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmPos330` | `obj::ipm::IpmPos` | Posición lógica de medios en el dispositivo. |
| `IpmPosCaps330` | `obj::ipm::IpmPosCaps` | Capacidades de una posición. |
| `IpmPosition330` | `obj::ipm::IpmPosition` | Posición física dentro del dispositivo. |
| `IpmNextItemOut330` | `obj::ipm::IpmNextItemOut` | Siguiente ítem de salida programado. |

### Entrada de documentos

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmMediaIn330` | `obj::ipm::IpmMediaIn` | Parámetros del comando de entrada de documentos. |
| `IpmMediaInEnd330` | `obj::ipm::IpmMediaInEnd` | Resultado del fin de ciclo de entrada. |
| `IpmMediaInRequest330` | `obj::ipm::IpmMediaInRequest` | Solicitud de introducción de documento. |
| `IpmAcceptItem330` | `obj::ipm::IpmAcceptItem` | Aceptación de un ítem durante la entrada. |
| `IpmMediaRefused330` | `obj::ipm::IpmMediaRefused` | Documento rechazado en la ranura de entrada. |
| `IpmMediaRejected330` | `obj::ipm::IpmMediaRejected` | Documento rechazado y enviado a la posición de rechazo. |

### Datos e imágenes

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmImageData330` | `obj::ipm::IpmImageData` | Datos de una imagen capturada del documento. |
| `IpmImageRequest330` | `obj::ipm::IpmImageRequest` | Solicitud de captura de imagen. |
| `IpmReadImageIn330` | `obj::ipm::IpmReadImageIn` | Parámetros de entrada para lectura de imagen. |
| `IpmGetImageAfterPrint330` | `obj::ipm::IpmGetImageAfterPrint` | Imagen obtenida tras operación de impresión. |
| `IpmMediaData330` | `obj::ipm::IpmMediaData` | Datos completos del documento procesado. |
| `IpmMediaDetected330` | `obj::ipm::IpmMediaDetected` | Notificación de documento detectado en el dispositivo. |
| `IpmMediaSize330` | `obj::ipm::IpmMediaSize` | Dimensiones físicas del documento. |

### Codeline (MICR / OCR)

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmCodelineMapping330` | `obj::ipm::IpmCodelineMapping` | Mapeo de campos de la línea de código (MICR/OCR). |
| `IpmCodelineMappingOut330` | `obj::ipm::IpmCodelineMappingOut` | Resultado del mapeo de codeline. |

### Presentación y retracción

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmPresentMedia330` | `obj::ipm::IpmPresentMedia` | Parámetros del comando de presentación de documentos. |
| `IpmMediaPresented330` | `obj::ipm::IpmMediaPresented` | Notificación de documentos presentados al usuario. |
| `IpmRetractMedia330` | `obj::ipm::IpmRetractMedia` | Parámetros del comando de retracción de documentos. |
| `IpmRetractMediaOut330` | `obj::ipm::IpmRetractMediaOut` | Resultado de la operación de retracción. |
| `IpmMediaStatus330` | `obj::ipm::IpmMediaStatus` | Estado actual de los documentos en el dispositivo. |

### Impresión

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmPrintText330` | `obj::ipm::IpmPrintText` | Parámetros del comando de impresión de texto en el documento. |
| `IpmPrintSize330` | `obj::ipm::IpmPrintSize` | Área de impresión disponible. |

### Comandos de control y configuración

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmReset330` | `obj::ipm::IpmReset` | Parámetros del comando reset del dispositivo. |
| `IpmSetMode330` | `obj::ipm::IpmSetMode` | Configurar modo de operación del IPM. |
| `IpmSetDestination330` | `obj::ipm::IpmSetDestination` | Configurar destino de los documentos procesados. |
| `IpmSetGuidLight330` | `obj::ipm::IpmSetGuidLight` | Control de la luz de guía del dispositivo. |
| `IpmPowerSaveControl330` | `obj::ipm::IpmPowerSaveControl` | Parámetros de control de ahorro de energía. |
| `IpmScannerThreshold330` | `obj::ipm::IpmScannerThreshold` | Umbral de sensibilidad del escáner. |
| `IpmThreshold330` | `obj::ipm::IpmThreshold` | Umbral genérico del dispositivo. |
| `IpmSynchronizeCommand330` | `obj::ipm::IpmSynchronizeCommand` | Parámetros de sincronización de comandos. |
| `IpmSupplyReplen330` | `obj::ipm::IpmSupplyReplen` | Notificación de reposición de suministro. |
| `IpmXData330` | `obj::ipm::IpmXData` | Datos extendidos (extra data) de transacción. |

### Eventos (objetos de notificación)

| Clase | Base IPM | Descripción |
|---|---|---|
| `IpmDevicePosition330` | `obj::ipm::IpmDevicePosition` | Notificación de cambio de posición del dispositivo. |
| `IpmPowerSaveChange330` | `obj::ipm::IpmPowerSaveChange` | Notificación de cambio de modo de ahorro de energía. |
| `IpmShutterStatusChanged330` | `obj::ipm::IpmShutterStatusChanged` | Notificación de cambio de estado del obturador. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IXfsSpiObject`, `IXfsSPI`, `IXfsSerializable` |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | Interfaces de la DLL XFS |
| `XfsObjectsIpm` | `../../externs/XfsObjectsIpm/inc` | Tipos base `obj::ipm::*` |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `xfs330` | `../../externs/xfs330/inc` | Estructuras binarias del protocolo XFS 3.30 (`WFSIPM*`) |

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
../../externs/XfsSpiObjectsIpm330/inc/                         ← cabeceras (.h)
../../externs/XfsSpiObjectsIpm330/lib/<Plat>/<Config>/         ← XfsSpiObjectsIpm330.lib
```

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjectsIpm330/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjectsIpm330/lib/$(Platform)/$(Configuration)/XfsSpiObjectsIpm330.lib`.
3. Incluir la cabecera de entrada única para acceder a todas las clases:

```cpp
#include "XfsSpiObjectsIpm330.h"

// Inicializar un objeto de estado a partir de una estructura XFS 3.30
objSPI::ipm330::IpmStatus330 status;
status.initialize(lpWfsResult->lpBuffer);

// Serializar para transmisión
std::vector<uint8_t> buffer;
status.serialize(buffer);

// Crear estructura XFS a partir del objeto de dominio
status.createStruct(lpWfsResult);
```

## Posición en el árbol de compilación (SPI)

```
xfs330            ──┐
xfsCoreSupport    ──┤
xfsCoreDll        ──┤──► XfsSpiObjectsIpm330 ──► SpIpm.dll (SPI)
XfsObjectsIpm     ──┤
Log               ──┘
```
