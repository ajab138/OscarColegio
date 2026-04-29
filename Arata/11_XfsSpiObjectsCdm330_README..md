# XfsSpiObjectsCdm330

Biblioteca estática C++ que implementa los **objetos SPI de CDM para la versión XFS 3.30** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsSpiObjectsCdm330` proporciona las clases concretas que traducen entre los objetos de dominio CDM genéricos (`obj::cdm::*`) y las estructuras binarias del protocolo XFS 3.30 (`WFSCDM*`):

- **Serialización / deserialización** de cada tipo CDM hacia/desde el buffer binario del SPI.
- **Inicialización** de objetos de dominio a partir de punteros de estructura XFS (`LPVOID lpData`).
- **Creación de estructuras XFS** (`LPWFSRESULT`) a partir de objetos de dominio, con soporte de herencia de puntero base.
- **Trazabilidad** mediante `toString` en formato legible para depuración y logging.

Todas las clases residen en el espacio de nombres `objSPI::cdm330` y son la contraparte de versión específica de los tipos neutros de `XfsObjectsCdm`.

## Arquitectura

```
obj::cdm::CdmXxx          (XfsObjectsCdm — objeto de dominio genérico)
       │
       ▼
CdmXxx330  ──── IXfsSpiObject
       │              │
       │              ├── IXfsSerializable  (serialize / deserialize / toString)
       │              └── IXfsSPI           (initialize / createStruct / createStructFromObject)
       │
       └── [implementación final, namespace objSPI::cdm330]
```

Cada clase hereda en doble vía: el **modelo de datos** del objeto CDM base y el **contrato SPI** de `IXfsSpiObject`. El método `initialize` rellena el objeto desde la estructura XFS 3.30; `createStruct` / `createStructFromObject` realizan la conversión inversa.

## Clases publicadas

### Cabecera de inclusión única: `XfsSpiObjectsCdm330.h`

Incluye automáticamente todas las clases de la biblioteca.

---

### Estado e información del dispositivo

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmStatus330` | `obj::cdm::CdmStatus` | Estado operativo del dispensador. |
| `CdmCaps330` | `obj::cdm::CdmCaps` | Capacidades del hardware CDM. |
| `CdmCuInfo330` | `obj::cdm::CdmCuInfo` | Información agregada de unidades de efectivo. |
| `CdmCurrencyExp330` | `obj::cdm::CdmCurrencyExp` | Exponentes de divisas configuradas. |
| `CdmPresentStatus330` | `obj::cdm::CdmPresentStatus` | Estado de la última presentación de billetes. |

### Unidades de efectivo

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmCashUnit330` | `obj::cdm::CdmCashUnit` | Unidad de efectivo lógica. |
| `CdmPhysicalCu330` | `obj::cdm::CdmPhysicalCu` | Unidad física de efectivo. |
| `CdmPhCu330` | `obj::cdm::CdmPhCu` | Representación alternativa de unidad física. |
| `CdmCountedPhysCu330` | `obj::cdm::CdmCountedPhysCu` | Resultado de conteo de unidad física. |
| `CdmCuError330` | `obj::cdm::CdmCuError` | Detalle de error asociado a una unidad de efectivo. |

### Mezcla de billetes

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmMixType330` | `obj::cdm::CdmMixType` | Descriptor de un tipo de mezcla. |
| `CdmMixTable330` | `obj::cdm::CdmMixTable` | Tabla de mezcla completa. |
| `CdmMixRow330` | `obj::cdm::CdmMixRow` | Fila individual de la tabla de mezcla. |

### Denominación y dispensación

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmDenominate330` | `obj::cdm::CdmDenominate` | Parámetros de entrada para calcular una denominación. |
| `CdmDenomination330` | `obj::cdm::CdmDenomination` | Resultado de denominación (desglose de billetes). |
| `CdmDispense330` | `obj::cdm::CdmDispense` | Parámetros del comando dispense. |
| `CdmCount330` | `obj::cdm::CdmCount` | Parámetros del comando count. |
| `CdmRetract330` | `obj::cdm::CdmRetract` | Parámetros del comando retract. |
| `CdmPrepareDispense330` | `obj::cdm::CdmPrepareDispense` | Parámetros del comando prepareDispense. |

### Información de billetes individuales

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmGetItemInfo330` | `obj::cdm::CdmGetItemInfo` | Consulta de información de un billete concreto. |
| `CdmGetAllItemsInfo330` | `obj::cdm::CdmGetAllItemsInfo` | Consulta de información de todos los billetes. |
| `CdmItemInfo330` | `obj::cdm::CdmItemInfo` | Información detallada de un billete. |
| `CdmItemInfoAll330` | `obj::cdm::CdmItemInfoAll` | Conjunto de información de todos los billetes. |
| `CdmItemInfoSummary330` | `obj::cdm::CdmItemInfoSummary` | Resumen de información de billetes. |
| `CdmAllItemsInfo330` | `obj::cdm::CdmAllItemsInfo` | Información completa de todos los ítems. |
| `CdmItemNumber330` | `obj::cdm::CdmItemNumber` | Número identificador de un ítem. |
| `CdmItemNumberList330` | `obj::cdm::CdmItemNumberList` | Lista de números de ítems. |
| `CdmItemPosition330` | `obj::cdm::CdmItemPosition` | Posición física de un ítem en el dispositivo. |
| `CdmSignature330` | `obj::cdm::CdmSignature` | Firma digital de un billete. |

