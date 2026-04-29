# XfsCoreDll

Biblioteca estática C++ que actúa como **capa genérica de la DLL del Service Provider XFS** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsCoreDll` implementa la parte común de cualquier DLL de Service Provider (SP) XFS:

- **Implementación del SPI XFS** (`WFPOpen`, `WFPClose`, `WFPGetInfo`, `WFPExecute`, `WFPLock`, `WFPUnlock`, `WFPRegister`, `WFPDeregister`, `WFPCancelAsyncRequest`, `WFPSetTraceLevel`): los puntos de entrada exportados que el Manager XFS invoca.
- **Gestión de sesiones** (`ASession`): ciclo de vida, lista de sesiones activas y negociación de versión SPI/servicio.
- **Comunicación con el proceso servidor** (`APipeClient`): canal named-pipe para enviar peticiones y recibir respuestas del proceso EXE del SP.
- **Hilo lector de respuestas**: despacha asincrónamente las respuestas recibidas por el pipe hacia las sesiones/solicitudes correctas.
- **Trazas SPI** (`SpTrace`): logging de cada llamada SPI y su retorno mediante `WFMOutputTraceData`.
- **Despacho de comandos** (`SpDllClass`): serialización/deserialización de parámetros XFS a través de `IXfsFactory` e `IXfsSpiObject`, y enrutamiento de solicitudes `GetInfo`/`Execute`/eventos.

Es la base de la que heredan todas las DLL de SP concretas (CDM, SIU, PIN, etc.).

## Arquitectura

```
XFS Manager
     │  WFP* (SPI exports)
     ▼
 SpDll.cpp  ──────────────────────────────────────┐
     │                                             │
     ├── ASession          (gestión de sesiones)   │
     ├── APipeClient       (named pipe ↔ SP EXE)   │
     ├── SpDllClass        (despacho GetInfo/Exec)  │
     │       └── IXfsDllDevice                     │
     │               └── IXfsFactory               │
     │                       └── IXfsSpiObject      │
     └── SpTrace           (trazas SPI)             │
                                                    │
               NamedPipe ──────────────► SP EXE    ◄┘
