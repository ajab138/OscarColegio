# XfsSpiObjects

Biblioteca estática C++ que proporciona los **objetos de datos SPI genéricos del framework XFS ARATA** (eXtensions for Financial Services).

## Propósito

`XfsSpiObjects` define e implementa los objetos de transporte utilizados por la capa SPI (Service Provider Interface) para serializar, deserializar y convertir estructuras XFS nativas (`LPVOID`/`LPWFSRESULT`) en objetos C++ tipados.

Cada clase de este proyecto implementa la interfaz dual `IXfsSpiObject`, que combina:

- **`IXfsSerializable`** — serialización binaria e impresión diagnóstica.
- **`IXfsSPI`** — inicialización desde estructuras XFS nativas y creación de estructuras de vuelta.

El punto de entrada principal es la función `newSystemEvent`, que actúa como **factoría** para instanciar el objeto SPI adecuado a partir del identificador de evento de sistema (`TXfs::SystemEventId`).

## Arquitectura

```
IXfsSpiObject  (= IXfsSerializable + IXfsSPI)
       │
       ├── objSPI::xfs::DevStatus      ◄── obj::xfs::DevStatus
       ├── objSPI::xfs::HwError        ◄── obj::xfs::HwError
       ├── objSPI::xfs::AppDisc        ◄── obj::xfs::AppDisc
       ├── objSPI::xfs::UnDevMsg       ◄── obj::xfs::UnDevMsg
       ├── objSPI::xfs::VrsnError      ◄── obj::xfs::VrsnError
       ├── objSPI::xfs::ObjUlong       ◄── obj::xfs::ObjNumber
       ├── objSPI::xfs::ObjUshort      ◄── obj::xfs::ObjNumber
       ├── objSPI::xfs::ObjVectorUshort◄── obj::xfs::ObjVectorNumber
       └── objSPI::xfs::ObjVoid
```

Todas las clases se encuentran en el namespace `objSPI::xfs`. Cada una hereda del objeto de datos lógico correspondiente (de `XfsObjects`) y de `IXfsSpiObject`, añadiendo la implementación de la capa SPI.

## Clases publicadas

### `XfsSpiObjects.h` — factoría de eventos de sistema

```cpp
std::unique_ptr<IXfsSpiObject> newSystemEvent(TXfs::SystemEventId dwEvent);
```

| `SystemEventId` | Objeto instanciado |
|---|---|
| `SyseDeviceStatus` | `objSPI::xfs::DevStatus` |
| `SyseUndeliverableMsg` | `objSPI::xfs::UnDevMsg` |
| `SyseAppDisconnect` | `objSPI::xfs::AppDisc` |
| `SyseHardwareError` / `SyseSoftwareError` / `SyseUserError` | `objSPI::xfs::HwError` |
| `SyseLockRequested` | `objSPI::xfs::ObjVoid` |
| `SyseVersionError` | `objSPI::xfs::VrsnError` |

### Clases SPI de eventos de sistema

| Clase | Cabecera | Tipo base lógico | Descripción |
|---|---|---|---|
| `objSPI::xfs::DevStatus` | `XfsSpiDevStatus.h` | `obj::xfs::DevStatus` | Estado del dispositivo XFS. |
| `objSPI::xfs::HwError` | `XfsSpiHwError.h` | `obj::xfs::HwError` | Error de hardware, software o usuario. |
| `objSPI::xfs::AppDisc` | `XfsSpiAppDisc.h` | `obj::xfs::AppDisc` | Desconexión de aplicación cliente. |
| `objSPI::xfs::UnDevMsg` | `XfsSpiUnDevMsg.h` | `obj::xfs::UnDevMsg` | Mensaje no entregable al dispositivo. |
| `objSPI::xfs::VrsnError` | `XfsSpiVrsnError.h` | `obj::xfs::VrsnError` | Error de versión en la negociación SPI. |

### Clases SPI de tipos primitivos

| Clase | Cabecera | Tipo base lógico | Descripción |
|---|---|---|---|
| `objSPI::xfs::ObjUlong` | `ObjSpiULong.h` | `obj::xfs::ObjNumber` | Valor numérico `ULONG` (p.ej. número de evento). |
| `objSPI::xfs::ObjUshort` | `ObjSpiUshort.h` | `obj::xfs::ObjNumber` | Valor numérico `USHORT`. |
| `objSPI::xfs::ObjVectorUshort` | `ObjSpiVectorUshort.h` | `obj::xfs::ObjVectorNumber` | Vector de valores `USHORT`. |
| `objSPI::xfs::ObjVoid` | `ObjSpiVoid.h` | — | Objeto vacío (sin payload de datos). |

### Interfaz `IXfsSpiObject`

Cada clase implementa los siguientes métodos:

#### `IXfsSerializable`

| Método | Descripción |
|---|---|
| `toString(stream, indent)` | Vuelca el contenido en formato legible para diagnóstico/log. |
| `serialize(buffer)` | Serializa el objeto a un buffer binario. |
| `deserialize(buffer)` | Reconstruye el objeto desde un buffer binario. |

#### `IXfsSPI`

| Método | Descripción |
|---|---|
| `initialize(lpData)` | Inicializa el objeto desde una estructura XFS nativa (`LPVOID`). |
| `createStruct(lpWFSResult)` | Rellena una estructura XFS nativa de salida desde el objeto. |
| `createStructFromObject(lpWFSResult, lpObjectBase, lpXfsStruct)` | Crea la estructura XFS a partir de un objeto base y un buffer de destino. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsObjects` | `../../externs/XfsObjects/inc` | Tipos lógicos base (`obj::xfs::*`, `obj::xfs::ObjNumber`, etc.) |
| `XfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IXfsSpiObject`, `IXfsSerializable`, `IXfsSPI`, `TXfs` |
| `XfsCoreDll` | `../../externs/xfsCoreDll/inc` | Enlace con la DLL core XFS (tipos nativos `LPVOID`, `LPWFSRESULT`) |
| `XFS320` | `../../externs/XFS320/inc` | Definiciones del estándar CEN/XFS 3.20 |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| STL | — | `<sstream>`, `<vector>`, `<cstdint>`, `<memory>` |

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
../../externs/XfsSpiObjects/inc/                         ← cabeceras (.h)
../../externs/XfsSpiObjects/lib/<Plat>/<Config>/         ← XfsSpiObjects.lib
```

Esto permite que los proyectos SPI consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjects/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjects/lib/$(Platform)/$(Configuration)/XfsSpiObjects.lib`.
3. Incluir `XfsSpiObjects.h` para acceder a la factoría y a todas las clases SPI.

```cpp
#include "XfsSpiObjects.h"

// Instanciar el objeto SPI correcto para un evento de sistema recibido
std::unique_ptr<IXfsSpiObject> pObj = newSystemEvent(TXfs::SystemEventId::SyseDeviceStatus);
if (pObj)
{
    pObj->initialize(lpWFSResult->lpBuffer);
    // Usar el objeto: pObj->toString(...), pObj->serialize(...), etc.
}
```

## Posición en el árbol de compilación

```
XfsCoreSupport   ──┐
XfsCoreDll       ──┤
XfsObjects       ──┤──► XfsSpiObjects ──► SpXxx.dll (SPI)
XFS320           ──┤
Log              ──┘
```
