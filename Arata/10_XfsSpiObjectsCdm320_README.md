# XfsSpiObjectsCdm320

Biblioteca estática C++ que proporciona los **objetos SPI de la versión XFS 3.20 para dispensadores de efectivo (CDM)** dentro del framework XFS ARATA.

## Propósito

`XfsSpiObjectsCdm320` implementa el puente bidireccional entre las estructuras C nativas de la especificación XFS 3.20 (`WFSCDM*`) y los objetos de dominio de alto nivel (`obj::cdm::*`):

- **Deserialización** (`initialize`): parsea un puntero `LPVOID` con una estructura `WFSCDM*` nativa y rellena el objeto de dominio.
- **Serialización** (`createStruct` / `createStructFromObject`): construye la estructura `WFSCDM*` en memoria XFS (mediante `WFMAllocateMore`) a partir del estado del objeto de dominio.
- **Serialización binaria** (`serialize` / `deserialize`): codificación/decodificación binaria de los objetos para transporte o persistencia.
- **Representación textual** (`toString`): volcado legible del objeto para logging y trazas.

Es consumida directamente por la capa SPI de la DLL de servicio CDM que comunica con el XFS Manager.

## Arquitectura

```
obj::cdm::CdmXxx        (XfsObjectsCdm)
       │
       ▼
CdmXxx320  ──────────── IXfsSpiObject  (xfsCoreSupport)
  namespace objSPI::cdm320               │
                                         ├── IXfsSPI
                                         │     initialize(LPVOID)
                                         │     createStruct(LPVOID)
                                         │     createStructFromObject(LPVOID, const void*, LPVOID)
                                         └── IXfsSerializable
                                               toString(stringstream&)
                                               serialize(vector<uint8_t>&)
                                               deserialize(const vector<uint8_t>&)
```

Cada clase del namespace `objSPI::cdm320` hereda en doble cadena: hereda el estado del objeto de dominio (`obj::cdm::CdmXxx`) y la interfaz SPI (`IXfsSpiObject`). Las implementaciones de los métodos son `final`.

## Objetos publicados

Todos los objetos residen en el namespace `objSPI::cdm320` y se exponen a través de la cabecera agregadora `XfsSpiObjectsCdm320.h`.

### Información de estado y capacidades

| Clase | Objeto de dominio base | Estructura XFS 3.20 |
|---|---|---|
| `CdmStatus320` | `obj::cdm::CdmStatus` | `WFSCDMSTATUS` |
| `CdmCaps320` | `obj::cdm::CdmCaps` | `WFSCDMCAPS` |

### Unidades de efectivo

| Clase | Objeto de dominio base | Estructura XFS 3.20 |
|---|---|---|
| `CdmCashUnit320` | `obj::cdm::CdmCashUnit` | `WFSCDMCASHUNIT` |
| `CdmCuInfo320` | `obj::cdm::CdmCuInfo` | `WFSCDMCUINFO` |
| `CdmCuError320` | `obj::cdm::CdmCuError` | `WFSCDMCUERROR` |
| `CdmPhysicalCu320` | `obj::cdm::CdmPhysicalCu` | `WFSCDMPHCU` |
| `CdmPhCu320` | `obj::cdm::CdmPhCu` | `WFSCDMPHCU` |
| `CdmCountedPhysCu320` | `obj::cdm::CdmCountedPhysCu` | `WFSCDMPHYSICALCU` |
| `CdmCountsChanged320` | `obj::cdm::CdmCountsChanged` | `WFSCDMCOUNTSCHANGED` |

### Denominación y mezcla

| Clase | Objeto de dominio base | Estructura XFS 3.20 |
|---|---|---|
| `CdmDenomination320` | `obj::cdm::CdmDenomination` | `WFSCDMDENOMINATION` |
| `CdmDenominate320` | `obj::cdm::CdmDenominate` | `WFSCDMDENOMINATE` |
| `CdmMixType320` | `obj::cdm::CdmMixType` | `WFSCDMMIXTYPE` |
| `CdmMixTable320` | `obj::cdm::CdmMixTable` | `WFSCDMMIXTABLE` |
| `CdmMixRow320` | `obj::cdm::CdmMixRow` | `WFSCDMMIXROW` |
| `CdmCurrencyExp320` | `obj::cdm::CdmCurrencyExp` | `WFSCDMCURRENCYEXP` |

### Comandos de operación

| Clase | Objeto de dominio base | Estructura XFS 3.20 |
|---|---|---|
| `CdmDispense320` | `obj::cdm::CdmDispense` | `WFSCDMDISPENSE` |
| `CdmCount320` | `obj::cdm::CdmCount` | `WFSCDMCOUNT` |
| `CdmRetract320` | `obj::cdm::CdmRetract` | `WFSCDMRETRACT` |
| `CdmPrepareDispense320` | `obj::cdm::CdmPrepareDispense` | `WFSCDMPREPAREDISPENSE` |
| `CdmStartEx320` | `obj::cdm::CdmStartEx` | `WFSCDMSTARTEX` |
| `CdmCalibrate320` | `obj::cdm::CdmCalibrate` | `WFSCDMCALIBRATE` |
| `CdmSetGuidLight320` | `obj::cdm::CdmSetGuidLight` | `WFSCDMSETGUIDLIGHT` |
| `CdmPowerSaveControl320` | `obj::cdm::CdmPowerSaveControl` | `WFSCDMPOWERSAVECONTROL` |

