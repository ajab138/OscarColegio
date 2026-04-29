# XfsBaseDevice

Biblioteca estática C++ que actúa como **capa base común** para la integración de dispositivos físicos con el framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsBaseDevice` implementa el patrón **Strategy** para separar:

- Las **operaciones comunes** a cualquier dispositivo XFS (`open`, `close`, `cancel`, `polling`).
- La **lógica específica** de cada dispositivo, delegada mediante interfaces abstractas que deben implementar las clases concretas (CDM, CIM, IPM, etc.).

Es el bloque fundamental del que heredan `XfsBaseCdm`, `XfsBaseCim`, `XfsBaseIpm` y similares.

## Arquitectura

```
IXfsCommonDevice          (XfsCoreExe)
       │
       ▼
CXfsBaseDevice  ◄──── IXfsExecutorCommonDevice  (implementada por cada dispositivo concreto)
       │
       ├── CXfsEventsSyse  →  IXfsEventsSyse
       └── CTimerPolling   →  ITimerPolling
```

`CXfsBaseDevice` implementa `IXfsCommonDevice` delegando cada operación en los métodos virtuales de `IXfsExecutorCommonDevice`. Las clases concretas de dispositivo solo necesitan implementar esos métodos específicos.

## Interfaces publicadas

### `IXfsBaseDevice.h`

| Interfaz | Descripción |
|---|---|
| `ITimerPolling` | Permite modificar en tiempo de ejecución el intervalo de polling del dispositivo. |
| `IXfsEventsSyse` | Publica eventos de sistema XFS hacia el framework. |
| `IXfsExecutorCommonDevice` | Contrato que toda clase concreta de dispositivo debe implementar. |

#### `ITimerPolling`

```cpp
virtual void changeTimerPolling(uint32_t timePolling) = 0;
```

#### `IXfsEventsSyse`

```cpp
virtual void fireEventDeviceStatus(obj::xfs::DevStatus& devStatus) const = 0;
virtual void fireEventHardwareError(obj::xfs::HwError& hwError) const = 0;
virtual void fireEventSoftwareError(obj::xfs::HwError& hwError) const = 0;
virtual void fireEventUserError(obj::xfs::HwError& hwError) const = 0;
virtual void fireEventFraudAttempt(obj::xfs::HwError& hwError) const = 0;
```

#### `IXfsExecutorCommonDevice`

```cpp
virtual TXfs::XFSError openDevice(std::string lpszLogicalName, TXfs::xfs_version version) = 0;
virtual bool closeDevice() = 0;
virtual bool cancelDevice() = 0;
virtual void pollingDevice() = 0;
virtual void sessionOpened(uint16_t hService) {};   // opcional
virtual void sessionClosed(uint16_t hService) {};   // opcional
```

### `XfsBaseDevice.h`

| Clase | Descripción |
|---|---|
| `CXfsBaseDevice` | Implementación base concreta de `IXfsCommonDevice`. Instancia internamente `CXfsEventsSyse` y `CTimerPolling`. |
| `CXfsEventsSyse` | Implementación de `IXfsEventsSyse`. Serializa los objetos de evento y los eleva al framework mediante `notifySystemEvent`. |
| `CTimerPolling` | Implementación de `ITimerPolling`. Delega en `IXfsCommonDevice::setTimePolling`. |

## Dependencias

| Dependencia | Ruta include | Propósito |
|---|---|---|
| `XfsCoreExe` | `../../externs/XfsCoreExe/inc` | `IXfsCommonDevice`, `SpDeviceManager`, `SpMain`, `AMainThread` |
| `XfsCoreSupport` | `../../externs/XfsCoreSupport/inc` | `IXfsSerializable` |
| `XfsObjects` | `../../externs/XfsObjects/inc` | Tipos XFS (`obj::xfs::DevStatus`, `obj::xfs::HwError`, `TXfs::XFSError`, etc.) |
| `Log` | `../../externs/Log/inc` | `BswLog`, `BswTrace` (logging y trazas) |
| STL | — | `<mutex>`, `<string>`, `<vector>` |

> **Nota:** La dependencia de trazas (`BswLog*`, `BswTrace*`, `ulTraceParam`) se resuelve mediante variables externas definidas en el ejecutable que enlaza con esta biblioteca.

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
../../externs/XfsBaseDevice/inc/          ← cabeceras (.h)
../../externs/XfsBaseDevice/lib/<Plat>/<Config>/  ← XfsBaseDevice.lib
```

Esto permite que otros proyectos del repositorio consuman la biblioteca sin pasos adicionales.

## Uso en proyectos consumidores

1. Añadir `../../externs/XfsBaseDevice/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsBaseDevice/lib/$(Platform)/$(Configuration)/XfsBaseDevice.lib`.
3. Heredar `CXfsBaseDevice` en la clase concreta del dispositivo e implementar los métodos de `IXfsExecutorCommonDevice`.

```cpp
class MiDispositivo : public CXfsBaseDevice
{
protected:
    TXfs::XFSError openDevice(std::string name, TXfs::xfs_version version) override;
    bool closeDevice() override;
    bool cancelDevice() override;
    void pollingDevice() override;
};
```

## Ventajas

- **Desacoplamiento:** La lógica del framework XFS queda aislada de la implementación del hardware.
- **Reutilización:** Un único punto de implementación para open/close/cancel/polling compartido por todos los tipos de dispositivo (CDM, CIM, IPM…).
- **Extensibilidad:** Añadir soporte a un nuevo tipo de dispositivo solo requiere implementar `IXfsExecutorCommonDevice`.
- **Gestión automática de recursos:** Uso de `std::unique_ptr` para `CXfsEventsSyse` y `CTimerPolling`; sin fugas de memoria.
- **Distribución simplificada:** El evento post-build garantiza que `inc/` y `lib/` siempre estén actualizados para los consumidores del repositorio.

## Posición en el árbol de compilación (EXE)

```
XfsCoreSupport  ──┐
XfsCoreExe      ──┤
XfsObjects      ──┤──► XfsBaseDevice ──► XfsBase{Cdm,Cim,Ipm,...} ──► Sp<DEVICE>.exe
Log             ──┘
```
