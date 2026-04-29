# XfsCoreExe

Biblioteca estática C++ que actúa como **capa genérica del proceso EXE del Service Provider XFS** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsCoreExe` implementa la parte común del proceso servidor (EXE) de cualquier Service Provider XFS:

- **Ciclo de vida del servicio Windows** (`Service`): instalación, eliminación, arranque, parada y modo depuración del servicio NT.
- **Punto de entrada del ejecutable** (`SpMain`): función `CommonExeMain` que arranca el hilo principal y registra el servicio; funciones auxiliares para obtener el nombre de servicio, pipe y SP.
- **Hilo principal** (`MainThreadClass`): bucle de eventos que procesa peticiones de la DLL, gestiona colas diferidas y no diferidas, timeouts, lock/unlock y el ciclo de cierre.
- **Servidor de named pipe** (`APipeManager` / `APipeInstance`): acepta conexiones de la DLL del SP y enruta los mensajes `MSG_PIPE` entrantes al hilo principal; envía las respuestas a los clientes.
- **Puente con el Device Manager** (`SpDeviceManager`): capa de funciones `DM_*` que comunica el hilo principal con la lógica de dispositivo ejecutada en su propio hilo.
- **Orquestador XFS** (`CSpXfs`): singleton que delega las operaciones `open`, `close`, `cancel`, `getinfo` y `execute` a los dispositivos registrados (`IXfsCommonDevice` e `IXfsClassDevice`).
- **Gestión de sesiones** (`ASession`, `CSession`): ciclo de vida de sesiones abiertas desde la DLL.
- **Mensajería interna** (`AMessage`): encapsula cualquier petición o respuesta que fluye por el sistema (REQ_OPEN, REQ_EXECUTE, RSP_EXECUTE, RSP_EXECUTE_EVENT, etc.).
- **Logging** (`SpExeLog`): logging estructurado mediante `BswLog`.

Es la base de la que heredan todos los procesos EXE de SP concretos (CDM, SIU, PIN, etc.).

## Arquitectura

```
SP DLL (XfsCoreDll)
     │  Named Pipe (MSG_PIPE)
     ▼
APipeManager / APipeInstance   (servidor de pipe, overlapped I/O)
     │  AMessage
     ▼
MainThreadClass                (bucle de eventos principal)
     ├── ANonDeferredReqQueue  (peticiones inmediatas: open, close, lock…)
     ├── ADeferredReqQueue     (peticiones diferidas: execute, getinfo)
     ├── ATimer                (gestión de timeouts)
     ├── ASession              (lista de sesiones activas)
     └── SpDeviceManager (DM_*)
              │
              ▼
         CSpXfs  ──────────────────────────────────┐
              ├── IXfsCommonDevice                  │
              │       open / close / cancel         │
              │       polling / sessionNotify        │
              └── IXfsClassDevice[]                 │
                      processGetInfo                │
                      processExecute                │
                      processExecuteImmediate        │
                      postExecute                   │
                      notifyXxxEvent ───────────────┘
                                         │
                              AMainThread notifyDM*()
