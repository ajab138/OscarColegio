# XfsObjectsApi

Biblioteca estática C++ que contiene los **tipos de datos del servicio API** (clase de servicio genérica XFS) dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsObjectsApi` define los objetos de dominio del servicio API XFS:

- **Tipos de entrada/salida** para todos los comandos `getInfo` y `execute` de la clase API.
- **Objetos de evento** para los mensajes de servicio y ejecución (`StatusChanged`, `SyncPicture`, etc.).
- **Enumeraciones y constantes** de la clase API: comandos, eventos, estados de transacción, estados de dispositivo y códigos de error (`TApi::*`).
- **Serialización/deserialización** de todos los tipos mediante Protobuf (`IXfsSerializable`).

Es la biblioteca de tipos que consumen tanto los servicios API concretos como sus clientes (ejecutables o capas de soporte).

## Arquitectura

Todos los tipos de datos heredan de `IXfsSerializable` (de `XfsCoreSupport`) y cada clase encapsula internamente un mensaje Protobuf generado desde `XfsProtos`:

```
IXfsSerializable          (XfsCoreSupport)
       │
       ├── obj::api::ApiTransactionState    (APITRANSACTIONSTATE)
       ├── obj::api::ApiTransactionInfo     (APITRANSACTIONINFO)
       ├── obj::api::ServiceInfo            (SERVICEINFO)
       ├── obj::api::DeviceInfo             (DEVICEINFO)
       ├── obj::api::E2EAuthentication      (E2EAUTHENTICATION)
       ├── obj::api::StatusChanged          (STATUSCHANGED)
       ├── obj::api::ApiGetNonce            (APIGETNONCE)
       ├── obj::api::ApiGetNonceOut         (APIGETNONCEOUT)
       ├── obj::api::ApiSecureCommand       (APISECURECOMMAND)
       ├── obj::api::ApiSecureCommandOut    (APISECURECOMMANDOUT)
       ├── obj::api::ApiSecureQuery         (APISECUREQUERY)
       ├── obj::api::ApiSecureQueryOut      (APISECUREQUERYOUT)
       ├── obj::api::ApiSecureOperation     (APISECUREOPERATION)
       ├── obj::api::ApiSyncPicture         (APISYNCPICTURE)
       ├── obj::api::ApiSyncPictureInfo     (APISYNCPICTUREINFO)
       ├── obj::api::ApiCameraInfo          (APICAMERAINFO)
       ├── obj::api::ApiClearReason         (APICLEARREASON)
       ├── obj::api::ApiPicture             (APIPICTURE)
       ├── obj::api::ApiPictures            (colección de ApiPicture)
       ├── obj::api::VendorModeInfo         (VENDORMODEINFO)
       ├── obj::api::ServiceInterface       (SERVICEINTERFACE)
       └── obj::api::CompoundDevice         (COMPOUNDDEVICE)