### Lista negra

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmBlacklist330` | `obj::cdm::CdmBlacklist` | Lista negra de billetes. |
| `CdmBlacklistElement330` | `obj::cdm::CdmBlacklistElement` | Elemento individual de la lista negra. |

### Posiciones de salida y cajero

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmOutPos330` | `obj::cdm::CdmOutPos` | Posición de salida de billetes. |
| `CdmTellerInfo330` | `obj::cdm::CdmTellerInfo` | Identificación del cajero a consultar. |
| `CdmTellerDetails330` | `obj::cdm::CdmTellerDetails` | Detalle de información de un cajero. |
| `CdmTellerTotals330` | `obj::cdm::CdmTellerTotals` | Totales acumulados de un cajero. |
| `CdmTellerUpdate330` | `obj::cdm::CdmTellerUpdate` | Parámetros de actualización de cajero. |

### Comandos de control y configuración

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmCalibrate330` | `obj::cdm::CdmCalibrate` | Parámetros de calibración de unidades. |
| `CdmSetGuidLight330` | `obj::cdm::CdmSetGuidLight` | Control de la luz de guía. |
| `CdmPowerSaveControl330` | `obj::cdm::CdmPowerSaveControl` | Parámetros de control de ahorro de energía. |
| `CdmStartEx330` | `obj::cdm::CdmStartEx` | Parámetros de inicio de ciclo de exchange. |
| `CdmSynchronizeCommand330` | `obj::cdm::CdmSynchronizeCommand` | Parámetros de sincronización de comandos. |

### Eventos (objetos de notificación)

| Clase | Base CDM | Descripción |
|---|---|---|
| `CdmCountsChanged330` | `obj::cdm::CdmCountsChanged` | Notificación de cambio de contadores. |
| `CdmShutterStatusChanged330` | `obj::cdm::CdmShutterStatusChanged` | Notificación de cambio de estado del obturador. |
| `CdmPowerSaveChange330` | `obj::cdm::CdmPowerSaveChange` | Notificación de cambio de modo de ahorro de energía. |
| `CdmDevicePosition330` | `obj::cdm::CdmDevicePosition` | Notificación de cambio de posición del dispositivo. |
| `CdmIncompleteRetract330` | `obj::cdm::CdmIncompleteRetract` | Notificación de retracción incompleta. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IXfsSpiObject`, `IXfsSPI`, `IXfsSerializable` |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | Interfaces de la DLL XFS |
| `XfsObjectsCdm` | `../../externs/XfsObjectsCdm/inc` | Tipos base `obj::cdm::*` |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| `xfs330` | `../../externs/xfs330/inc` | Estructuras binarias del protocolo XFS 3.30 (`WFSCDM*`) |

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
../../externs/XfsSpiObjectsCdm330/inc/                         ← cabeceras (.h)
../../externs/XfsSpiObjectsCdm330/lib/<Plat>/<Config>/         ← XfsSpiObjectsCdm330.lib
```

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjectsCdm330/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjectsCdm330/lib/$(Platform)/$(Configuration)/XfsSpiObjectsCdm330.lib`.
3. Incluir la cabecera de entrada única para acceder a todas las clases:

```cpp
#include "XfsSpiObjectsCdm330.h"

// Crear y rellenar un objeto de estado a partir de una estructura XFS 3.30
objSPI::cdm330::CdmStatus330 status;
status.initialize(lpWfsResult->lpBuffer);

// Serializar para transmisión
std::vector<uint8_t> buffer;
status.serialize(buffer);

// Crear estructura XFS a partir del objeto de dominio
status.createStruct(lpWfsResult);
```

## Posición en el árbol de compilación (SPI)

```
xfs330            ──┐
xfsCoreSupport    ──┤
xfsCoreDll        ──┤──► XfsSpiObjectsCdm330 ──► SpCdm.dll (SPI)
XfsObjectsCdm     ──┤
Log               ──┘
```
