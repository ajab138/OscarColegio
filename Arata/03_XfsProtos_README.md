# XfsProtos

Biblioteca estática C++ que contiene los **mensajes Protocol Buffers (protobuf) generados** para todos los servicios XFS del framework XFS ARATA (eXtensions for Financial Services).

## Propósito

`XfsProtos` centraliza la serialización/deserialización de los objetos de dominio XFS mediante Google Protocol Buffers. Proporciona los tipos de mensaje compilados que se utilizan como capa de transporte entre los servicios XFS y la capa gRPC:

- **Serialización** de comandos, respuestas y eventos de los servicios CDM, CIM, IPM, PIN y de la API genérica XFS.
- **Definición de tipos** de datos comunes y específicos de cada clase de dispositivo en namespaces separados.
- **Independencia de versión** del esquema de mensajes: los `.pb.cc` / `.pb.h` se generan a partir de archivos `.proto` y se distribuyen como biblioteca compilada.

Es la dependencia de serialización que consumen los proyectos `XfsObjects*` y cualquier otro componente que necesite transmitir datos serializados.

## Arquitectura

```
XfsProtos
├── proto::api   (iApi.pb.h / iApi.pb.cc)   — Mensajes de la API genérica XFS
├── proto::xfs   (iXfs.pb.h / iXfs.pb.cc)   — Mensajes base y eventos genéricos XFS
├── proto::cdm   (iCdm.pb.h / iCdm.pb.cc)   — Mensajes del servicio CDM (dispensador)
├── proto::cim   (iCim.pb.h / iCim.pb.cc)   — Mensajes del servicio CIM (receptora)
├── proto::ipm   (iIpm.pb.h / iIpm.pb.cc)   — Mensajes del servicio IPM (impresora)
└── proto::pin   (iPin.pb.h / iPin.pb.cc)   — Mensajes del servicio PIN (teclado PIN)
```

Cada módulo define sus mensajes en el namespace `proto::<servicio>`. Todos los tipos heredan de `::google::protobuf::MessageLite`.

## Cabeceras publicadas

### `iXfs.pb.h` — `namespace proto::xfs`

Mensajes de infraestructura y eventos comunes XFS.

| Clase | Descripción |
|---|---|
| `APPDISC` | Desconexión de aplicación. |
| `DEVSTATUS` | Estado del dispositivo. |
| `HWERROR` | Error de hardware. |
| `ObjNumber` | Objeto numérico genérico. |
| `ObjVectorNumber` | Vector de valores numéricos. |
| `UNDEVMSG` | Mensaje de dispositivo no definido. |
| `VRSNERROR` | Error de versión. |

### `iApi.pb.h` — `namespace proto::api`

Mensajes de la API de gestión y administración de servicios XFS.

| Clase | Descripción |
|---|---|
| `SERVICEINFO` | Información del servicio. |
| `SERVICEINTERFACE` | Interfaz de servicio. |
| `DEVICEINFO` | Información del dispositivo. |
| `COMPOUNDDEVICE` | Dispositivo compuesto. |
| `STATUSCHANGED` | Cambio de estado. |
| `VENDORMODEINFO` | Información de modo vendedor. |
| `APITRANSACTIONINFO` | Información de transacción API. |
| `APITRANSACTIONSTATE` | Estado de transacción API. |
| `E2EAUTHENTICATION` | Autenticación end-to-end. |
| `APICLEARREASON` | Motivo de limpieza API. |
| `APISECUREOPERATION` | Operación segura API. |
| `APISECUREQUERY` / `APISECUREQUERYOUT` | Consulta segura API (entrada/salida). |
| `APISECURECOMMAND` / `APISECURECOMMANDOUT` | Comando seguro API (entrada/salida). |
| `APIGETNONCE` / `APIGETNONCEOUT` | Obtención de nonce (entrada/salida). |
| `APICAMERAINFO` | Información de cámara API. |
| `APIPICTURE` / `APIPICTURES` | Imagen / colección de imágenes. |
| `APISYNCPICTURE` / `APISYNCPICTUREINFO` | Sincronización de imagen. |
| `SOFTWARE` / `FIRMWARE` | Información de software / firmware. |

