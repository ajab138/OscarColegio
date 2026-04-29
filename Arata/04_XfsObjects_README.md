# XfsObjects

Biblioteca estática C++ que proporciona los **tipos de datos y enumeraciones comunes del framework XFS ARATA** (eXtensions for Financial Services), compartidos por todas las clases de servicio.

## Propósito

`XfsObjects` define los objetos y tipos fundamentales que forman el vocabulario genérico del protocolo XFS:

- **Enumeraciones XFS** de uso transversal: errores, versiones, clases de servicio, estados de dispositivo y eventos de sistema.
- **Objetos de evento de sistema** serializables (sistema `WFS_SYSE_*`): desconexión de aplicación, estado de dispositivo, error de hardware, mensaje no entregable y error de versión.
- **Tipos contenedor genéricos** serializables: número entero simple y vector de números.

Es la dependencia de tipos más básica del framework; la consumen tanto las bibliotecas base (`XfsBaseCdm`, etc.) como las implementaciones concretas de cada dispositivo.

## Arquitectura

```
XfsCoreSupport (IXfsSerializable)
       │
       ▼
 obj::xfs::*       ←── IXfsSerializable
       │
       ├── ObjNumber
       ├── ObjVectorNumber
       ├── AppDisc
       ├── DevStatus
       ├── HwError
       ├── UnDevMsg
       └── VrsnError

 TXfs::*           (enumeraciones — solo cabecera, sin instancias)
       ├── XFSError
       ├── xfs_version
       ├── ServiceClass
       ├── SystemEventId
       ├── ErrorAction
       └── DeviceStatus
```

Todos los objetos del namespace `obj::xfs` implementan `IXfsSerializable` y utilizan Protocol Buffers (via `XfsProtos`) para su serialización binaria.

## Cabeceras publicadas

### `XfsObjects.h` — cabecera paraguas

Incluye todas las cabeceras del proyecto. Punto de entrada único para los consumidores.

---

### `TXfs.h` — enumeraciones XFS comunes

Namespace `TXfs`. Todas las enumeraciones incluyen funciones auxiliares `getNumber()` / `toString()`.

| Tipo | Descripción |
|---|---|
| `XFSError` | Códigos de retorno estándar XFS (`SP_SUCCESS`, `SP_ERR_*`). Incluye `getXFSError(int32_t)`. |
| `xfs_version` | Versiones del protocolo XFS: `v3_03` … `v3_50`. |
| `ServiceClass` | Identificadores numéricos de clase de servicio XFS: `CDM(3)`, `IDC(2)`, `SIU(8)`, etc. |
| `SystemEventId` | IDs de evento de sistema (`WFS_SYSE_*`): `SyseHardwareError`, `SyseDeviceStatus`, `SyseAppDisconnect`, etc. |
| `ErrorAction` | Máscara de bits con las acciones recomendadas ante un error (`Reset`, `SwError`, `HwMaint`, `Suspend`, …). |
| `DeviceStatus` | Estado del dispositivo (`Online`, `Offline`, `PowerOff`, `NoDevice`, `HwError`, `Busy`, `FraudAttempt`, …). |

---

### `ObjNumber.h`

| Clase | Campo | Descripción |
|---|---|---|
| `obj::xfs::ObjNumber` | `uint32_t number` | Envuelve un entero sin signo de 32 bits como objeto serializable XFS. Usado como carga útil genérica de eventos (p. ej., `fireExeeDelayedDispense`). |

---

### `ObjVectorNumber.h`

| Clase | Campo | Descripción |
|---|---|---|
| `obj::xfs::ObjVectorNumber` | `std::vector<uint32_t> vectorNumber` | Envuelve una lista de enteros como objeto serializable XFS. |

---

### `XfsAppDisc.h`

| Clase | Campos | Descripción |
|---|---|---|
| `obj::xfs::AppDisc` | `lpszLogicalName`, `lpszWorkstationName`, `lpszAppID` | Datos del evento de sistema `WFS_SYSE_APP_DISCONNECT`. Notifica la desconexión de una aplicación del servicio. |