```

## Clases publicadas

### Tipos de comandos — `TApi.h`

Contiene los enumerados y constantes del servicio API:

| Tipo | Descripción |
|---|---|
| `TApi::ApiInfoCmd` | Comandos `getInfo`: `TransactionState`, `ServiceInfo`, `ApiSecureQuery`, `ApiSyncPicture`. |
| `TApi::ApiExecCmd` | Comandos `execute`: `SetTransactionState`, `GetCommandNonce`, `SecureCommand`, `ClearCommandNonce`, `ApiSyncPicture`, `ApiStartSecureOperation`. |
| `TApi::ApiEvents` | Eventos del servicio: `SrveStatusChanged`, `ExeeErrorInfo`, `SrveNonceCleared`, `SrveSyncPicture`. |
| `TApi::ApiError` | Códigos de error específicos: `keyNoValue`, `InvalidNonce`, `InvalidToken`, `InvalidTokenHmac`, `InvalidAuth`. |
| `TApi::ApiTransactionState` | Estado de transacción: `Active` / `Inactive`. |
| `TApi::DeviceStatus` | Estado de dispositivo: `Online`, `Offline`, `PowerOff`, `NoDevice`, `HwError`, `UserError`, `Busy`, `FraudAttempt`, `PotentialFraud`. |

### Objetos de consulta (`getInfo`)

| Clase | Header | Campos principales |
|---|---|---|
| `obj::api::ApiTransactionState` | `ApiTransactionState.h` | `fwState`, `transactionInfo` |
| `obj::api::ApiTransactionInfo` | `ApiTransactionInfo.h` | `lpszTransactionID`, `mapExtra` |
| `obj::api::ServiceInfo` | `ApiServiceInfo.h` | `vDeviceInformation`, `vendorModeInformation`, `serviceInterface`, `vCompoundDevices`, `lpszServiceProviderVersion`, `mapExtra` |
| `obj::api::ApiSecureQuery` | `ApiSecureQuery.h` | `dwCategoryID`, `lpszInputToken`, `lpszResponseNonce`, `lpvInputData`, `mapExtra` |
| `obj::api::ApiSecureQueryOut` | `ApiSecureQueryOut.h` | `dwCategoryID`, `lpszOutputToken`, `lpvOutputData`, `mapExtra` |
| `obj::api::ApiSyncPicture` | `ApiSyncPicture.h` | `wCamera`, `fwEvents`, `lpszText`, `lpszFolder`, `mapExtra` |
| `obj::api::ApiSyncPictureInfo` | `ApiSyncPictureInfo.h` | `vCameraInfo`, `mapExtra` |

### Objetos de ejecución (`execute`)

| Clase | Header | Campos principales |
|---|---|---|
| `obj::api::ApiGetNonce` | `ApiGetNonce.h` | `mapExtra` |
| `obj::api::ApiGetNonceOut` | `ApiGetNonceOut.h` | `lpszNonce`, `mapExtra` |
| `obj::api::ApiSecureCommand` | `ApiSecureCommand.h` | `dwCommandID`, `lpszInputToken`, `lpszResponseNonce`, `lpvInputData`, `mapExtra` |
| `obj::api::ApiSecureCommandOut` | `ApiSecureCommandOut.h` | `dwCommandID`, `lpszOutputToken`, `lpvOutputData`, `mapExtra` |
| `obj::api::ApiSecureOperation` | `ApiSecureOperation.h` | `dwOperation`, `lpszUniqueID` |
| `obj::api::ApiClearReason` | `ApiClearReason.h` | `usReason` |

### Objetos de evento

| Clase | Header | Campos principales |
|---|---|---|
| `obj::api::StatusChanged` | `ApiStatusChanged.h` | `lpvOldStatus`, `lpvNewStatus` |
| `obj::api::ApiPicture` | `ApiPicture.h` | `wCamera`, `lpszFileName`, `wStatus`, `mapExtra` |
| `obj::api::ApiPictures` | `ApiPictures.h` | Colección de `ApiPicture` |
| `obj::api::ApiCameraInfo` | `ApiCameraInfo.h` | `wCamera`, `wStatus`, `wMedia`, `fwAvailableEvents`, `fwConfiguredEvents`, `usMaxDataLength`, `mapExtra` |

### Objetos de información de servicio (anidados en `ServiceInfo`)

| Clase | Header | Descripción |
|---|---|---|
| `obj::api::DeviceInfo` | `ApiDeviceInfo.h` | Información de dispositivo: modelo, número de serie, revisión, firmware y software. |
| `obj::api::E2EAuthentication` | `ApiE2EAuthentication.h` | Configuración E2E: comandos execute protegidos, categorías getInfo y timeout de nonce. |
| `obj::api::VendorModeInfo` | `ApiVendorModeInfo.h` | Permisos en modo proveedor: sesiones abiertas y comandos execute permitidos. |
| `obj::api::ServiceInterface` | `ApiServiceInterface.h` | Interfaz de servicio expuesta. |
| `obj::api::CompoundDevice` | `ApiCompoundDevice.h` | Información de dispositivo compuesto. |

### Header de acceso unificado — `ApiObjectsApi.h`

Incluye todos los tipos de datos de la clase API en un único `#include` (excluye `ApiServiceInfo.h` y los tipos de información de servicio anidados, que se incluyen por separado).

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | `IXfsSerializable` y utilidades del core XFS |
| `XfsProtos` | `../../externs/XfsProtos/inc` | Mensajes Protobuf generados (`proto::api::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `vcpkg` | `C:\Projects\vcpkg\installed\$(PlatformShortName)-windows-static\include` | Dependencias de terceros (Protobuf, etc.) |
| STL | — | `<string>`, `<vector>`, `<map>`, `<memory>`, `<cstdint>` |

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
../../externs/XfsObjectsApi/inc/                          ← cabeceras (.h)
../../externs/XfsObjectsApi/lib/<Plat>/<Config>/          ← XfsObjectsApi.lib
```

Esto permite que los proyectos consumidores accedan a la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsObjectsApi/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsObjectsApi/lib/$(Platform)/$(Configuration)/XfsObjectsApi.lib`.
3. Incluir el header agregador o el header específico del tipo necesario:

```cpp
// Acceso a todos los tipos de datos API
#include "ApiObjectsApi.h"

// O sólo al tipo necesario
#include "ApiTransactionState.h"
#include "ApiServiceInfo.h"
```

Ejemplo de uso con serialización:

```cpp
obj::api::ApiTransactionState state;
state.fwState = TApi::getNumber(TApi::ApiTransactionState::Active);
state.transactionInfo.lpszTransactionID = "TX-001";

// Serializar a buffer Protobuf
std::vector<uint8_t> buffer;
state.serialize(buffer);

// Deserializar desde buffer
obj::api::ApiTransactionState stateOut;
stateOut.deserialize(buffer);
```

## Posición en el árbol de compilación

```
XfsCoreSupport  ──┐
XfsProtos       ──┤──► XfsObjectsApi ──► XfsBaseApi / SpApi.exe
Log             ──┘
```
