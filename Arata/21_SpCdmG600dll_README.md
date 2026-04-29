# SpCdmG600

DLL C++ que actúa como **Service Provider (SP) CDM para el dispositivo G600** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`SpCdmG600` implementa la interfaz SPI de XFS para dispensadores de efectivo (CDM) específica del hardware G600:

- **Exposición de los puntos de entrada SPI** (`WFP*`) que el manager XFS invoca para gestionar el servicio CDM.
- **Delegación de la construcción de objetos SPI** a `CXfsFactoryCdm`, que serializa/deserializa los parámetros XFS CDM.
- **Declaración de soporte de comandos**: todos los comandos CDM estándar excepto `SynchronizeCommand` y `SetClassificationList`.
- **Identificación del SP** con el nombre lógico `"SpG600"`.

Es el punto de entrada del framework XFS hacia el hardware G600 para la clase de dispositivo CDM.

## Arquitectura

```
XFS Manager (msxfs)
       │  WFP* (SPI)
       ▼
SpCdmG600.dll
       │
       └── SpCdmDllG600  ──implements──►  IXfsDllDevice  (XfsCoreDll)
                │
                ├── InitalizeDllCommon()   →  XfsCoreDll (gestión SPI común)
                └── CXfsFactoryCdm        →  IXfsFactory (XfsFactoryCdm)
```

`SpCdmDllG600` implementa `IXfsDllDevice` proporcionando:
- El nombre lógico del SP (`"SpG600"`).
- La instancia de `CXfsFactoryCdm` para construir los objetos SPI CDM.
- El filtro de comandos y categorías soportados.

El arranque en `DLL_PROCESS_ATTACH` cede el control a `InitalizeDllCommon`, que registra el SP y conecta los puntos de entrada `WFP*` con el core genérico.

## Puntos de entrada exportados

Definidos en `SpCdmG600.def` y declarados en `SpDll.h`:

| Función | Descripción |
|---|---|
| `WFPOpen` | Abre una sesión de servicio CDM. |
| `WFPClose` | Cierra la sesión de servicio. |
| `WFPUnloadService` | Descarga el SP del manager. |
| `WFPCancelAsyncRequest` | Cancela una petición asíncrona pendiente. |
| `WFPRegister` | Registra una ventana para recibir eventos. |
| `WFPDeregister` | Cancela el registro de eventos de una ventana. |
| `WFPGetInfo` | Ejecuta un comando de tipo `getInfo` CDM. |
| `WFPExecute` | Ejecuta un comando de tipo `execute` CDM. |
| `WFPLock` | Bloquea el dispositivo para uso exclusivo. |
| `WFPUnlock` | Libera el bloqueo exclusivo. |
| `WFPSetTraceLevel` | Configura el nivel de traza del SP. |

## Soporte de comandos

### Categorías `getInfo`

Todas las categorías CDM (`isCategorySupported` devuelve `true` para cualquier valor de `dwCategory`).

### Comandos `execute`

| Comando | Soportado |
|---|---|
| `SynchronizeCommand` | ✗ |
| `SetClassificationList` | ✗ |
| Resto de comandos CDM | ✓ |

## Dependencias

| Dependencia | Ruta include | Biblioteca | Propósito |
|---|---|---|---|
| `XfsCoreDll` | `../../externs/xfsCoreDll/inc` | `XfsCoreDll.lib` | `IXfsDllDevice`, `IXfsFactory`, `InitalizeDllCommon`, puntos de entrada SPI |
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `XfsCoreSupport.lib` | Tipos y utilidades de soporte del core XFS |
| `XfsFactoryCdm` | `../../externs/xfsFactoryCdm/inc` | `XfsFactoryCdm.lib` | `CXfsFactoryCdm`: construcción de objetos SPI CDM |
| `XfsObjectsCdm` | `../../externs/xfsObjectsCdm/inc` | `XfsObjectsCdm.lib` | Tipos CDM (`obj::cdm::*`, `TCdm::*`) |
| `XfsObjects` | `../../externs/xfsObjects/inc` | `XfsObjects.lib` | Tipos XFS genéricos (`TXfs::*`) |
| `XfsSpiObjects` | — | `XfsSpiObjects.lib` | Objetos SPI genéricos |
| `XfsSpiObjectsCdm320` | — | `XfsSpiObjectsCdm320.lib` | Objetos SPI CDM versión 3.20 |
| `XfsSpiObjectsCdm330` | — | `XfsSpiObjectsCdm330.lib` | Objetos SPI CDM versión 3.30 |
| `XfsProtos` | — | `XfsProtos.lib` | Serialización protobuf de objetos XFS |
| `Log` | `../../externs/Log/inc` | `BswLog.lib` | Logging y trazas |
| `msxfs` | — | `msxfs.lib` | SDK XFS de Microsoft |
| Protobuf / Abseil | `C:\Projects\vcpkg\...` | `libprotobuf[d].lib`, `absl_*.lib` | Serialización protobuf (vcpkg) |

## Compilación

- **Tipo de salida:** Biblioteca dinámica (`.dll`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Módulo de definición:** `SpCdmG600.def`

### Directorio de salida

| Plataforma | Ruta |
|---|---|
| Win32 | `$(SolutionDir)bin\$(Configuration)\` |
| x64 | `$(SolutionDir)bin\$(Configuration)\x64\` |

## Posición en el árbol de compilación

```
XfsCoreSupport      ──┐
XfsCoreDll          ──┤
XfsFactoryCdm       ──┤
XfsSpiObjects       ──┤──► SpCdmG600.dll  ←── XFS Manager (msxfs)
XfsSpiObjectsCdm320 ──┤
XfsSpiObjectsCdm330 ──┤
XfsObjectsCdm       ──┤
XfsObjects          ──┤
XfsProtos           ──┤
Log                 ──┘
```
