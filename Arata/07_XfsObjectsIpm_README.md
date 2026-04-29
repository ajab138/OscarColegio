# XfsObjectsIpm

Biblioteca estática C++ que proporciona los **tipos de datos IPM** (Image Processing Module) del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsObjectsIpm` define todas las clases C++ que representan estructuras de datos del estándar XFS IPM (versión 3.40):

- **Objetos de comandos** `getInfo` y `execute`: parámetros de entrada y resultados de salida para cada comando IPM.
- **Objetos de eventos**: estructuras empleadas en los eventos execute, service y user del servicio IPM.
- **Tipos enumerados** (`TIpm::*`): mapeo tipado y sin dependencias de los `#define` de `xfsipm.h` a `enum class` de C++11.
- **Serialización** bidireccional vía Protobuf (`proto::ipm::*`) e interfaz `IXfsSerializable`.

Es consumida por `XfsBaseIpm` y por las implementaciones concretas de módulos de procesamiento de imágenes de cheques y documentos.

## Arquitectura

Todos los objetos de datos residen en el namespace `obj::ipm` e implementan la interfaz `IXfsSerializable`:

```
IXfsSerializable      (XfsCoreSupport)
        │
        ▼
obj::ipm::<Clase>  ←──── proto::ipm::<Proto>   (XfsProtos / Protobuf)
```

Cada clase proporciona:

- Constructor por defecto, constructor de copia y operador de asignación.
- Operadores `==` y `!=`.
- `toString(stream, indent)` — representación textual para trazas/log.
- `serialize(buffer)` / `deserialize(buffer)` — serialización binaria vía Protobuf.
- `toProto()` / `fromProto()` / `setProto(proto)` — acceso directo al mensaje Protobuf subyacente.

## Clases de objetos (`obj::ipm::*`)

### Objetos de resultado getInfo

| Clase | Comando IPM | Descripción |
|---|---|---|
| `IpmStatus` | `Status` (1601) | Estado del dispositivo: acceptor, media, toner, tinta, escáneres, MICR, stacker, re-buncher, feeder, posiciones, guid lights, posición del dispositivo, modo mixto, módulo antifraude. |
| `IpmCaps` | `Capabilities` (1602) | Capacidades del dispositivo: tipo de entrada, máximo media en stacker, formatos de imagen, escaneado, codeline, posiciones, retracción, modo mixto, etc. |
| `IpmCodelineMappingOut` | `CodelineMapping` (1603) | Mapeo de codeline de salida: formato y datos de mapeo de caracteres. |
| `IpmMediaBinInfo` | `MediaBinInfo` (1604) | Información de las papeleras de medios: colección de `IpmMediaBin`. |
| `IpmTransStatus` | `TransactionStatus` (1605) | Estado de transacción en curso: contadores de medios, estado de la transacción, información por ítem (`IpmMediaStatus`). |
| `IpmBinCaps` | `MediaBinCapabilities` (1606) | Capacidades de las papeleras: colección de `IpmMediaBinCaps`. |

### Parámetros de entrada getInfo

| Clase | Descripción |
|---|---|
| `IpmCodelineMapping` | Parámetro de entrada para `CodelineMapping`: formato de codeline solicitado. |

### Objetos de comandos execute (entrada)