### `iCdm.pb.h` — `namespace proto::cdm`

Mensajes del servicio CDM (Cash Dispenser Module — dispensador de efectivo).

| Clase | Descripción |
|---|---|
| `STATUS` | Estado del dispensador. |
| `CAPS` | Capacidades del dispensador. |
| `CUINFO` | Información de unidades de efectivo. |
| `CASHUNIT` | Unidad de efectivo. |
| `PHYSICALCU` / `PHCU` | Unidad física / sub-unidad física. |
| `DENOMINATION` | Denominación de billetes. |
| `DENOMINATE` | Solicitud de cálculo de denominación. |
| `DISPENSE` | Solicitud de dispensación. |
| `COUNT` | Conteo de billetes. |
| `RETRACT` | Solicitud de retracción. |
| `CALIBRATE` | Calibración de unidades. |
| `MIXTYPE` / `MIXROW` / `MIXTABLE` | Tipo / fila / tabla de mezcla. |
| `CURRENCYEXP` | Exponente de divisa. |
| `TELLERINFO` / `TELLERDETAILS` / `TELLERTOTALS` / `TELLERUPDATE` | Gestión de cajero. |
| `ITEMINFO` / `ITEMINFOALL` / `ITEMINFOSUMMARY` | Información de billetes. |
| `ITEMNUMBER` / `ITEMNUMBERLIST` | Número(s) de ítem. |
| `GETITEMINFO` / `GETALLITEMSINFO` | Consultas de información de ítems. |
| `PRESENTSTATUS` | Estado de presentación. |
| `ITEMPOSITION` / `OUTPOS` / `DEVICEPOSITION` | Posicionamiento de ítems y dispositivo. |
| `BLACKLIST` / `BLACKLISTELEMENT` | Lista negra de billetes. |
| `CLASSIFICATIONLIST` / `CLASSIFICATIONELEMENT` | Lista / elemento de clasificación. |
| `CUERROR` | Error de unidad de efectivo. |
| `COUNTSCHANGED` | Cambio de contadores. |
| `COUNTEDPHYSCU` | Unidad física contada. |
| `INCOMPLETERETRACT` | Retracción incompleta. |
| `POWERSAVECONTROL` / `POWERSAVECHANGE` | Gestión de ahorro de energía. |
| `SETGUIDLIGHT` | Control de luz de guía. |
| `STARTEX` | Inicio de intercambio. |
| `PREPAREDISPENSE` | Preparación de dispensación. |
| `SHUTTERSTATUSCHANGED` | Cambio de estado del obturador. |
| `SIGNATURE` | Firma de seguridad. |
| `SYNCHRONIZECOMMAND` | Sincronización de comandos. |

### `iCim.pb.h` — `namespace proto::cim`

Mensajes del servicio CIM (Cash-In Module — receptora de efectivo).

