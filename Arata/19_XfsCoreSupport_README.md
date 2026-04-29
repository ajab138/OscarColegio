# XfsCoreSupport

Biblioteca estática C++ que proporciona los **primitivos de infraestructura de bajo nivel** utilizados por el framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsCoreSupport` agrupa las utilidades transversales que necesitan tanto la DLL del Service Provider (`XfsCoreDll`) como el proceso EXE del SP:

- **Sincronización** (`ACritSec`): envoltorio de mutex Win32 con nombre, para secciones críticas entre procesos.
- **Gestión de eventos Win32** (`AEvent`, `AEventHandler`): clase base abstracta para objetos evento con I/O superpuesto (overlapped) y multiplexor de hasta 64 eventos con espera configurable.
- **Manejo de excepciones** (`AException`, `NonMfcExcSupport`): excepción propia con texto/código de error/fichero/línea y traductor SEH→C++ con macros `TRY_SP`/`CATCH_SP`.
- **Lista intrusiva genérica** (`ALink`, `AList<T>`): lista doblemente enlazada intrusiva, type-safe mediante plantilla.
- **Peticiones XFS** (`ARequest`): nodo de lista que encapsula una solicitud SPI en curso (RequestID, HWND, función, timeout, mensaje de retorno).
- **Configuración desde registro** (`CRegConfig`): acceso a `HKLM\SOFTWARE\XFS\PHYSICAL_SERVICES\<SpName>` para leer valores DWORD y cadena de configuración del SP.
- **Detección USB** (`UsbDetection`): consulta si un dispositivo USB está conectado (por VID/PID) y selecciona el módulo software asociado al primer/último dispositivo enchufado de una lista.
- **Protocolo de pipe** (`IPipeProtocol.h`): define la cabecera binaria `HEADER` (64 bytes, magic «ArataFUJ»), el mensaje `MSG_PIPE`, las estructuras de payload (`PayloadOpen`, `RegisterDeregisterPayload`) y todas las constantes de comando/respuesta/evento del canal named-pipe entre DLL y EXE.
- **API XFS** (`IXfsApi.h`): códigos de error `SP_ERR_*`, mensajes de ventana `SP_*_COMPLETE`/`SP_*_EVENT`, máscaras de eventos y tipos fundamentales (`HSERVICE`, `HAPP`, `REQUESTID`, …). Incluye `getXfsErrorString()` para diagnóstico.
- **Serialización XFS** (`IXfsSerializable.h`): interfaz `IXfsSerializable` con `serialize`/`deserialize`/`toString` y la excepción `XfsObjectException`.
- **Utilidad de flags de enumeración** (`Flags.h`): plantilla `Flags<Enum>` que permite combinar valores de enumeración mediante operadores de bits, con comprobaciones type-safe en tiempo de compilación.

## Arquitectura

```
XfsCoreDll / SP EXE
        │
        ├── IPipeProtocol.h   (protocolo binario named-pipe DLL↔EXE)
        ├── IXfsApi.h         (tipos XFS, error codes, mensajes Win)
        ├── IXfsSerializable  (interfaz serialización objetos XFS)
        │
        ├── AEvent ──► AEventHandler   (multiplexación eventos Win32)
        ├── ACritSec                   (sección crítica con mutex)
        ├── ALink ──► AList<T>         (lista intrusiva genérica)
        ├── ARequest : ALink           (petición SPI en curso)
        │
        ├── AException                 (excepción básica)
        ├── NonMfcExcSupport           (SEH → C++ / TRY_SP / CATCH_SP)
        ├── AllocMem                   (heap Win32 C-linkage)
        ├── CRegConfig                 (lectura registro Windows)
        ├── UsbDetection               (detección USB por VID/PID)
        └── Flags<Enum>                (combinación flags enum)