### Posición y presentación

| Clase | Objeto de dominio base | Estructura XFS 3.20 |
|---|---|---|
| `CdmOutPos320` | `obj::cdm::CdmOutPos` | `WFSCDMOUTPOS` |
| `CdmItemPosition320` | `obj::cdm::CdmItemPosition` | `WFSCDMITEMPOSITION` |
| `CdmItemNumber320` | `obj::cdm::CdmItemNumber` | `WFSCDMITEMNUMBER` |
| `CdmItemNumberList320` | `obj::cdm::CdmItemNumberList` | `WFSCDMITEMNUMBERLIST` |
| `CdmPresentStatus320` | `obj::cdm::CdmPresentStatus` | `WFSCDMPRESENTSTATUS` |
| `CdmDevicePosition320` | `obj::cdm::CdmDevicePosition` | `WFSCDMDEVICEPOSITION` |
| `CdmPowerSaveChange320` | `obj::cdm::CdmPowerSaveChange` | `WFSCDMPOWERSAVECHANGE` |

### Información de cajero (Teller)

| Clase | Objeto de dominio base | Estructura XFS 3.20 |
|---|---|---|
| `CdmTellerInfo320` | `obj::cdm::CdmTellerInfo` | `WFSCDMTELLERINFO` |
| `CdmTellerDetails320` | `obj::cdm::CdmTellerDetails` | `WFSCDMTELLERDETAILS` |
| `CdmTellerTotals320` | `obj::cdm::CdmTellerTotals` | `WFSCDMTELLERTOTALS` |
| `CdmTellerUpdate320` | `obj::cdm::CdmTellerUpdate` | `WFSCDMTELLERUPDATE` |

## Interfaz de cada objeto (`IXfsSpiObject`)

### `IXfsSPI`

```cpp
// Rellena el objeto de dominio a partir de la estructura nativa XFS 3.20
virtual void initialize(LPVOID lpData) override final;

// Construye la estructura WFSCDM* dentro de un WFSRESULT (usa WFMAllocateMore)
virtual void createStruct(LPVOID lpWFSResult) override final;

// Construye la estructura WFSCDM* a partir de un objeto de dominio externo
virtual void createStructFromObject(LPVOID lpWFSResult, const void* lpObjectBase, LPVOID lpXfsStruct) override final;
```

### `IXfsSerializable`

```cpp
virtual void toString(std::stringstream& stream, uint8_t indent = 0) override final;
virtual void serialize(std::vector<uint8_t>& buffer) override final;
virtual void deserialize(const std::vector<uint8_t>& buffer) override final;
```

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | `IXfsSpiObject`, `IXfsSPI`, `IXfsSerializable`, `XfsSPIObject.h` |
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | Utilidades de la DLL de servicio XFS, `WFMAllocateMore` |
| `XfsObjectsCdm` | `../../externs/XfsObjectsCdm/inc` | Objetos de dominio CDM (`obj::cdm::*`, `TCdm::*`) |
| `xfs320` | `../../externs/xfs320/inc` | Estructuras C nativas XFS 3.20 (`xfscdm.h`, `xfsspi.h`) |
| `Log` | `../../externs/Log/inc` | Logging y trazas |
| STL | — | `<sstream>`, `<vector>` |

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Cabeceras precompiladas:** no utilizadas


## Uso en proyectos consumidores

1. Añadir `../../externs/XfsSpiObjectsCdm320/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsSpiObjectsCdm320/lib/$(Platform)/$(Configuration)/XfsSpiObjectsCdm320.lib`.
3. Incluir `XfsSpiObjectsCdm320.h` para acceder a todos los objetos SPI CDM 3.20.

```cpp
#include "XfsSpiObjectsCdm320.h"

// Deserializar una respuesta WFS_GETINFO_CDM_STATUS
objSPI::cdm320::CdmStatus320 status;
status.initialize(lpWFSResult->lpBuffer);

// Serializar para enviar al WFS Manager
objSPI::cdm320::CdmDenomination320 denom;
denom = /* ... rellenar desde lógica de negocio ... */;
denom.createStruct(lpWFSResult);
```

## Posición en el árbol de compilación (SPI DLL)

```
xfsCoreSupport     ──┐
xfsCoreDll         ──┤
XfsObjectsCdm      ──┤──► XfsSpiObjectsCdm320 ──► SpCdm.dll (SPI)
xfs320             ──┤
Log                ──┘
```