| Clase | Comando IPM | Descripción |
|---|---|---|
| `IpmMediaInRequest` | `MediaIn` (1601) | Parámetros de entrada para iniciar recepción de medios: formato de codeline, solicitudes de imagen, máximo en stacker, modo de rechazo por aplicación. |
| `IpmReadImageIn` | `ReadImage` (1604) | Parámetros de lectura de imagen: ID de medio, formato de codeline, solicitudes de imagen. |
| `IpmSetDestination` | `SetDestination` (1605) | Destino de un ítem: ID de medio y número de papelera. |
| `IpmPresentMedia` | `PresentMedia` (1606) | Presentar medios en la posición indicada. |
| `IpmRetractMedia` | `RetractMedia` (1607) | Parámetros de retracción: ubicación y número de papelera. |
| `IpmPrintText` | `PrintText` (1608) | Impresión de texto en un medio: ID de medio, flag de stamp y datos de texto. |
| `IpmReset` | `Reset` (1610) | Reset del dispositivo: control de media y número de papelera. |
| `IpmSetGuidLight` | `SetGuidanceLight` (1611) | Control de luz de guía: índice de luz y comando de estado. |
| `IpmGetImageAfterPrint` | `GetImageAfterPrint` (1615) | Obtener imagen tras impresión: ID de medio y solicitudes de imagen. |
| `IpmAcceptItem` | `AcceptItem` (1616) | Aceptar o rechazar el ítem actual en modo `ApplicationRefuse`. |
| `IpmSupplyReplen` | `SupplyReplenish` (1617) | Reposición de consumibles: tóner e tinta. |
| `IpmPowerSaveControl` | `PowerSaveControl` (1618) | Control de ahorro de energía: tiempo máximo de recuperación. |
| `IpmSetMode` | `SetMode` (1619) | Configurar modo mixto de operación. |
| `IpmSynchronizeCommand` | `SynchronizeCommand` (1620) | Sincronización de comandos: comando y datos de sincronización. |

### Objetos de resultado execute

| Clase | Comando IPM | Descripción |
|---|---|---|
| `IpmMediaIn` | `MediaIn` (1601) | Resultado intermedio de `MediaIn`: contadores de medios en stacker y estado del feeder. |
| `IpmMediaInEnd` | `MediaInEnd` (1602) | Resultado de fin de `MediaIn`: ítems devueltos, rechazados, bunches rechazados e información de papeleras. |
| `IpmRetractMediaOut` | `RetractMedia` (1607) | Resultado de retracción: cantidad de media, ubicación y papelera destino. |
| `IpmNextItemOut` | `GetNextItem` (1612) | Resultado de obtención del siguiente ítem: estado del feeder. |

### Objetos de eventos

| Clase | Evento IPM | Descripción |
|---|---|---|
| `IpmMbError` | `ExeeMediaBinError` (1605) | Error en papelera de medios durante ejecución: motivo y papelera afectada. |
| `IpmMediaPresented` | `ExeeMediaPresented` (1611) | Medios presentados al cliente: posición, índice de bunch y total de bunches. |
| `IpmMediaRefused` | `ExeeMediaRefused` (1612) | Medio rechazado: razón, ubicación, flag de presentación requerida y tamaño del medio. |
| `IpmMediaData` | `ExeeMediaData` (1613) | Datos capturados de un medio: ID, datos de codeline, indicador MICR, imágenes, orientación, tamaño y validez. |
| `IpmMediaRejected` | `ExeeMediaRejected` (1615) | Medio rechazado físicamente: razón de rechazo. |
| `IpmThreshold` | `UsreTonerThreshold` / `UsreInkThreshold` / `UsreMicrThreshold` | Umbral de consumible alcanzado: valor del umbral. |
| `IpmScannerThreshold` | `UsreScannerThreshold` (1608) | Umbral de escáner alcanzado: escáner afectado y nivel umbral. |
| `IpmMediaDetected` | `SrveMediaDetected` (1610) | Medio detectado en el dispositivo: posición y número de papelera de retracción. |
| `IpmDevicePosition` | `SrveDevicePosition` (1616) | Cambio de posición física del dispositivo. |
| `IpmPowerSaveChange` | `SrvePowerSaveChange` (1617) | Cambio de estado del modo ahorro de energía: tiempo de recuperación. |
| `IpmShutterStatusChanged` | `SrveShutterStatusChanged` (1618) | Cambio de estado del obturador: posición y estado. |
| `IpmPosition` | `SrveMediaTaken` (1606) | Posición genérica: posición lógica del medio. |

### Objetos auxiliares