```

## Interfaces publicadas

### `IXfsApi.h`

Tipos fundamentales XFS y constantes de error/mensajes.

| Símbolo | Descripción |
|---|---|
| `HSERVICE` / `HAPP` / `HPROVIDER` / `REQUESTID` | Handles y tipos XFS básicos. |
| `SP_SUCCESS` | Valor de éxito (0). |
| `SP_ERR_*` | Códigos de error XFS (–1 … –59). |
| `SP_OPEN_COMPLETE` … `SP_EXECUTE_COMPLETE` | Mensajes `WM_USER+n` de finalización de operación SPI. |
| `SP_EXECUTE_EVENT` … `SP_SYSTEM_EVENT` | Mensajes `WM_USER+n` de eventos del dispositivo. |
| `SP_MASK_SERVICE_EVENTS` … `SP_MASK_EXECUTE_EVENTS` | Máscaras de clase de evento para `Register`/`Deregister`. |
| `SP_TRACE_API` … `SP_TRACE_MGR` | Niveles de traza XFS. |
| `getXfsErrorString(hResult)` | Devuelve la cadena descriptiva de un código de error XFS. |

### `IXfsSerializable.h`

| Clase / Interfaz | Descripción |
|---|---|
| `IXfsSerializable` | Interfaz de serialización: `serialize` (→ `vector<uint8_t>`), `deserialize` (← `vector<uint8_t>`), `toString` (texto indentado), `streamToString` (utilidad final). |
| `XfsObjectException` | Excepción de serialización XFS con nombre de clase, detalles y `HRESULT`. |

### `IPipeProtocol.h`

| Símbolo | Descripción |
|---|---|
| `HEADER` | Cabecera fija de 64 bytes: magic «ArataFUJ», versión, cmd, hService, requestID, timeout, HWND, punteros de sesión/petición, comando/evento y hResult, longitud de payload. |
| `MSG_PIPE` | Mensaje completo: `HEADER` + `vector<uint8_t>` de payload variable. |
| `PayloadOpen` | Payload de 224 bytes para `CMD_REQ_OPEN` (workstation, logical_name, HAPP, appID, versiones SPI/servicio). |
| `RegisterDeregisterPayload` | Payload de 16 bytes para `CMD_REQ_REGISTER` / `CMD_REQ_DEREGISTER` (HWND registrado, máscara de eventos). |
| `CMD_REQ_*` / `CMD_RSP_*` | Constantes de comando de petición (0-11) y respuesta (100-111). |
| `CMD_EXECUTE_EVENT` … `CMD_SYSTEM_EVENT` | Constantes de evento (201-204). |
| `ValidateHeader(h)` | Valida que el magic number del header sea correcto. |
| `MAX_PAYLOAD_SIZE` | Tamaño máximo de payload: 2 MB. |
| `VERSION_PROTOCOL` | Versión actual del protocolo: `0x0001`. |

### `ACritSec.h`

| Clase | Descripción |
|---|---|
| `ACritSec` | Sección crítica basada en mutex con nombre Win32. `enter()` bloquea; `exit()` libera. |

### `AEvent.h` / `AEventHandler.h`

| Clase | Descripción |
|---|---|
| `AEvent` | Clase base abstracta para un evento Win32 con I/O superpuesto. Expone `getEventHandle()`, `setEvent()`, `resetEvent()` y el método virtual puro `manageEvent()`. |
| `AEventHandler` | Gestiona un array de hasta 64 punteros `AEvent*`. `addObject`/`removeObject` administran el array; `waitForObject(maxTime)` espera con `WaitForMultipleObjects` y devuelve el objeto señalizado. |

### `ALink.h` / `AList.h`

| Clase | Descripción |
|---|---|
| `ALink` | Nodo intrusivo con puntero `Next`. Base de todos los elementos enlazables. |
| `AListBase` | Implementación de lista circular sin plantilla: `insert`, `append`, `remove`, `getFirst`, `getNext`, `getLast`, `removeFirst`, `isInList`, `isEmpty`, `selectiveInsert`. |
| `AList<T>` | Wrapper type-safe sobre `AListBase` mediante plantilla. |

### `ARequest.h`

| Clase | Descripción |
|---|---|
| `ARequest : ALink` | Petición SPI en curso. Almacena `RequestId`, `WindowHandle`, `Function` (enum `OPEN`…`IMMEDIATEEXECUTE`), timeout y `MessageCode` de retorno. Posee el flag `IsCanceling`. |

Constante: `WITHOUT_TIMEOUT = 0xFFFFFFFF`.

### `AException.h`

| Clase | Descripción |
|---|---|
| `AException` | Excepción básica con texto (256 chars), nombre de fichero, número de línea y código de error Win32. |

### `NonMfcExcSupport.h`

| Símbolo | Descripción |
|---|---|
| `SPException` | Excepción C++ que encapsula una excepción estructurada (SEH). Almacena código, fichero, línea y módulo. |
| `i_translator` | Traductor SEH (`_set_se_translator`) que convierte excepciones Win32 en `SPException`. |
| `TRY_SP` / `CATCH_SP(e)` | Macros que instalan el traductor y añaden captura interna de `SPException` y `long`. |
| `ThrowMemoryException()` | Lanza excepción de falta de memoria. |


### `RegConfig.h` ⚠️ *Pendiente: usar `KEY_READ | KEY_WOW64_32KEY` para leer la rama de 32 bits desde un proceso de 64 bits.

| Clase / Método | Descripción |
|---|---|
| `CRegConfig(pSpName)` | Abre la clave `HKLM\SOFTWARE\XFS\PHYSICAL_SERVICES\<SpName>`* |
| `GetString(pSubKey, pValueName, pData, pdwLen)` | Lee un valor REG_SZ bajo una subclave de CONFIGURATION. |
| `GetString(pValueName, pData, pdwLen)` | Lee un valor REG_SZ directamente bajo la clave del SP. |
| `GetDWORD(pSubKey, pValueName, pdwData)` | Lee un valor REG_DWORD bajo una subclave de CONFIGURATION. |
| `GetDWORD(pValueName, pdwData)` | Lee un valor REG_DWORD directamente bajo la clave del SP. |

Clave base: `HKEY_LOCAL_MACHINE\SOFTWARE\XFS\PHYSICAL_SERVICES`.

### `UsbDetection.h` ⚠️ *Candidato a mover a una biblioteca de utilidades independiente (`XfsUtil.lib`).*

| Símbolo | Descripción |
|---|---|
| `DEVICE_MODULE` | Estructura con `szDeviceId` (VID_XXXX&PID_YYYY), `szDeviceName` y `szModuleName`. |
| `CheckUsbConnection(pszDeviceId)` | Devuelve `true` si el dispositivo USB está conectado. |
| `GetModuleForDeviceConnected(pDevMod, n, pszLogMsg)` | Devuelve el módulo asociado al primer dispositivo conectado de la lista. |
| `GetModuleForLastDevicePlugged(pDevMod, n, pszLogMsg)` | Devuelve el módulo asociado al último dispositivo enchufado de la lista. |

### `Flags.h`

| Plantilla | Descripción |
|---|---|
| `Flags<Enum>` | Contenedor de flags sobre un `enum`. Soporta `\|`, `&`, `\|=`, `&=`, `has(flag)`, `hasAll(flags)`, `empty()`, `raw()`, `fromRaw(v)`. Requiere tipo `enum` (verificado con `static_assert`). |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `Log` | `../../externs/Log/inc` | `BswLog` (logging estructurado, incluido en compilación) |
| Win32 API | — | Mutex, eventos, I/O superpuesto, registro, USB (SetupAPI) |
| STL | — | `<vector>`, `<string>`, `<sstream>`, `<type_traits>` |

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
../../externs/XfsCoreSupport/inc/                         ← cabeceras (.h)
../../externs/XfsCoreSupport/lib/<Plat>/<Config>/         ← XfsCoreSupport.lib
```