| Clase | Descripción |
|---|---|
| `STATUS` | Estado de la receptora. |
| `CAPS` | Capacidades de la receptora. |
| `CASHIN` / `CASHINSTATUS` / `CASHINSTART` / `CASHINTYPE` | Ciclo de recepción de efectivo. |
| `CASHINLIMIT` / `AMOUNTLIMIT` | Límites de recepción. |
| `CASHINFO` | Información de efectivo. |
| `PHCU` / `PHCUCOUNTSTATUS` / `PHCUCAPABILITIES` | Unidad física y sus capacidades. |
| `CASHCOUNTSTATUS` / `CASHCAPABILITIES` | Estado / capacidades de conteo. |
| `CASHUNITLOCK` / `CASHUNITCOUNTSTATUS` / `CASHUNITCAPABILITIES` | Bloqueo / estado / capacidades de unidad. |
| `UNITLOCKCONTROL` / `DEVICELOCKCONTROL` / `DEVICELOCKSTATUS` | Control de bloqueos. |
| `PRESENT` / `PRESENTSTATUS` | Presentación de efectivo. |
| `RETRACT` / `MOVEITEMS` | Retracción / movimiento de ítems. |
| `COUNT` / `COUNTSCHANGED` | Conteo y cambios de contadores. |
| `ITEMINFO` / `ITEMINFOALL` / `ITEMINFOSUMMARY` | Información de billetes. |
| `ITEMPOSITION` | Posición de ítems. |
| `GETITEMINFO` / `GETALLITEMSINFO` | Consultas de información de ítems. |
| `NOTETYPE` / `NOTETYPELIST` / `NOTENUMBER` / `NOTENUMBERLIST` | Tipos y números de billete. |
| `CURRENCYEXP` | Exponente de divisa. |
| `TELLERINFO` / `TELLERDETAILS` / `TELLERTOTALS` / `TELLERUPDATE` | Gestión de cajero. |
| `POSITIONINFO` / `POSCAPS` / `POSCAPABILITIES` | Información y capacidades de posición. |
| `OUTPUT` / `INPOS` | Posición de salida / entrada. |
| `DEP` / `DEPRES` / `DEPINFO` / `DEPINFORES` / `DEPINFOSOURCE` / `DEPSOURCE` / `DEPSOURCERES` | Operaciones de depleción. |
| `REP` / `REPRES` / `REPINFO` / `REPINFORES` / `REPINFOTARGET` / `REPTARGET` / `REPTARGETRES` | Operaciones de reposición. |
| `P6INFO` / `P6SIGNATURE` / `P6SIGNATURESINDEX` / `P6COMPARESIGNATURE` / `P6COMPARERESULT` | Firmas P6. |
| `GETP6SIGNATURE` | Solicitud de firma P6. |
| `CONFIGURENOTEREADER` / `CONFIGURENOTEREADEROUT` | Configuración del lector de billetes. |
| `BLACKLIST` / `BLACKLISTELEMENT` | Lista negra. |
| `CLASSIFICATIONLIST` / `CLASSIFICATIONELEMENT` | Lista de clasificación. |
| `STARTEX` | Inicio de intercambio. |
| `INCOMPLETERETRACT` / `INCOMPLETEDEPLETE` / `INCOMPLETEREPLENISH` | Operaciones incompletas. |
| `POWERSAVECONTROL` / `POWERSAVECHANGE` | Gestión de ahorro de energía. |
| `SETGUIDLIGHT` / `SETMODE` | Luz de guía / modo de operación. |
| `SHUTTERSTATUSCHANGED` / `DEVICEPOSITION` | Obturador y posición del dispositivo. |
| `SYNCHRONIZECOMMAND` | Sincronización de comandos. |
| `CASHINSTATUS` | Estado de recepción de efectivo. |

### `iIpm.pb.h` — `namespace proto::ipm`

Mensajes del servicio IPM (Imager / Printer Module — módulo de impresora/escáner).

| Clase | Descripción |
|---|---|
| `STATUS` / `CAPS` | Estado y capacidades. |
| `TRANSSTATUS` | Estado de transacción. |
| `MEDIAIN` / `MEDIAINEND` / `MEDIAINREQUEST` | Entrada de medios. |
| `MEDIABIN` / `MEDIABINCAPS` / `MEDIABININFO` | Bandeja y capacidades de medios. |
| `MEDIASIZE` / `MEDIADETECTED` / `MEDIADATA` / `MEDIASTATUS` | Gestión de medios. |
| `MEDIAPRESENTED` / `MEDIAREJECTED` / `MEDIAREFUSED` | Estado de presentación/rechazo. |
| `PRESENTMEDIA` | Solicitud de presentación. |
| `RETRACTMEDIA` / `RETRACTMEDIAOUT` | Retracción de medios. |
| `IMAGEREQUEST` / `IMAGEDATA` | Solicitud/datos de imagen. |
| `READIMAGEIN` | Lectura de imagen de entrada. |
| `GETIMAGEAFTERPRINT` | Imagen obtenida tras impresión. |
| `PRINTTEXT` / `PRINTSIZE` | Impresión de texto y tamaño. |
| `CODELINEMAPPING` / `CODELINEMAPPINGOUT` | Mapeo de línea de código MICR. |
| `POSCAPS` / `POS` / `POSITION` | Posición y capacidades. |
| `NEXTITEMOUT` | Siguiente ítem de salida. |
| `ACCEPTITEM` | Aceptación de ítem. |
| `SETDESTINATION` | Destino de ítem. |
| `BINCAPS` | Capacidades de bandeja. |
| `MBERROR` | Error de bandeja de medios. |
| `XDATA` | Datos extendidos. |
| `THRESHOLD` / `SCANNERTHRESHOLD` | Umbral de detección. |
| `SUPPLYREPLEN` | Reposición de suministros. |
| `DEVICEPOSITION` | Posición del dispositivo. |
| `POWERSAVECONTROL` / `POWERSAVECHANGE` | Gestión de ahorro de energía. |
| `SETGUIDLIGHT` / `SETMODE` | Luz de guía / modo. |
| `SHUTTERSTATUSCHANGED` | Cambio de estado del obturador. |
| `SYNCHRONIZECOMMAND` | Sincronización de comandos. |
| `MEDIAINEND` | Fin de ciclo de entrada de medios. |

