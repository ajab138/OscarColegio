# SpG600

Ejecutable C++ que actúa como **Service Provider XFS para el reciclador G600** dentro del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`SpG600` es el punto de entrada del proceso XFS para el dispositivo reciclador G600. Implementa simultáneamente tres clases de dispositivo XFS:

- **CIM** (Cash-In Module): recepción y clasificación de billetes.
- **CDM** (Cash Dispenser Module): dispensado de efectivo.
- **IPM** (Item Processing Module): gestión de cheques.

Cada clase de dispositivo se instancia como una clase concreta que hereda de su base XFS correspondiente (`CXfsBaseCim`, `CXfsBaseCdm`, `CXfsBaseIpm`) e inyecta la fachada `WfsRecyclerFacade` para delegar las operaciones físicas en la DevApi del dispositivo.

## Arquitectura

```
main()
  │
  ├── WfsRecyclerFacade  (acceso unificado a la DevApi del G600)
  │       │
  │       ├── DevFacade          (singleton de bajo nivel — DevApi)
  │       └── WfsContext         (contexto de estado del dispositivo)
  │
  ├── CSpCommonG600  ──► CXfsBaseDevice   (open/close/cancel/polling)
  ├── CSpCimG600     ──► CXfsBaseCim      (operaciones CIM)
  ├── CSpCdmG600     ──► CXfsBaseCdm      (operaciones CDM)
  └── CSpIpmG600     ──► CXfsBaseIpm      (operaciones IPM)
          │
          └── CSpXfs (framework) ── registerCommonDevice / registerClassDevice
```

El `main` detecta si se trata de un comando de configuración (`install`, `remove`, `register`); si no lo es, realiza la inyección de dependencias antes de ceder el control a `CommonExeMain`.

## Clases principales

### `SpG600.h` / `SpG600.cpp`

| Clase | Base | Descripción |
|---|---|---|
| `CSpCommonG600` | `CXfsBaseDevice` | Gestiona ciclo de vida del dispositivo: `openDevice`, `closeDevice`, `cancelDevice`, `pollingDevice`. Inyecta `IXfsEventsSyse` e `ITimerPolling` en la fachada. |
| `CSpCimG600` | `CXfsBaseCim` | Clase concreta CIM. Implementa `executeCashInStart` y `executeCashIn` delegando en `WfsRecyclerFacade`. |
| `CSpCdmG600` | `CXfsBaseCdm` | Clase concreta CDM. Implementa `executeDenominate` (actualmente devuelve `DevNotReady`). |
| `CSpIpmG600` | `CXfsBaseIpm` | Clase concreta IPM. Implementa `executeMediaInfo` (actualmente devuelve `DevNotReady`). |

### `WfsFacade/WfsRecyclerFacade.h`

Fachada compartida entre todas las clases de dispositivo. Recibe las interfaces del framework mediante inyección:

| Método de inyección | Interfaz recibida |
|---|---|
| `injectCommonInterfaces` | `IXfsEventsSyse*`, `ITimerPolling*` |
| `injectCimInterfaces` | `IXfsCacheCim*`, `IXfsEventsSrveUsreCim*` |
| `injectCdmInterfaces` | `IXfsCacheCdm*`, `IXfsEventsSrveUsreCdm*` |
| `injectIpmInterfaces` | `IXfsCacheIpm*`, `IXfsEventsSrveUsreIpm*` |

Expone la API de alto nivel hacia el framework XFS:

```cpp
TXfs::XFSError  wfs_open_recycler() const;
bool            wfs_close_recycler() const;
void            wfs_polling() const;
bool            wfs_cancel() const;

// CIM
std::variant<TXfs::XFSError, TCim::CimError>
    executeCashInStart(const obj::cim::CimCashInStart& in);
std::variant<TXfs::XFSError, TCim::CimError>
    executeCashIn(obj::cim::CimNoteNumberList& out,
                  const IXfsEventsExeeCim& ExeeEvents);
```

### `DevFacade/DevFacade.h`

Singleton de acceso directo a la DevApi física del G600. Encapsula todas las llamadas de bajo nivel al dispositivo:

| Categoría | Métodos |
|---|---|
| Ciclo de vida | `dev_open`, `dev_close`, `dev_reset` |
| Estado e información | `dev_status`, `dev_cash_units`, `dev_hw_info` |
| Control del obturador | `dev_open_shutter`, `dev_close_shutter`, `dev_items_removed` |
| Aceptación de billetes | `dev_pre_accept`, `dev_accept_wait`, `dev_get_accept_notes` |
| Depósito | `dev_deposit`, `dev_reject`, `dev_return` |
| Dispensado | `dev_present` |
| Configuración | `dev_set_accept_notes` |
| Otros | `dev_cancel`, `dev_hw_reset`, `dev_hw_reset_reject` |

`DevFacade` genera eventos hacia `WfsRecyclerFacade` a través de `IEventToWfsFacade`.

## Estructuras de interfaces inyectadas