| Clase | Descripción |
|---|---|
| `IpmPos` | Posición física IPM: estado del obturador, estado de posición, transporte, estado de media en transporte y posición del obturador atascado. |
| `IpmPosCaps` | Capacidades de una posición: sensores de toma/inserción de ítems y áreas de retracción soportadas. |
| `IpmPrintSize` | Tamaño de área de impresión: filas y columnas. |
| `IpmImageRequest` | Solicitud de imagen: fuente (front/back), tipo, formato de color, color de escaneado y ruta de fichero. |
| `IpmImageData` | Datos de imagen capturada: fuente, tipo, formato de color, color de escaneado, estado de la imagen y ruta de fichero. |
| `IpmMediaBin` | Papelera de medios: número, nombre de posición, tipo, tipo de media, ID, contadores (entrada, recuento, retracciones), sensores, máximos y estado. |
| `IpmMediaBinCaps` | Capacidades de una papelera: número, nombre, sensores hardware/ítem, máximos y datos extra. |
| `IpmMediaBinInfo` | Colección de papeleras de medios: contador y vector de `IpmMediaBin`. |
| `IpmMediaStatus` | Estado de un ítem en transacción: ID, ubicación, papelera, datos de codeline, indicador MICR, imágenes, orientación, tamaño, validez y acceso de cliente. |
| `IpmMediaSize` | Tamaño físico de un medio: dimensiones X e Y en unidades XFS. |
| `IpmXData` | Datos hexadecimales genéricos: longitud y buffer de bytes. |
| `IpmCodelineMappingOut` | Mapeo de codeline: formato y datos de mapeo extendido (`IpmXData`). |

## Tipos enumerados (`TIpm::*`)

Definidos en `TIpm.h`, generados a partir de `xfsipm.h` versión 3.40. Sin dependencia de cabeceras XFS o Windows.

| Enum class | Descripción |
|---|---|
| `IpmInfoCmd` | Comandos getInfo IPM (1601–1606). |
| `IpmExecCmd` | Comandos execute IPM (1601–1620). |
| `IpmEvent` | Eventos IPM execute/service/user (1601–1618). |
| `IpmDeviceStatus` | Estado del hardware del módulo IPM. |
| `IpmAcceptor` | Estado del acceptor / papeleras. |
| `IpmMedia` | Presencia de media en el dispositivo. |
| `IpmToner` | Nivel de tóner. |
| `IpmInk` | Nivel de tinta. |
| `IpmScanner` | Estado del escáner (frontal / trasero). |
| `IpmMicr` | Estado del lector MICR. |
| `IpmStacker` | Estado del stacker. |
| `IpmReBuncher` | Estado del re-buncher. |
| `IpmMediaFeeder` | Estado del alimentador de medios. |
| `IpmDevicePosition` | Posición física del dispositivo. |
| `IpmPosition` | Posición lógica (Input / Output / Refused). |
| `IpmShutter` | Estado del obturador. |
| `IpmPositionStatus` | Estado de ocupación de una posición. |
| `IpmTransport` | Estado del sistema de transporte interno. |
| `IpmTransportMediaStatus` | Presencia de media en el transporte. |
| `IpmJammedShutterPos` | Posición del obturador atascado. |
| `IpmMixedModeCaps` | Capacidad de modo mixto (CIM + IPM). |
| `IpmMixedModeStatus` | Estado del modo mixto activo. |
| `IpmAntiFraudModule` | Estado del módulo antifraude. |
| `IpmGuidLightIndex` | Índice de luz de guía (MediaIn / MediaOut / Refused). |
| `IpmGuidLightValue` | Valor/color de luz de guía (flags). |
| `IpmInputType` | Tipo de entrada de medios: individual o en bundle (flags). |
| `IpmRetractLocation` | Ubicación de retracción disponible (flags). |
| `IpmResetControl` | Control de media durante reset (flags). |
| `IpmImageType` | Tipos de imagen soportados: TIF, WMF, BMP, JPG (flags). |
| `IpmImageColorFormat` | Formato de color de imagen: binario, escala de grises, completo (flags). |
| `IpmScanColor` | Color de escaneado: rojo, azul, verde, amarillo, blanco (flags). |
| `IpmCodelineFormat` | Formato de codeline: CMC7, E13B, OCR, OCR-A, OCR-B (flags). |
| `IpmDataSource` | Fuentes de datos: imagen frontal, imagen trasera, codeline (flags). |
| `IpmReturnedItemsProcessing` | Procesamiento de ítems devueltos: endorse, endorse+imagen (flags). |
| `IpmMediaBinType` | Tipo de papelera: MediaIn / Retract (flags). |
| `IpmMediaType` | Tipo de media: IPM / Compound (flags). |
| `IpmMediaBinStatus` | Estado de una papelera: Ok, Full, High, Inop, Missing, Unknown, Empty. |
| `IpmMediaInTransaction` | Estado de la transacción MediaIn en curso. |
| `IpmMediaLocation` | Ubicación de un ítem: Device / Bin / Customer / Unknown. |
| `IpmCustomerAccess` | Estado de acceso del cliente al medio. |
| `IpmImageStatus` | Estado de un dato de imagen capturada. |
| `IpmMagneticReadIndicator` | Indicador de lectura MICR del medio. |
| `IpmInsertOrientation` | Orientación de inserción del medio (flags). |
| `IpmMediaValidity` | Validez del medio: Ok, Suspect, Unknown, NoValidation. |
| `IpmSupplyReplen` | Consumibles reponibles: tóner e tinta (flags). |
| `IpmMediaRefusedReason` | Razón de rechazo de un medio. |
| `IpmRefuseLocation` | Ubicación del medio rechazado. |
| `IpmMediaBinFailure` | Tipo de fallo de una papelera. |
| `IpmMediaRejectedReason` | Razón de rechazo físico de un medio. |
| `IpmScannerSide` | Lado del escáner: Front / Back. |