### `iPin.pb.h` — `namespace proto::pin`

Mensajes del servicio PIN (PIN Entry Device — teclado PIN / módulo criptográfico).

| Clase | Descripción |
|---|---|
| `STATUS` / `CAPS` | Estado y capacidades. |
| `GETPIN` | Captura de PIN. |
| `GETDATA` | Captura de datos. |
| `BLOCK` / `BLOCKEX` / `BLOCK340` | Bloqueo de PIN (variantes). |
| `CRYPT` / `CRYPT340` | Operaciones de cifrado. |
| `IMPORT` / `IMPORTKEYEX` / `IMPORTKEYBLOCK` | Importación de claves. |
| `IMPORTKEY340` / `IMPORTKEY340OUT` | Importación de claves (XFS 3.40). |
| `IMPORTRSAPUBLICKEY` / `IMPORTRSAPUBLICKEYOUTPUT` | Importación de clave pública RSA. |
| `IMPORTRSASIGNEDDESKEY` / `IMPORTRSASIGNEDDESKEYOUTPUT` | Importación de clave DES firmada RSA. |
| `IMPORTRSAENCIPHEREDPKCS7KEY` / `IMPORTRSAENCIPHEREDPKCS7KEYOUTPUT` | Importación PKCS#7 RSA. |
| `IMPORTRSAENCIPHEREDPKCS7KEYEX` / `IMPORTRSAENCIPHEREDPKCS7KEYEXOUTPUT` | Importación PKCS#7 RSA extendida. |
| `EXPORTRSAISSUERSIGNEDITEM` / `EXPORTRSAISSUERSIGNEDITEMOUTPUT` | Exportación firmada por emisor. |
| `EXPORTRSAEPPSIGNEDITEM` / `EXPORTRSAEPPSIGNEDITEMOUTPUT` | Exportación firmada por EPP. |
| `KEY` / `KEYDETAIL` / `KEYDETAILEX` / `KEYDETAIL340` | Detalle de claves. |
| `KEYBLOCKINFO` | Información de bloque de clave. |
| `KCV` / `GENERATEKCV` | Código de verificación de clave. |
| `GENERATERSAKEYPAIR` | Generación de par de claves RSA. |
| `LOADCERTIFICATE` / `LOADCERTIFICATEOUTPUT` | Carga de certificado. |
| `LOADCERTIFICATEEX` / `LOADCERTIFICATEEXOUTPUT` | Carga de certificado extendida. |
| `GETCERTIFICATE` / `GETCERTIFICATEOUTPUT` | Obtención de certificado. |
| `REPLACECERTIFICATE` / `REPLACECERTIFICATEOUTPUT` | Reemplazo de certificado. |
| `AUTHENTICATE` / `AUTHENTICATEOUT` | Autenticación. |
| `STARTAUTHENTICATE` / `STARTAUTHENTICATEOUT` | Inicio de autenticación. |
| `STARTKEYEXCHANGE` | Inicio de intercambio de claves. |
| `EMVIMPORTPUBLICKEY` / `EMVIMPORTPUBLICKEYOUTPUT` | Importación de clave pública EMV. |
| `SECUREKEYENTRY` / `SECUREKEYENTRYOUT` | Entrada segura de clave. |
| `SECUREKEYDETAIL` | Detalle de clave segura. |
| `RESTKEYENCKEY` | Clave de cifrado de clave REST. |
| `HSMINFO` / `HSMINIT` / `HSMIDENTIFIER` / `HSMDETAIL` | Gestión de HSM. |
| `PCIPTSDEVICEID` | Identificador de dispositivo PCI PTS. |
| `ETSCAPS` / `SIGNERCAP` | Capacidades ETS / firmante. |
| `DIGEST` / `DIGESTOUTPUT` | Operaciones de hash. |
| `ENCIO` / `BANKSYSIO` | E/S cifrada / bancaria. |
| `SECMSG` | Mensaje seguro. |
| `PASSWORD` / `PASSWORDENTRY` / `PASSWORDENTRYOUT` | Gestión de contraseñas. |
| `GETLAYOUT` / `LAYOUT` / `FRAME` / `DATA` | Diseño del teclado. |
| `PRESENTIDC` / `PRESENTRESULT` / `PRESENTCLEAR` | Presentación de ID. |
| `MAINTAINPIN` | Mantenimiento de PIN. |
| `LOCALDES` / `LOCALVISA` / `LOCALBANKSYS` / `LOCALEUROCHEQUE` | Verificación local de PIN. |
| `HEXKEYS` / `FK` / `FDK` | Teclas hexadecimales y de función. |
| `ACCESS` / `ATTRIBUTES` / `INIT` | Control de acceso e inicialización. |
| `FUNCKEYDETAIL` | Detalle de tecla de función. |
| `CREATEOFFSET` | Creación de offset de PIN. |
| `DEVICEPOSITION` | Posición del dispositivo. |
| `POWERSAVECONTROL` / `POWERSAVECHANGE` | Gestión de ahorro de energía. |
| `SETGUIDLIGHT` | Control de luz de guía. |
| `SYNCHRONIZECOMMAND` | Sincronización de comandos. |