```

`SpDll.cpp` implementa los puntos de entrada SPI de forma `final` para todo SP XFS. La lógica específica de cada clase de dispositivo la aporta `IXfsDllDevice` (instanciada por el SP concreto en `DllMain` a través de `InitalizeDllCommon`).

## Interfaces publicadas

### `SpDll.h`

Puntos de entrada SPI exportados y tipos de soporte.

| Símbolo | Descripción |
|---|---|
| `WFPCancelAsyncRequest` | Cancela una solicitud asíncrona pendiente. |
| `WFPClose` | Cierra una sesión de servicio. |
| `WFPDeregister` | Cancela el registro de eventos de una ventana. |
| `WFPExecute` | Envía un comando execute al SP. |
| `WFPGetInfo` | Consulta información al SP. |
| `WFPLock` | Bloquea el acceso exclusivo al dispositivo. |
| `WFPOpen` | Abre una sesión de servicio, negocia versiones SPI y servicio. |
| `WFPRegister` | Registra una ventana para recibir eventos XFS. |
| `WFPSetTraceLevel` | Establece el nivel de traza. |
| `WFPUnlock` | Libera el bloqueo exclusivo del dispositivo. |
| `InitalizeDllCommon` | Punto de entrada de inicialización; los SP concretos lo llaman en `DllMain` inyectando su `IXfsDllDevice`. |
| `WFSRESULT` | Estructura resultado XFS (RequestID, hService, timestamp, hResult, command/event, buffer). |
| `WFSVERSION` | Estructura de versión XFS (wVersion, wLowVersion, wHighVersion, descripción, estado). |

Funciones de soporte XFS también declaradas:

```cpp
HRESULT WINAPI WFMAllocateBuffer(ULONG ulSize, ULONG ulFlags, LPVOID* lppvData);
HRESULT WINAPI WFMAllocateMore(ULONG ulSize, LPVOID lpvOriginal, LPVOID* lppvData);
HRESULT WINAPI WFMReleaseDLL(HPROVIDER hProvider);
HRESULT WINAPI WFMOutputTraceData(LPCSTR lpszData);
```

### `XfsFactory.h`

| Interfaz / Clase | Descripción |
|---|---|
| `IXfsFactory` | Factoría abstracta que crea objetos `IXfsSpiObject` para serializar/deserializar los parámetros XFS de cada comando, query y evento según versión negociada. |
| `IXfsDllDevice` | Contrato que cada SP concreto debe implementar: expone `IXfsFactory`, el nombre del SP y los filtros de comandos/categorías soportados. |

#### `IXfsFactory` — métodos principales

| Método | Descripción |
|---|---|
| `newExecuteInput(dwCommand, wVersion)` | Crea el objeto de entrada para un comando execute. |
| `newExecuteOutput(dwCommand, wVersion)` | Crea el objeto de salida para un comando execute. |
| `newGetInfoInput(dwCategory, wVersion)` | Crea el objeto de entrada para una query getInfo. |
| `newGetInfoOutput(dwCategory, wVersion)` | Crea el objeto de salida para una query getInfo. |
| `newEventOutput(dwEventID, wVersion)` | Crea el objeto de salida para un evento. |
| `versionRange()` | Rango de versiones de servicio soportadas (`DWORD` compacto). |
| `getCommandType(dwRequest_type, dwCommand)` | Determina el tipo interno del comando (`GETINFO`, `EXECUTE`, `GETDYNINFO`, `IMMEDIATEEXECUTE`). |
| `getServiceClass()` | Devuelve la clase de servicio XFS (`WFS_SERVICE_CLASS_XXX`). |

### `XfsSpiObject.h`

| Interfaz | Descripción |
|---|---|
| `IXfsSPI` | Contrato de serialización XFS: `initialize` (desde estructura WFS), `createStruct` (hacia estructura WFS), `createStructFromObject` (desde objeto base). Incluye utilidades `toUtf8`, `buildMultiString` y `parseMultiString`. |
| `IXfsSpiObject` | Combina `IXfsSerializable` e `IXfsSPI`; clase base de todos los objetos de parámetro XFS. |

### `SpDllClass.h`

| Clase | Descripción |
|---|---|
| `SpDllClass` | Orquesta el ciclo de vida de una petición: delega en `IXfsFactory` para crear objetos de parámetro, gestiona el despacho `ManageGetInfoRequest`/`ManageExecuteRequest` y el retorno `ManageGetInfoResponse`/`ManageExecuteResponse`/`ManageEventResponse`. |

Constante: `SPI_VERSION_RANGE 0x0003FF03`.

### `ASession.h`

| Clase | Descripción |
|---|---|
| `ASession` | Representa una sesión abierta. Mantiene `HSERVICE`, `HAPP`, `HPROVIDER`, versión negociada, lista de peticiones en curso y estado de traza. Gestiona la lista global de sesiones activas. |

### `APipe.h`

| Clase | Descripción |
|---|---|
| `APipeClient` | Cliente de named pipe asíncrono (overlapped I/O). Abre/cierra la conexión con el proceso EXE del SP y proporciona `send`/`receive` de mensajes `MSG_PIPE`. Buffer de 2 MB, timeout de 3 s. |

### `SpTrace.h`

Funciones de traza (no públicas, uso interno):

```cpp
VOID traceOpen(...)         / traceOpenReturn(...)
VOID traceClose(...)        / traceCloseReturn(...)
VOID traceCancelAsyncRequest(...) / traceCancelAsyncRequestReturn(...)
VOID traceRegister(...)     / traceRegisterReturn(...)
VOID traceDeregister(...)   / traceDeregisterReturn(...)
VOID traceGetInfo(...)      / traceGetInfoReturn(...)
VOID traceExecute(...)      / traceExecuteReturn(...)
VOID traceLock(...)         / traceLockReturn(...)
VOID traceUnlock(...)       / traceUnlockReturn(...)
VOID traceSetTraceLevel(...)/ traceSetTraceLevelReturn(...)
VOID traceResult(...)
VOID traceGetInfoResult(...)
VOID traceExecuteResult(...)
VOID traceEvent(...)
```

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IPipeProtocol`, `ARequest`, `ALink`, `AList`, `AEvent`, `AException`, `AllocMem`, `RegConfig`, `IXfsSerializable` |
| `Log` | `../../externs/Log/inc` | `BswLog` (logging estructurado) |
| Win32 API | — | Named pipes, threads, mensajes de ventana |
| STL | — | `<mutex>`, `<vector>`, `<string>`, `<map>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Manejo de excepciones:** Asíncrono (`/EHa`)

### Post-build automático

Tras cada compilación, los artefactos se copian automáticamente a:

```
../../externs/XfsCoreDll/inc/                          ← cabeceras (.h)
../../externs/XfsCoreDll/lib/<Plat>/<Config>/          ← XfsCoreDll.lib
```

Esto permite que las DLL de SP concretas consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsCoreDll/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsCoreDll/lib/$(Platform)/$(Configuration)/XfsCoreDll.lib`.
3. En `DllMain`, crear la implementación concreta de `IXfsDllDevice` y llamar a `InitalizeDllCommon`:

```cpp
BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved)
{
    if (ul_reason_for_call == DLL_PROCESS_ATTACH)
    {
        return InitalizeDllCommon(std::make_unique<MiSpDevice>());
    }
    return TRUE;
}
```

4. Implementar `IXfsDllDevice` y `IXfsFactory` con los objetos `IXfsSpiObject` específicos para cada comando/categoría/evento del dispositivo.

```cpp
class MiSpDevice : public IXfsDllDevice
{
public:
    bool isCategorySupported(uint32_t dwCategory) override;
    bool isCommandSupported(uint32_t dwCommand)   override;
    const LPSTR getSpName(VOID)                   override;
    IXfsFactory* getFactory()                     override;
};
```

## Posición en el árbol de compilación (SP DLL)

```
XfsCoreSupport ──┐
Log             ──┤──► XfsCoreDll ──► SpCdm.dll
                          │           SpPin.dll
                          │           SpSiu.dll
                          └─────────► (cualquier SP DLL)
```