---

### `XfsDevStatus.h`

| Clase | Campos | Descripción |
|---|---|---|
| `obj::xfs::DevStatus` | `lpszPhysicalName`, `lpszWorkstationName`, `TXfs::DeviceStatus dwState` | Datos del evento de sistema `WFS_SYSE_DEVICE_STATUS`. Informa del cambio de estado del dispositivo. |

---

### `XfsHwError.h`

| Clase | Campos | Descripción |
|---|---|---|
| `obj::xfs::HwError` | `lpszLogicalName`, `lpszPhysicalName`, `lpszWorkstationName`, `lpszAppID`, `dwAction`, `dwSize`, `lpbDescription` | Datos del evento `WFS_SYSE_HARDWARE_ERROR`. Describe el error hardware y las acciones recomendadas. |

---

### `XfsUnDevMsg.h`

| Clase | Campos | Descripción |
|---|---|---|
| `obj::xfs::UnDevMsg` | `lpszLogicalName`, `lpszWorkstationName`, `lpszAppID`, `dwMsg`, `dwSize`, `lpbDescription`, `lpResult` | Datos del evento `WFS_SYSE_UNDELIVERABLE_MSG`. Contiene el mensaje que no pudo ser entregado al destinatario. |

---

### `XfsVrsnError.h`

| Clase | Campos | Descripción |
|---|---|---|
| `obj::xfs::VrsnError` | `lpszLogicalName`, `lpszWorkstationName`, `lpszAppID`, `dwSize`, `lpbDescription`, `lpVersion` | Datos del evento `WFS_SYSE_VERSION_ERROR`. Informa de incompatibilidades de versión detectadas durante la negociación. |

---

### Interfaz `IXfsSerializable` (implementada por todos los objetos)

```cpp
virtual void toString(std::stringstream& stream, uint8_t indent = 0);
virtual void serialize(std::vector<uint8_t>& buffer);
virtual void deserialize(const std::vector<uint8_t>& buffer);
```

Cada objeto expone además acceso a su mensaje Protobuf subyacente:

```cpp
proto::xfs::T* getProto() const;
void fromProto();   // proto → campos públicos
void toProto();     // campos públicos → proto
void setProto(const proto::xfs::T& proto);
```

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | `IXfsSerializable`, `IXfsApi.h` (constantes `SP_*`) |
| `XfsProtos` | `../../externs/XfsProtos/inc` | Mensajes Protobuf generados (`proto::xfs::*`) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| vcpkg | `$(PlatformShortName)-windows-static` | Dependencias de Protobuf y otras librerías externas |
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
../../externs/XfsObjects/inc/                          ← cabeceras (.h)
../../externs/XfsObjects/lib/<Plat>/<Config>/          ← XfsObjects.lib
```

Esto permite que el resto del framework consuma la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsObjects/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsObjects/lib/$(Platform)/$(Configuration)/XfsObjects.lib`.
3. Incluir `XfsObjects.h` para acceder a todos los tipos, o la cabecera específica que se necesite.

```cpp
#include "XfsObjects.h"

// Usar enumeraciones XFS
TXfs::XFSError err = TXfs::XFSError::Success;
TXfs::DeviceStatus status = TXfs::DeviceStatus::Online;

// Construir objeto de evento de hardware
obj::xfs::HwError hwError;
hwError.lpszLogicalName = "CDM";
hwError.dwAction = TXfs::getNumber(TXfs::ErrorAction::Reset);

// Serializar / deserializar
std::vector<uint8_t> buffer;
hwError.serialize(buffer);

obj::xfs::HwError hwError2;
hwError2.deserialize(buffer);
```

## Posición en el árbol de compilación

```
XfsCoreSupport ──┐
XfsProtos      ──┤──► XfsObjects ──► XfsBaseCdm, XfsObjectsCdm, ...  ──► SpCdm.exe
Log            ──┘
```
