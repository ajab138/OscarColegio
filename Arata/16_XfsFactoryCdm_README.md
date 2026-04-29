# XfsFactoryCdm

Biblioteca estática C++ que actúa como **fábrica de objetos SPI versionados para dispensadores de efectivo (CDM)** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsFactoryCdm` implementa el patrón **Factory Method** para instanciar los objetos `IXfsSpiObject` adecuados a cada combinación de comando/categoría y versión del protocolo XFS:

- **Creación versionada** de objetos de entrada y salida para los comandos `getInfo`, `execute` y los eventos CDM.
- **Despacho por versión** hacia `CXfsFactoryCdm320` (XFS 3.20) o `CXfsFactoryCdm330` (XFS 3.30) según el `wVersion` recibido.
- **Declaración del rango de versiones** soportado (`3.20` – `3.30`) mediante `versionRange()`.
- **Identificación de clase de servicio** CDM mediante `getServiceClass()`.

Es la fábrica de objetos SPI que consume el framework para serializar y deserializar las estructuras XFS específicas de CDM.

## Arquitectura

```
IXfsFactory          (XfsSpiObjects)
       │
       ▼
CXfsFactoryCdm
       │
       ├── CXfsFactoryCdm320   →  objSPI::cdm320::*   (XFS v3.20)
       └── CXfsFactoryCdm330   →  objSPI::cdm330::*   (XFS v3.30)