```

`SpMain.cpp` / `Service.cpp` registran el servicio NT y llaman a `MainThreadClass::main()`. La lógica específica de cada clase de dispositivo la aportan `IXfsCommonDevice` e `IXfsClassDevice` (instanciados por el SP concreto antes de llamar a `CommonExeMain`).

## Interfaces publicadas

### `SpMain.h`

Punto de entrada principal y funciones de identificación del SP.

| Símbolo | Descripción |
|---|---|
| `CommonExeMain(argc, argv)` | Punto de entrada del EXE; los SP concretos lo llaman desde `main()`. |
| `getServiceName(buf, size)` | Rellena `buf` con el nombre del servicio NT registrado. |
| `getDisplayName()` | Nombre de presentación del servicio. |
| `getAppName()` | Nombre de la aplicación. |
| `getPipeName(buf, size)` | Rellena `buf` con el nombre del named pipe del SP. |
| `getSpName()` | Nombre corto del SP (p. ej. `"CDM"`). |

### `SpXfs.h`

| Interfaz / Clase | Descripción |
|---|---|
| `ISpXfs` | Contrato del orquestador XFS: registro de dispositivos, ciclo de vida (open/close/cancel/polling), operaciones getinfo/execute y notificación de sesiones. |
| `CSpXfs` | Implementación singleton de `ISpXfs`; enruta cada operación al `IXfsCommonDevice` o al `IXfsClassDevice` correspondiente según `dwCategoryOrCommand`. |

#### `ISpXfs` — métodos principales

| Método | Descripción |
|---|---|
| `registerCommonDevice(device)` | Registra el dispositivo común (open/close/cancel/polling). |
| `registerClassDevice(device)` | Registra un dispositivo de clase (getinfo/execute). |
| `open(lpszLogicalName, wSrvcVersion)` | Abre el dispositivo con el nombre lógico y versión de servicio negociados. |
| `close()` | Cierra el dispositivo. |
| `cancel()` | Cancela el comando en curso. |
| `polling()` | Ejecuta un ciclo de polling del dispositivo. |
| `notifyClientSessionOpened(hService)` | Informa al dispositivo de que se ha abierto una sesión cliente. |
| `notifyClientSessionClosed(hService)` | Informa al dispositivo de que se ha cerrado una sesión cliente. |
| `getinfo(dwCategory, bufIn, bufOut)` | Consulta información síncrona al dispositivo. |
| `executeImmediate(hService, dwCategory, bufIn, bufOut)` | Ejecuta un comando inmediato (no diferido). |
| `execute(pRequest, bufOut)` | Ejecuta un comando diferido. |
| `getinfoDeferred(pRequest, bufOut)` | Consulta información diferida. |
| `postExecute(pRequest, hr)` | Notifica la finalización de un execute al dispositivo. |

### `IXfsClassDevice.h`

| Interfaz | Descripción |
|---|---|
| `IXfsClassDevice` | Estrategia para operaciones específicas de clase de dispositivo (getinfo, execute). Base de todos los dispositivos de clase concretos. |

#### Métodos principales

| Método | Descripción |
|---|---|
| `getDeviceType()` | Tipo de dispositivo XFS (`WFS_SERVICE_CLASS_XXX`). |
| `processGetInfo(dwCategory, bufIn, bufOut)` | Procesa una consulta GetInfo. |
| `processExecute(pRequest, bufOut)` | Procesa un comando Execute diferido. |
| `processExecuteImmediate(hService, dwCategory, bufIn, bufOut)` | Procesa un Execute inmediato (impl. por defecto: no-op). |
| `proccessGetinfoDeferred(pRequest, bufOut)` | Procesa un GetInfo diferido (impl. por defecto: no-op). |
| `postExecute(pRequest, hr)` | Acciones post-execute (impl. por defecto: no-op). |
| `notifyExecuteEvent(dwEventId, buf)` | Lanza un evento de tipo Execute. |
| `notifyUserEvent(dwEventId, buf)` | Lanza un evento de tipo User. |
| `notifyServiceEvent(dwEventId, buf)` | Lanza un evento de tipo Service. |
| `notifySystemEventToMyClass(dwEventId, buf)` | Lanza un evento de tipo System filtrado por clase. |
| `isExecuting()` | Indica si hay un comando execute en curso. |
| `getSpName()` | Nombre del SP. |

### `IXfsCommonDevice.h`

| Interfaz | Descripción |
|---|---|
| `IXfsCommonDevice` | Contrato para operaciones comunes: open, close, cancel, polling, notificación de sesiones. |

#### Métodos principales

| Método | Descripción |
|---|---|
| `open(logicalName, wSrvcVersion)` | Abre el dispositivo físico. |
| `close()` | Cierra el dispositivo físico. |
| `cancel()` | Cancela la operación en curso. |
| `polling()` | Ciclo de polling periódico. |
| `notifyClientSessionOpened(hService)` | Nueva sesión cliente abierta. |
| `notifyClientSessionClosed(hService)` | Sesión cliente cerrada. |
| `notifySystemEvent(dwEventId, buf)` | Lanza un evento de tipo System global. |
| `setTimePolling(timePolling)` | Configura el intervalo de polling (ms). |

### `SpDeviceManager.h`

Capa de funciones C (`DM_*`) que el hilo principal usa para comunicarse con el hilo del Device Manager.

| Función | Descripción |
|---|---|
| `DM_main()` | Hilo principal del Device Manager. |
| `DM_openService(logicalName, wSPIVersion, wSrvcVersion)` | Abre el servicio en el DM. |
| `DM_getInformation(hService, fGetInfo, dwCategory, bufIn, bufOut)` | Consulta información síncrona. |
| `DM_startExecution(hService, fGetInfo, dwCommand, bufIn)` | Inicia un execute/getinfo asíncrono. |
| `DM_startCancel()` | Inicia la cancelación del comando en curso. |
| `DM_closeService()` | Cierra el servicio en el DM. |
| `DM_startTermination()` | Inicia la secuencia de terminación. |
| `DM_updateEventClassMask(dwMask)` | Actualiza la máscara global de clases de eventos. |
| `DM_modifyPollInterval(ulPollInterval)` | Modifica el intervalo de polling. |
| `DM_isExecuting()` | Informa si el DM está ejecutando un comando. |
| `DM_getCurrentRequest()` | Devuelve la petición en ejecución actual. |
| `DM_getSession(hService)` | Obtiene la sesión asociada a un `HSERVICE`. |
| `DM_NotifyClientSessionOpened(hService)` | Notifica apertura de sesión. |
| `DM_NotifyClientSessionClosed(hService)` | Notifica cierre de sesión. |

### `AMainThread.h`

Notificaciones que el Device Manager (o el dispositivo) usa para comunicar resultados al hilo principal.

| Función | Descripción |
|---|---|
| `notifyDMCancelCompleted()` | La cancelación ha terminado. |
| `notifyDMCompletionEvent(fGetInfo, hResult, bufferData)` | El comando Execute o GetInfo ha finalizado. |
| `notifyDMExecuteEvent(dwEventId, bufferData)` | Evento de tipo Execute generado por el dispositivo. |
| `notifyDMEvent(dwClassIdMask, dwEventId, bufferData, dwClass)` | Evento de tipo System, Service o User. |
| `notifyDMInitialized()` | El Device Manager ha terminado su inicialización. |

| Clase | Descripción |
|---|---|
| `MainThreadClass` | Bucle de eventos principal; procesa peticiones de las colas diferida y no diferida, gestiona timers y eventos externos. |

### `APipeManager.h`

| Clase | Descripción |
|---|---|
| `APipeManager` | Servidor de named pipe; gestiona las instancias de pipe y un hilo dedicado. `start(dispatch)` / `stop()` / `sendResponse(msg)`. |
| `APipeInstance` | Instancia individual de pipe (overlapped I/O, 2 MB de buffer, timeout 3 s). Gestiona la conexión, lectura del header y payload, y escritura de respuestas. |

### `AMessage.h`

| Clase | Descripción |
|---|---|
| `AMessage` | Mensaje interno que encapsula cualquier petición o respuesta. Contiene tipo (`MSG_TYPE`), handles de sesión/aplicación/ventana, código de comando/categoría/evento, buffer de datos y referencia al timer. |

Tipos de mensaje relevantes: `REQ_OPEN`, `REQ_CLOSE`, `REQ_EXECUTE`, `REQ_GETINFO`, `REQ_GETDYNINFO`, `REQ_IMMEDIATEEXECUTE`, `RSP_EXECUTE`, `RSP_EXECUTE_EVENT`, `RSP_SERVICE_EVENT`, `RSP_SYSTEM_EVENT`, `RSP_USER_EVENT`, `BROKENPIPE`.

### `ASession.h` / `CSession.h`

| Clase | Descripción |
|---|---|
| `ASession` | Sesión activa en el EXE; mantiene `HSERVICE`, handles de aplicación/proveedor, nombre lógico, nombre de workstation y la `APipeInstance` asociada. Gestiona la lista global de sesiones. |
| `CSession` | Objeto ligero de sesión para el Device Manager; almacena `HSERVICE`, nombre lógico y nombre de workstation. |

### `ExecutionRequest.h`

| Clase | Descripción |
|---|---|
| `CExecutionRequest` | Encapsula un comando en ejecución: `HSERVICE`, código de comando, flag `fGetInfo` y buffer de datos de entrada. |

### `SpExeLog.h`

| Símbolo | Descripción |
|---|---|
| `logger` | Instancia global de `BswLog` para logging estructurado. |
| `logSPException(log, unit, desc, spe)` | Registra una excepción SP. |
| `logWinErr(log, unit, err)` | Registra un error del sistema Windows con módulo y línea. |
| `EXECOMMON` | Nombre de unidad para logging. |

### `Service.h`

| Función | Descripción |
|---|---|
| `ServiceStart(dwArgc, lpszArgv)` | Arranca la lógica del servicio. |
| `ServiceStop()` | Detiene el servicio. |
| `CmdInstallService(dwServiceType)` | Instala el servicio NT. |
| `CmdRemoveService()` | Elimina el servicio NT. |
| `CmdRegisterEventLog(dll, name)` | Registra el origen de eventos de log. |
| `CmdDebugService(argc, argv)` | Ejecuta como aplicación de consola para depuración. |
| `serviceEntry(argc, argv)` | Punto de despacho de argumentos de línea de comandos. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IPipeProtocol`, `ARequest`, `ALink`, `AList`, `AEvent`, `AEventHandler`, `AException`, `AllocMem`, `IXfsApi`, `IXfsSerializable` |
| `Log` | `../../externs/Log/inc` | `BswLog` (logging estructurado) |
| Win32 API | — | Named pipes (overlapped), servicios NT, threads, mensajes de ventana |
| STL | — | `<vector>`, `<string>`, `<memory>`, `<mutex>` |

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
../../externs/XfsCoreExe/inc/                          ← cabeceras (.h)
../../externs/XfsCoreExe/lib/<Plat>/<Config>/          ← XfsCoreExe.lib
```

Esto permite que los EXE de SP concretos consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsCoreExe/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsCoreExe/lib/$(Platform)/$(Configuration)/XfsCoreExe.lib`.
3. Implementar `IXfsCommonDevice` e `IXfsClassDevice` con la lógica específica del dispositivo.
4. Registrar las implementaciones y llamar a `CommonExeMain` desde `main`:

```cpp
#include "SpMain.h"
#include "SpXfs.h"

int main(int argc, char *argv[])
{
    CSpXfs::getInterface().registerCommonDevice(std::make_unique<MiCommonDevice>());
    CSpXfs::getInterface().registerClassDevice(std::make_unique<MiClassDevice>());
    CommonExeMain(argc, argv);
    return 0;
}
```

5. Implementar las interfaces del dispositivo:

```cpp
class MiCommonDevice : public IXfsCommonDevice
{
public:
    int32_t open(std::string logicalName, uint16_t wSrvcVersion) override;
    bool    close()   override;
    bool    cancel()  override;
    void    polling() override;
    void    notifyClientSessionOpened(uint16_t hService) override;
    void    notifyClientSessionClosed(uint16_t hService) override;
};

class MiClassDevice : public IXfsClassDevice
{
public:
    uint16_t getDeviceType() const override;
    int32_t processGetInfo(uint32_t dwCategory,
                           const std::vector<uint8_t>& bufIn,
                           std::vector<uint8_t>& bufOut) override;
    int32_t processExecute(CExecutionRequest* pRequest,
                           std::vector<uint8_t>& bufOut) override;
};
```

## Posición en el árbol de compilación (SP EXE)

```
XfsCoreSupport ──┐
Log             ──┤──► XfsCoreExe ──► SpCdm.exe
                           │           SpPin.exe
                           │           SpSiu.exe
                           └─────────► (cualquier SP EXE)
```
