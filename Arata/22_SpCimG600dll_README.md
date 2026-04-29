# SpCimG600

DLL de Service Provider C++ que implementa la **interfaz XFS SPI para módulos de aceptación de efectivo (CIM)** del hardware G600 dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`SpCimG600` es el punto de entrada XFS para el hardware G600 en su rol de Cash-In Module (CIM). Actúa como **adaptador entre el Manager XFS y la lógica de dispositivo CIM**:

- **Implementa el SPI XFS** exportando las funciones estándar `WFP*` que el Manager XFS invoca.
- **Declara las capacidades del dispositivo** indicando qué categorías de `getInfo` y qué comandos `execute` están soportados por el hardware G600.
- **Delega la serialización/deserialización** de estructuras XFS en `CXfsFactoryCim`, que gestiona el ciclo de vida de los objetos SPI para versiones CIM 3.20 y 3.30.
- **Inicializa el framework común DLL** (`InitalizeDllCommon`) con la instancia específica del dispositivo.

## Arquitectura

```
Manager XFS (msxfs)
       │  WFP* calls
       ▼
  SpCimG600.dll  (SpCimDllG600 : IXfsDllDevice)
       │
       ├── CXfsFactoryCim  →  IXfsFactory
       │       ├── CXfsFactoryCim320   (versión 3.20)
       │       └── CXfsFactoryCim330   (versión 3.30)
       │
       └── XfsCoreDll  (SpDllClass / InitalizeDllCommon)
               ├── ManageGetInfoRequest / ManageGetInfoResponse
               └── ManageExecuteRequest / ManageExecuteResponse
```

`SpCimDllG600` implementa `IXfsDllDevice` e informa al framework común qué categorías y comandos soporta el G600. El framework (`XfsCoreDll`) gestiona el despacho asíncrono, versiones negociadas y la comunicación con el Manager.

## Clase principal

### `SpCimDllG600` — `SpCimG600.cpp`

| Método | Descripción |
|---|---|
| `isCategorySupported` | Valida una categoría `getInfo` CIM. Todas las categorías son soportadas. |
| `isCommandSupported` | Valida un comando `execute` CIM. Retorna `false` para los comandos no soportados por este hardware. |
| `getSpName` | Retorna el nombre lógico del SP: `"SpG600"`. |
| `getFactory` | Retorna la instancia de `CXfsFactoryCim` para serialización de objetos XFS. |

## Comandos no soportados

Los siguientes comandos `execute` no están implementados en esta versión del SP y retornan `false` en `isCommandSupported`:

| Comando | Razón |
|---|---|
| `SetGuidanceLight` | No soportado por el hardware G600 CIM. |
| `SynchronizeCommand` | No soportado en esta versión del SP. |
| `SetClassificationList` | No soportado en esta versión del SP. |

## Exportaciones SPI

El módulo exporta la superficie SPI XFS completa mediante `SpCimG600.def`:

| Función exportada | Descripción |
|---|---|
| `WFPOpen` | Abre una sesión de servicio con el SP. |
| `WFPClose` | Cierra una sesión de servicio. |
| `WFPUnloadService` | Descarga el servicio. |
| `WFPCancelAsyncRequest` | Cancela una solicitud asíncrona pendiente. |
| `WFPRegister` | Registra una ventana para recibir eventos. |
| `WFPDeregister` | Anula el registro de una ventana. |
| `WFPGetInfo` | Solicita información de categoría CIM. |
| `WFPExecute` | Ejecuta un comando CIM. |
| `WFPLock` | Bloquea el servicio para uso exclusivo. |
| `WFPUnlock` | Libera el bloqueo del servicio. |
| `WFPSetTraceLevel` | Establece el nivel de traza del SP. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreDll` | `../../externs/xfsCoreDll/inc` | Framework DLL común: `IXfsDllDevice`, `IXfsFactory`, `SpDllClass`, implementación SPI |
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `XfsFactoryCim` | `../../externs/xfsFactoryCim/inc` | Fábrica de objetos CIM: `CXfsFactoryCim` (versiones 3.20 y 3.30) |
| `XfsObjectsCim` | `../../externs/xfsObjectsCim/inc` | Tipos CIM (`obj::cim::*`, `TCim::*`) |
| `XfsObjects` | `../../externs/xfsObjects/inc` | Tipos XFS genéricos (`obj::xfs::*`, `TXfs::*`) |
| `XfsSpiObjects` | `../../externs/XfsSpiObjects` | Objetos SPI genéricos |
| `XfsSpiObjectsCim320` | `../../externs/XfsSpiObjectsCim320` | Objetos SPI CIM versión 3.20 |
| `XfsSpiObjectsCim330` | `../../externs/XfsSpiObjectsCim330` | Objetos SPI CIM versión 3.30 |
| `XfsProtos` | `../../externs/XfsProtos` | Serialización protobuf para comunicación con el Manager |
| `neoManager` | `../../externs/neoManager` | Integración con el gestor neo |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `msxfs.lib` | Windows SDK | Librería XFS del sistema (Manager side) |
| Protobuf + absl | vcpkg | `libprotobuf`, `libprotocd`, `utf8_range`, `absl_*` |

## Compilación

- **Tipo de salida:** Biblioteca dinámica (`.dll`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Versión de producto:** 12.0.0.0 (Fujitsu — BENNU.Sp.SpcimG600)

### Directorio de salida

| Plataforma | Configuración | Ruta de salida |
|---|---|---|
| Win32 | Debug / Release | `$(SolutionDir)bin\$(Configuration)\` |
| x64 | Debug / Release | `$(SolutionDir)bin\$(Configuration)\x64\` |

## Posición en el árbol de compilación (DLL)

```
XfsCoreSupport    ──┐
XfsCoreDll        ──┤
XfsFactoryCim     ──┤
XfsSpiObjects     ──┤──► SpCimG600.dll  ◄── Manager XFS
XfsObjectsCim     ──┤
XfsObjects        ──┤
XfsProtos         ──┤
Log               ──┘
```