```

`CXfsFactoryCdm` implementa `IXfsFactory` de forma `final` en cada método de creación. Tras convertir el `DWORD` de comando/categoría al enum tipado correspondiente (`TCdm::CdmInfoCmd`, `TCdm::CdmExecCmd`, `TCdm::CdmEvents`) y el `WORD` de versión a `TXfs::xfs_version`, delega en la clase de versión apropiada.

## Clases publicadas

### `XfsFactoryCdm.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryCdm` | Implementación de `IXfsFactory` para CDM. Despacha por versión a `CXfsFactoryCdm320` o `CXfsFactoryCdm330`. Declara rango de versiones `0x14031E03` (3.20–3.30) y clase de servicio `TCdm::SERVICE_CLASS_CDM`. |

```cpp
virtual std::unique_ptr<IXfsSpiObject> newExecuteInput(DWORD dwCommand, WORD wVersion) override final;
virtual std::unique_ptr<IXfsSpiObject> newExecuteOutput(DWORD dwCommand, WORD wVersion) override final;
virtual std::unique_ptr<IXfsSpiObject> newGetInfoInput(DWORD dwCategory, WORD wVersion) override final;
virtual std::unique_ptr<IXfsSpiObject> newGetInfoOutput(DWORD dwCategory, WORD wVersion) override final;
virtual std::unique_ptr<IXfsSpiObject> newEventOutput(DWORD dwEventID, WORD wVersion) override final;
virtual DWORD versionRange() override final;
virtual uint16_t getServiceClass() override final;
```

### `XfsFactoryCdm320.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryCdm320` | Fábrica estática de objetos SPI para XFS v3.20. Todos sus métodos son `static`. |

### `XfsFactoryCdm330.h`

| Clase | Descripción |
|---|---|
| `CXfsFactoryCdm330` | Fábrica estática de objetos SPI para XFS v3.30. Extiende los comandos de 3.20 con `SetBlacklist`, `SynchronizeCommand`, `GetBlacklist`, `GetItemInfo` y `GetAllItemsInfo`. |

## Objetos creados por versión

### GetInfo — Input

| Categoría (`CdmInfoCmd`) | v3.20 | v3.30 |
|---|---|---|
| `Status` | `ObjVoid` | `ObjVoid` |
| `Capabilities` | `ObjVoid` | `ObjVoid` |
| `CashUnitInfo` | `ObjVoid` | `ObjVoid` |
| `TellerInfo` | `CdmTellerInfo320` | `CdmTellerInfo330` |
| `CurrencyExp` | `ObjVoid` | `ObjVoid` |
| `PresentStatus` | `ObjUshort` | `ObjUshort` |
| `MixTypes` | `ObjVoid` | `ObjVoid` |
| `MixTable` | `ObjUshort` | `ObjUshort` |
| `GetBlacklist` | — | `ObjVoid` |
| `GetItemInfo` | — | `CdmGetItemInfo330` |
| `GetAllItemsInfo` | — | `CdmGetAllItemsInfo330` |

### GetInfo — Output

| Categoría (`CdmInfoCmd`) | v3.20 | v3.30 |
|---|---|---|
| `Status` | `CdmStatus320` | `CdmStatus330` |
| `Capabilities` | `CdmCaps320` | `CdmCaps330` |
| `CashUnitInfo` | `CdmCuInfo320` | `CdmCuInfo330` |
| `TellerInfo` | `CdmTellerDetails320` | `CdmTellerDetails330` |
| `CurrencyExp` | `CdmCurrencyExp320` | `CdmCurrencyExp330` |
| `PresentStatus` | `CdmPresentStatus320` | `CdmPresentStatus330` |
| `MixTypes` | `CdmMixType320` | `CdmMixType330` |
| `MixTable` | `CdmMixTable320` | `CdmMixTable330` |
| `GetBlacklist` | — | `CdmBlacklist330` |
| `GetItemInfo` | — | `CdmItemInfo330` |
| `GetAllItemsInfo` | — | `CdmAllItemsInfo330` |

### Execute — Input

| Comando (`CdmExecCmd`) | v3.20 | v3.30 |
|---|---|---|
| `Denominate` | `CdmDenominate320` | `CdmDenominate330` |
| `Dispense` | `CdmDispense320` | `CdmDispense330` |
| `Count` | `CdmPhysicalCu320` | `CdmPhysicalCu330` |
| `Present` | `ObjUshort` | `ObjUshort` |
| `Reject` | `ObjVoid` | `ObjVoid` |
| `Retract` | `CdmRetract320` | `CdmRetract330` |
| `OpenShutter` / `CloseShutter` | `ObjUshort` | `ObjUshort` |
| `SetTellerInfo` | `CdmTellerUpdate320` | `CdmTellerUpdate330` |
| `SetCashUnitInfo` | `CdmCuInfo320` | `CdmCuInfo330` |
| `StartExchange` | `CdmStartEx320` | `CdmStartEx330` |
| `EndExchange` | `CdmCuInfo320` | `CdmCuInfo330` |
| `OpenSafeDoor` | `ObjVoid` | `ObjVoid` |
| `CalibrateCashUnit` | `CdmCalibrate320` | `CdmCalibrate330` |
| `SetMixTable` | `CdmMixTable320` | `CdmMixTable330` |
| `Reset` / `TestCashUnits` | `CdmItemPosition320` | `CdmItemPosition330` |
| `SetGuidanceLight` | `CdmSetGuidLight320` | `CdmSetGuidLight330` |
| `PowerSaveControl` | `CdmPowerSaveControl320` | `CdmPowerSaveControl330` |
| `PrepareDispense` | `CdmPrepareDispense320` | `CdmPrepareDispense330` |
| `SetBlacklist` | — | `CdmBlacklist330` |
| `SynchronizeCommand` | — | `CdmSynchronizeCommand330` |

### Execute — Output

| Comando (`CdmExecCmd`) | v3.20 | v3.30 |
|---|---|---|
| `Denominate` / `Dispense` | `CdmDenomination320` | `CdmDenomination330` |
| `Count` | `CdmCount320` | `CdmCount330` |
| `Retract` | `CdmItemNumberList320` | `CdmItemNumberList330` |
| `StartExchange` | `CdmCuInfo320` | `CdmCuInfo330` |
| `CalibrateCashUnit` | `CdmCalibrate320` | `CdmCalibrate330` |
| `TestCashUnits` | `CdmItemPosition320` | `CdmItemPosition330` |
| Resto | `ObjVoid` | `ObjVoid` |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `xfsCoreDll` | `../../externs/xfsCoreDll/inc` | `IXfsFactory`, `IXfsSpiObject` |
| `xfsCoreSupport` | `../../externs/xfsCoreSupport/inc` | Tipos de soporte del core XFS (`TXfs::xfs_version`, etc.) |
| `xfsObjectsCdm` | `../../externs/xfsObjectsCdm/inc` | Tipos CDM genéricos (`TCdm::*`) |
| `xfsObjects` | `../../externs/xfsObjects/inc` | Tipos XFS genéricos (`ObjVoid`, `ObjUshort`) |
| `XfsSpiObjects` | `../../externs/XfsSpiObjects/inc` | Interfaz `IXfsSpiObject` y tipos SPI genéricos |
| `XfsSpiObjectsCdm320` | `../../externs/XfsSpiObjectsCdm320/inc` | Objetos SPI versionados XFS 3.20 (`objSPI::cdm320::*`) |
| `XfsSpiObjectsCdm330` | `../../externs/XfsSpiObjectsCdm330/inc` | Objetos SPI versionados XFS 3.30 (`objSPI::cdm330::*`) |
| `Log` | `../../externs/Log/inc` | Logging y trazas |

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
../../externs/XfsFactoryCdm/inc/                         ← cabeceras (.h)
../../externs/XfsFactoryCdm/lib/<Plat>/<Config>/         ← XfsFactoryCdm.lib
```

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsFactoryCdm/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsFactoryCdm/lib/$(Platform)/$(Configuration)/XfsFactoryCdm.lib`.
3. Registrar una instancia de `CXfsFactoryCdm` en el framework para que este pueda crear los objetos SPI de CDM.

```cpp
#include <XfsFactoryCdm.h>

// Registro de la fábrica en el framework XFS
auto pFactory = std::make_unique<CXfsFactoryCdm>();
xfsCore.registerFactory(std::move(pFactory));
```

## Posición en el árbol de compilación

```
XfsSpiObjectsCdm320  ──┐
XfsSpiObjectsCdm330  ──┤
XfsSpiObjects        ──┤──► XfsFactoryCdm ──► SpCdm.exe
xfsObjectsCdm        ──┤
xfsObjects           ──┤
xfsCoreDll           ──┘
```
