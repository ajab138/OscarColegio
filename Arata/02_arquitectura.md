# Arquitectura del Framework XFS — SpG600

## Diagrama de Capas

```
┌─────────────────────────────────────────────────────────────────┐
│                    Aplicación Financiera                        │
│                    (ATM Software / Branch)                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │ XFS SPI API (SP<Class><Device>.dll)
┌──────────────────────────▼──────────────────────────────────────┐
│  CAPA 1: XfsCoreDll (Cliente SPI)                               │
│  - WFPOpen, WFPExecute, WFPGetInfo, etc.                        │
│  - APipeClient    (Named Pipes)                                 │
│  - ASession       (gestión de sesiones)                         │
│  - SpTrace        (tracing SPI)                                 │
│  - XfsFactory     (serialización)                               │
└──────────────────────────┬──────────────────────────────────────┘
                           │ MSG_PIPE Protocol (IPipeProtocol)
                           | (SP<Device>.exe)
┌──────────────────────────▼──────────────────────────────────────┐
│  CAPA 2: XfsCoreExe (Servidor)                                  │
│  - SpG600.exe          (proceso servidor)                       │
│  - APipeManager        (gestión de pipes servidor)              │
│  - CSpXfs              (coordinador singleton)                  │
│  - SpDeviceManager     (gestión de dispositivos)                │
│  - AMainThread + DeviceManager Thread                           │
└──────────────────────────┬──────────────────────────────────────┘
                           │ IXfsCommonDevice + IXfsClassDevice
┌──────────────────────────▼──────────────────────────────────────┐
│  CAPA 3: XfsBase (Lógica Abstracta)                             │
│  - CXfsBaseDevice        (operaciones comunes)                  │
│  - CXfsBaseCim/Cdm/Ipm   (lógica por clase)                     │
│  - IXfsCache             (gestión de caché)                     │
│  - IXfsEvents            (sistema de eventos)                   │
└──────────────────────────┬──────────────────────────────────────┘
                           │ IXfsExecutor (comandos protegidos)
┌──────────────────────────▼──────────────────────────────────────┐
│  CAPA 4: SpG600 (Implementación Concreta)                       │
│  - CSpCommonG600       (gestión común del dispositivo)          │
│  - CSpCimG600          (aceptador de billetes)                  │
│  - CSpCdmG600          (dispensador de billetes)                │
│  - CSpIpmG600          (procesador de documentos)               │
│  - WfsRecyclerFacade   (interacción con hardware-Devapi)        │
| main() Inyecta a través de singleton CSpXfs instancias únicas   │
|        de CspCommonG600, CSpCimG600, CSpCdmG600 y CSpIpmG600    │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Protocolos específicos del hardware
┌──────────────────────────▼──────────────────────────────────────┐
│               Dispositivo Físico (Reciclador G600)              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Descripción de Capas

| Capa | Proyecto | Rol |
|------|----------|-----|
| **1 — Cliente SPI** | `XfsCoreDll` | Implementa la SPI de XFS. Serializa comandos y los envía al servidor via Named Pipes. |
| **2 — Servidor** | `XfsCoreExe` | Recibe mensajes del cliente, coordina dispositivos y gestiona el ciclo de vida de sesiones. |
| **3 — Lógica Abstracta** | `XfsBase` | Define comportamiento común y por clase de dispositivo, independiente del hardware concreto. |
| **4 — Implementación Concreta** | `SpG600` | Adapta los comandos XFS al protocolo específico del hardware Reciclador G600. |

---

## Protocolos de Comunicación entre Capas

| Frontera | Mecanismo |
|----------|-----------|
| App → Capa 1 | WFS API (`SP<Device>.dll`) |
| Capa 1 → Capa 2 | `MSG_PIPE` Protocol (`IPipeProtocol`) via Named Pipes |
| Capa 2 → Capa 3 | `IXfsCommonDevice` + `IXfsClassDevice` |
| Capa 3 → Capa 4 | `IXfsExecutor` (comandos protegidos) |
| Capa 4 → Hardware | Protocolos específicos del Reciclador G600 |