```cpp
struct CommonInterfaces {
    IXfsEventsSyse*  eventSyse;
    ITimerPolling*   timerPolling;
};
struct CimInterfaces {
    IXfsCacheCim*            cache;
    IXfsEventsSrveUsreCim*   eventSrveUsre;
};
struct CdmInterfaces {
    IXfsCacheCdm*            cache;
    IXfsEventsSrveUsreCdm*   eventSrveUsre;
};
struct IpmInterfaces {
    IXfsCacheIpm*            cache;
    IXfsEventsSrveUsreIpm*   eventSrveUsre;
};
```

## Dependencias

### Bibliotecas del framework XFS ARATA

| Biblioteca | Ruta include | Propósito |
|---|---|---|
| `XfsCoreExe` | `../../externs/XfsCoreExe/inc` | `ISpXfs`, `CSpXfs`, `CommonExeMain` |
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | Tipos y utilidades del core XFS |
| `XfsBaseDevice` | `../../externs/XfsBaseDevice/inc` | `CXfsBaseDevice`, `IXfsEventsSyse`, `ITimerPolling` |
| `XfsBaseCim` | `../../externs/XfsBaseCim/inc` | `CXfsBaseCim`, `IXfsBaseCim` |
| `XfsBaseCdm` | `../../externs/XfsBaseCdm/inc` | `CXfsBaseCdm`, `IXfsBaseCdm` |
| `XfsBaseIpm` | `../../externs/XfsBaseIpm/inc` | `CXfsBaseIpm`, `IXfsBaseIpm` |
| `XfsObjects` | `../../externs/XfsObjects/inc` | Tipos XFS genéricos (`TXfs::XFSError`, etc.) |
| `XfsObjectsCim` | `../../externs/XfsObjectsCim/inc` | Tipos CIM (`obj::cim::*`, `TCim::*`) |
| `XfsObjectsCdm` | `../../externs/XfsObjectsCdm/inc` | Tipos CDM (`obj::cdm::*`, `TCdm::*`) |
| `XfsObjectsIpm` | `../../externs/XfsObjectsIpm/inc` | Tipos IPM (`obj::ipm::*`, `TIpm::*`) |
| `XfsProtos` | `../../externs/XfsProtos/lib/...` | Mensajes Protocol Buffers del framework |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |

### Terceros (vcpkg)

| Biblioteca | Propósito |
|---|---|
| `libprotobuf` / `libprotobufd` | Serialización Protocol Buffers |
| `abseil` (`absl_*`) | Utilidades de Google usadas por protobuf |
| `utf8_range` | Validación UTF-8 (dependencia de protobuf) |

## Compilación

- **Tipo de salida:** Aplicación ejecutable (`.exe`)
- **Versión:** 12.0.0 (`fileversion.h`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Manejo de excepciones:** Asíncrono (`/EHa`)

### Directorio de salida

```
$(SolutionDir)bin\$(Configuration)\              ← Win32
$(SolutionDir)bin\$(Configuration)\$(Platform)\  ← x64
```

## Posición en el árbol de compilación

```
XfsCoreSupport  ──┐
XfsCoreExe      ──┤
XfsBaseDevice   ──┤
XfsBaseCim      ──┤
XfsBaseCdm      ──┤──► SpG600.exe
XfsBaseIpm      ──┤
XfsObjectsCim   ──┤
XfsObjectsCdm   ──┤
XfsObjectsIpm   ──┤
XfsProtos       ──┤
Log             ──┘
```

## Flujo de arranque

```
main(argc, argv)
  │
  ├── ¿argv contiene install/remove/register?
  │     └── Sí → saltar inyección (modo configuración)
  │
  ├── injectInterfacesToCSpXfs()
  │     ├── new WfsRecyclerFacade
  │     ├── new CSpCommonG600(facade) → injectCommonInterfaces
  │     ├── new CSpCimG600(facade)    → injectCimInterfaces
  │     ├── new CSpCdmG600(facade)    → injectCdmInterfaces
  │     ├── new CSpIpmG600(facade)    → injectIpmInterfaces
  │     └── CSpXfs::registerCommonDevice / registerClassDevice
  │
  └── CommonExeMain(argc, argv)   ← bucle principal del framework XFS
```

## Extensión del Service Provider

Para añadir soporte a un nuevo comando del dispositivo:

1. Localizar el método correspondiente en la clase concreta (`CSpCimG600`, `CSpCdmG600` o `CSpIpmG600`).
2. Eliminar la implementación provisional (`DevNotReady`) o el comentario que bloquea el override.
3. Implementar el método delegando en `WfsRecyclerFacade`.
4. Añadir el método correspondiente en `WfsRecyclerFacade` y delegar en `DevFacade` (o en la DevApi directamente).

```cpp
// Ejemplo: activar executeDispense en CSpCdmG600
inline virtual std::variant<TXfs::XFSError, TCdm::CdmError>
executeDispense(const obj::cdm::CdmDispense& in,
                obj::cdm::CdmDenomination& out,
                const IXfsEventsExeeCdm& ExeeEvents) override final
{
    return _facade->executeDispense(in, out, ExeeEvents);
}
```