Cada enum dispone de funciones helper: `getNumber(e)`, `getIpm<Enum>(v)` y en la mayoría de casos `toString(e)`.

## Cabecera de inclusión única

`XfsObjectsIpm.h` incluye todas las cabeceras de objeto. `TIpm.h` incluye todos los enumerados IPM.

```cpp
#include "XfsObjectsIpm.h"   // todos los objetos obj::ipm::*
#include "TIpm.h"            // todos los enums TIpm::*
```

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | `IXfsSerializable`, tipos de soporte del core XFS |
| `XfsProtos` | `../../externs/XfsProtos/inc` | Mensajes Protobuf generados (`proto::ipm::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| vcpkg (Protobuf) | `C:\Projects\vcpkg\installed\<plat>-windows-static\include` | Runtime de serialización Protobuf |
| STL | — | `<memory>`, `<vector>`, `<string>`, `<array>`, `<map>`, `<cstdint>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0


## Uso en proyectos consumidores

1. Añadir `../../externs/XfsObjectsIpm/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsObjectsIpm/lib/$(Platform)/$(Configuration)/XfsObjectsIpm.lib`.

```cpp
#include "XfsObjectsIpm.h"   // o incluir sólo las cabeceras necesarias

// Construir una solicitud de entrada de medios
obj::ipm::IpmMediaInRequest req;
req.fwCodelineFormat   = TIpm::IpmCodelineFormat::E13B;
req.usMaxMediaOnStacker = 50;
req.bApplicationRefuse  = false;

obj::ipm::IpmImageRequest imgReq;
imgReq.wImageSource      = TIpm::IpmScannerSide::Front;
imgReq.wImageType        = TIpm::IpmImageType::Jpg;
imgReq.wImageColorFormat = TIpm::IpmImageColorFormat::Full;
imgReq.wImageScanColor   = TIpm::IpmScanColor::Default;
req.vImages.push_back(imgReq);

// Serializar para transporte
std::vector<uint8_t> buffer;
req.serialize(buffer);
```

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport  ──┐
XfsProtos       ──┤──► XfsObjectsIpm ──┐
Log             ──┘                    │
                                       ▼
XfsCoreExe      ─────────────────► XfsBaseIpm ──► SpIpm.exe
XfsObjects      ─────────────────►
```