## Dependencias

| Dependencia | Origen | Propósito |
|---|---|---|
| `protobuf::libprotobuf` | vcpkg (`C:\Projects\vcpkg\installed\<plat>-windows-static`) | Runtime de serialización Protocol Buffers |
| STL | — | Soporte estándar C++17 |

Los headers de protobuf se tratan como externos (`/external:I`) con nivel de advertencias suprimido (`/W0`).

## Compilación

- **Tipo de salida:** Biblioteca estática (`.lib`)
- **Estándar C++:** C++17 (`/std:c++17`)
- **Plataformas:** Win32 / x64
- **Configuraciones:** Debug / Release
- **Toolset:** MSVC v143 (Visual Studio 2022)
- **Windows SDK:** 10.0
- **Sistema de build alternativo:** CMake 3.15+ (via `CMakeLists.txt`)



## Uso en proyectos consumidores

1. Añadir `../../externs/XfsProtos/inc` a los directorios de include adicionales.
2. Enlazar contra `../../externs/XfsProtos/lib/$(Platform)/$(Configuration)/XfsProtos.lib`.
3. Incluir el header del namespace correspondiente al servicio requerido.

```cpp
#include "iCdm.pb.h"   // Mensajes CDM

proto::cdm::DISPENSE dispense;
dispense.set_mixnumber(1);
// ... rellenar campos y serializar con SerializeToString()
```

```cpp
#include "iPin.pb.h"   // Mensajes PIN

proto::pin::GETPIN getPin;
getPin.set_maxlen(4);
// ... serializar para envío por PIPE/GRPC/...
```

## Posición en el árbol de compilación

```
protobuf (vcpkg) ──► XfsProtos ──► XfsObjectsCdm
                              ──► XfsObjectsCim
                              ──► XfsObjectsIpm
                              ──► XfsObjectsPin
                              ──► XfsObjects (tipos genéricos)
```