Esto permite que `XfsCoreDll` y los procesos EXE de SP consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsCoreSupport/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsCoreSupport/lib/$(Platform)/$(Configuration)/XfsCoreSupport.lib`.

```cpp
// Ejemplo de uso de la lista intrusiva y peticiones
#include "AList.h"
#include "ARequest.h"

AList<ARequest> pendingRequests;
auto* req = new ARequest(requestId, hwnd, ARequest::GETINFO, 5000);
pendingRequests.append(req);
// ...
pendingRequests.remove(req);
```

```cpp
// Ejemplo de lectura de configuración del SP
#include "RegConfig.h"

CRegConfig config("SpCdm");
DWORD timeout = 0;
config.GetDWORD("CONFIGURATION", "Timeout", &timeout);
```

```cpp
// Ejemplo de uso del protocolo de pipe
#include "IPipeProtocol.h"

MSG_PIPE msg;
msg.header.cmd     = CMD_REQ_GETINFO;
msg.header.version = VERSION_PROTOCOL;
// Rellenar el resto de campos y enviar por el named pipe...
```

## Posición en el árbol de compilación

```
Log ──┐
      ├──► XfsCoreSupport ──► XfsCoreDll ──► SpCdm.dll
      │                   │                  SpPin.dll
      │                   │                  SpSiu.dll
      │                   │                  (cualquier SP DLL)
      │                   │
      │                   └──► SpCdm.exe
      │                        SpPin.exe
      │                        (cualquier SP EXE)
      └──► (otros consumidores directos)
```
