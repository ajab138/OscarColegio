# SpIpmG600

DLL C++ que actúa como **Service Provider XFS para módulos de procesamiento de imágenes (IPM)** del hardware G600, dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`SpIpmG600` es el punto de entrada del Service Provider IPM para el hardware G600:

- **Implementa la SPI XFS** exportando las funciones estándar `WFP*` que el XFS Manager invoca para gestionar sesiones, comandos e información del dispositivo.
- **Delega la construcción de objetos** de entrada/salida en `CXfsFactoryIpm`, que cubre los protocolos IPM 3.20 y 3.30.
- **Identifica el SP** con el nombre lógico `"SpG600"` y la clase de servicio `TIpm::SERVICE_CLASS_IPM`.

## Arquitectura

```
XFS Manager
     │  WFP* (SPI)
     ▼
XfsCoreDll  ─────────────────────────────────────────┐
     │  InitalizeDllCommon(SpIpmDllG600)              │
     ▼                                                │
IXfsDllDevice                                         │
     │                                                │
     ▼                                                │
SpIpmDllG600 ──── CXfsFactoryIpm                      │
                  ├── XfsFactoryIpm320  (IPM 3.20)    │
                  └── XfsFactoryIpm330  (IPM 3.30)    │
                                                      │
     XfsSpiObjects / XfsSpiObjectsIpm320/330 ◄────────┘
```

`XfsCoreDll` proporciona la implementación común de las funciones SPI (`WFPOpen`, `WFPExecute`, etc.) y llama a `SpIpmDllG600` para los aspectos específicos del hardware G600.

## Clase principal: `SpIpmDllG600`

Implementa `IXfsDllDevice` y es instanciada una única vez en `DLL_PROCESS_ATTACH`:

| Método | Descripción |
|---|---|
| `isCategorySupported(dwCategory)` | Valida si la categoría `getInfo` es soportada (via `TIpm::getIpmInfoCmd`). |
| `isCommandSupported(dwCommand)` | Valida si el comando `execute` es soportado (via `TIpm::getIpmExecCmd`). |
| `getSpName()` | Devuelve `"SpG600"`, nombre lógico del Service Provider. |
| `getFactory()` | Proporciona la instancia de `CXfsFactoryIpm` para la serialización de objetos IPM. |

## Funciones SPI exportadas

Definidas en `SpIpmG600.def` e implementadas por `XfsCoreDll`:

| Función | Descripción |
|---|---|
| `WFPOpen` | Abre una sesión con el Service Provider. |
| `WFPClose` | Cierra una sesión. |
| `WFPUnloadService` | Descarga el Service Provider. |
| `WFPCancelAsyncRequest` | Cancela una petición asíncrona pendiente. |
| `WFPRegister` | Registra una ventana para recibir eventos. |
| `WFPDeregister` | Elimina el registro de eventos de una ventana. |
| `WFPGetInfo` | Solicita información del dispositivo IPM. |
| `WFPExecute` | Ejecuta un comando IPM. |
| `WFPLock` | Bloquea el dispositivo para uso exclusivo. |
| `WFPUnlock` | Libera el bloqueo del dispositivo. |
| `WFPSetTraceLevel` | Configura el nivel de trazas del SP. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreDll` | `../../externs/xfsCoreDll/inc` | `IXfsDllDevice`, `IXfsFactory`, `SpDll.h`, implementación SPI común |
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | Tipos y utilidades de soporte del core XFS |
| `XfsFactoryIpm` | `../../externs/xfsFactoryIpm/inc` | `CXfsFactoryIpm`: fábrica de objetos IPM 3.20/3.30 |
| `XfsObjectsIpm` | `../../externs/xfsObjectsIpm/inc` | Tipos IPM (`obj::ipm::*`, `TIpm::*`) |
| `XfsObjects` | `../../externs/xfsObjects/inc` | Tipos XFS genéricos (`obj::xfs::*`, `TXfs::XFSError`) |
| `XfsSpiObjects` | `../../externs/XfsSpiObjects` | Objetos SPI genéricos (serialización XFS) |
| `XfsSpiObjectsIpm320` | `../../externs/XfsSpiObjectsIpm320` | Serialización SPI IPM protocolo 3.20 |
| `XfsSpiObjectsIpm330` | `../../externs/XfsSpiObjectsIpm330` | Serialización SPI IPM protocolo 3.30 |
| `XfsProtos` | `../../externs/XfsProtos` | Definiciones Protocol Buffers XFS |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `msxfs.lib` | Sistema | XFS Manager API estándar |
| Protobuf / absl | vcpkg | Serialización Protocol Buffers (libprotobuf, absl_*) |

## Compilación

- **Tipo de salida:** Biblioteca dinámica (`.dll`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Excepción handling:** Asíncrono (`/EHa`)

### Salida de compilación

```
$(SolutionDir)bin\$(Configuration)\             ← Win32
$(SolutionDir)bin\$(Configuration)\$(Platform)\ ← x64
```

## Posición en el árbol de compilación

```
XfsCoreSupport      ──┐
XfsCoreDll          ──┤
XfsFactoryIpm       ──┤
XfsObjectsIpm       ──┤──► SpIpmG600.dll  ──► XFS Manager (msxfs)
XfsObjects          ──┤
XfsSpiObjects*      ──┤
XfsProtos           ──┤
Log                 ──┘
```
