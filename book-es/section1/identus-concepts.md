# Conceptos de Identus {#sec-identus-concepts}

::: {.content-visible when-format="html:js"}
{{< include ../diagrams/d04-identus-components.html >}}
:::

::: {.content-visible unless-format="html:js"}
![El controlador ejecuta la lógica de negocio mediante la API de Cloud Agent. La billetera intercambia mensajes DIDComm con Cloud Agent y obtiene los mensajes entrantes del mediador. PRISM Node publica operaciones DID y resuelve el estado de los DIDs mediante el registro.](../diagrams/d04-identus-components.svg){fig-alt="Componentes de Identus y vías de comunicación"}
:::

Identus reúne varios componentes de código abierto. Puedes usar cada componente por separado o crear tu propia versión, aunque sus autores los diseñaron para que funcionen juntos.

## PRISM Node

PRISM Node implementa el método `did:prism` y actúa como un nodo de segunda capa para el libro mayor distribuido. Actualmente solo admite la cadena de bloques Cardano o una base de datos local, pero en el futuro actuará como una interfaz integral para varios [VDR (registros de datos verificables)](../glossary.md#vdr). El nodo puede resolver DIDs PRISM y escribir transacciones en una cadena de bloques o una base de datos. Mantiene un estado interno indexado que sincroniza con la cadena de bloques subyacente para realizar búsquedas eficientes.

PRISM Node constituye un componente esencial del ecosistema Identus y ofrece una plataforma segura y confiable para almacenar y administrar identificadores descentralizados. Gestiona la creación, la actualización, la resolución y la desactivación de DIDs PRISM: genera transacciones con la información de operación necesaria, verifica y valida estas operaciones, y las publica en la cadena de bloques. Cuando la cadena confirma las transacciones, el nodo actualiza su estado interno.

La arquitectura de PRISM Node permite que los usuarios:

- Creen DIDs sin publicar mediante el bloque de construcción Apollo, con la opción de anunciarlos públicamente más adelante. No todos los DIDs necesitan publicación en un VDR. Por ejemplo, un titular puede mantener su DID fuera del ámbito público, pero Cloud Agent exige que el emisor publique y ancle su DID de emisión en un VDR. PRISM Node realiza esa operación.
- Actualicen documentos DID mediante operaciones de actualización en la cadena.
- Desactiven DIDs mediante operaciones de desactivación en la cadena.
- Resuelvan DIDs mediante consultas a los cambios históricos en la cadena.
- Consulten el estado de las operaciones enviadas al nodo.

Este enfoque de segunda capa permite que los DIDs sean escalables y eficientes. Ofrece procesamiento y almacenamiento fuera de la cadena y usa la seguridad y la inmutabilidad de la cadena de bloques subyacente. PRISM Node debe permanecer en línea para prestar un servicio confiable.

## Cloud Agent

Los autores escribieron Cloud Agent en Scala. Se ejecuta en un servidor y se comunica con clientes y pares mediante una API REST. Constituye un componente esencial de una aplicación Identus: puede administrar billeteras de identidad y sus operaciones, además de emitir credenciales verificables. Cloud Agent debe permanecer en línea.

Cloud Agent ofrece servicios completos de identidad autosoberana con un diseño escalable, robusto y conforme a los estándares. Admite los estándares del W3C, DIDCommV2 y los protocolos de [Hyperledger Aries](https://www.lfdecentralizedtrust.org/projects/aries) para facilitar la interoperabilidad en el ecosistema SSI. Sus principales capacidades incluyen:

- Compatibilidad con varios roles de agente: emisor, titular y verificador.
- Administración de credenciales verificables del estándar W3C (formatos JSON y JSON-LD codificados como JWT), SD-JWT y AnonCreds.
- Implementación de DIF Presentation Exchange para solicitudes y envío de credenciales.
- Compatibilidad con los métodos DID `did:prism` y `did:peer` (versión 2).
- Implementación completa de la mensajería y los protocolos DIDCommV2.
- Compatibilidad con las RFC de [Hyperledger Aries](https://www.lfdecentralizedtrust.org/projects/aries), incluidos el intercambio de DIDs, el protocolo fuera de banda, la emisión de credenciales y la presentación de pruebas.

La API REST de Cloud Agent permite que los desarrolladores creen controladores en cualquier lenguaje de programación sin conocer en profundidad los estándares SSI subyacentes. Esta arquitectura separa la lógica de negocio de la infraestructura de identidad. Así facilita el desarrollo de aplicaciones especializadas que usan las capacidades de la identidad descentralizada.

En un despliegue, Cloud Agent se comunica con PRISM Node mediante el protocolo gRPC. Lo usa como registro de datos verificables para anclar DIDs en un libro mayor distribuido y ofrecer alta seguridad y disponibilidad.

## Bloques de construcción

Identus divide las operaciones SSI importantes en bibliotecas especializadas que llama «bloques de construcción». Puedes combinar y configurar estos componentes modulares para distintos casos de uso y requisitos de producto. Esta arquitectura ofrece flexibilidad y opciones de personalización para implementar soluciones de identidad descentralizada que respondan a tus necesidades.

### Apollo: criptografía

Apollo reúne primitivas criptográficas que Identus usa para garantizar la integridad, la autenticidad y la confidencialidad de los datos. Ofrece la base para la comunicación segura y la protección de datos en el ecosistema Identus.

Apollo usa funciones hash criptográficas para crear huellas digitales de los datos y detectar modificaciones no autorizadas. Por ejemplo, cuando un verificador recibe una credencial, las funciones criptográficas de Apollo pueden comprobar que nadie la haya modificado desde su emisión.

Apollo también implementa firmas digitales para autenticar la identidad de remitentes y destinatarios, y usa algoritmos de cifrado para proteger los datos sensibles frente a accesos no autorizados. En un contexto sanitario, el cifrado de Apollo mantendría la confidencialidad de las credenciales de los pacientes cuando las comparten las partes autorizadas.

### Castor: DID

Castor permite crear, administrar y resolver identificadores descentralizados (DIDs). Actualmente admite el método nativo `did:prism` y el método `did:peer`. El equipo debate una arquitectura más flexible que permitiría que cualquier persona escribiera complementos para admitir otros métodos DID.

Cuando un usuario crea una identidad digital en una aplicación Identus, Castor genera el DID y el material criptográfico asociado. Por ejemplo, una universidad que emite credenciales de estudiantes usaría Castor para crear y administrar DIDs de la institución y, posiblemente, de cada estudiante, con lo que establecería la base de relaciones digitales de confianza.

El componente resolvedor de Castor puede buscar un DID y obtener su documento DID asociado. Ese documento contiene las claves públicas, los mecanismos de autenticación y los puntos de acceso de servicios necesarios para interactuar de forma segura con esa identidad.

### Pollux: credenciales verificables

Pollux gestiona todas las operaciones de credenciales verificables. Permite que los usuarios emitan, administren y verifiquen credenciales de forma que protejan la privacidad. Este bloque de construcción implementa la funcionalidad central del intercambio de credenciales en los sistemas de identidad autosoberana.

En un contexto laboral, una empresa podría usar Pollux para emitir credenciales de empleados que estos guardarían en sus billeteras digitales. Al solicitar un préstamo, un empleado podría revelar de forma selectiva la información laboral pertinente de esa credencial, sin revelar datos personales innecesarios. El banco podría usar las funciones de verificación de Pollux para confirmar la autenticidad y la validez de la credencial sin contactar directamente con el empleador.

Pollux también gestiona el estado de las credenciales. Permite que los emisores revoquen credenciales cuando sea necesario y que los verificadores comprueben si una credencial sigue siendo válida antes de aceptarla.

### Mercury: DIDComm

Mercury ofrece una interfaz al protocolo [DIDCommV2](https://www.didcomm.org) que permite la comunicación segura y privada entre DIDs con independencia del mecanismo de transporte subyacente. Este bloque de construcción establece la base de la comunicación entre agentes en el ecosistema Identus.

Por ejemplo, cuando un ciudadano quiere compartir una credencial emitida por el gobierno con un proveedor de servicios, Mercury facilita el intercambio de mensajes cifrados y autenticados entre la billetera del ciudadano y el sistema de verificación del proveedor. Ambos se comunican entre pares y prescinden de intermediarios centralizados para ese intercambio.

El diseño de Mercury no depende del transporte. Estas comunicaciones seguras pueden usar distintos canales, incluidos HTTP, WebSockets y Bluetooth. Así permite distintos despliegues, desde aplicaciones web hasta dispositivos móviles.

La [documentación de Identus](https://hyperledger-identus.github.io/docs/home/identus/cloud-agent/building-blocks) ofrece más información sobre cada bloque de construcción.

## Edge Agent

Los Edge Agents ofrecen capacidades de agente a las aplicaciones cliente, como sitios web, aplicaciones móviles y otro software orientado al usuario. Los Edge Agents pueden perder la conexión, por lo que envían y reciben todas las comunicaciones mediante un proxy en línea llamado mediador.

Los desarrolladores pueden integrar los SDK de Edge Agent en sus aplicaciones para acceder a un conjunto completo de funciones SSI en el cliente. Estos SDK gestionan operaciones esenciales, como:

- Crear y administrar DIDs.
- Guardar y administrar credenciales verificables.
- Comunicarse de forma segura mediante DIDComm.
- Realizar operaciones criptográficas de firma y verificación.
- Guardar datos de identidad de forma segura en el dispositivo.

Identus ofrece SDK de Edge Agent en varios lenguajes para distintas plataformas:

- **SDK de TypeScript**: para aplicaciones web.
- **SDK de Swift**: para aplicaciones iOS.
- **SDK de Kotlin Multiplatform**: para aplicaciones Android y desarrollo multiplataforma.

Cada SDK implementa los mismos bloques de construcción centrales que Cloud Agent: las interfaces de Apollo, Castor, Mercury, Pollux y Pluto. Esto ofrece funciones coherentes e interoperabilidad en todo el ecosistema Identus. Esta coherencia arquitectónica permite que los desarrolladores creen experiencias fluidas en las que las operaciones de identidad se ejecuten en el cliente o en el servidor, según el caso de uso.

Los Edge Agents suelen funcionar como billeteras digitales. Permiten que los usuarios mantengan el control de sus credenciales y sus datos de identidad en sus dispositivos personales. Este enfoque sigue los principios centrales de la identidad autosoberana: conserva el control de los datos en manos del usuario y reduce la dependencia de servicios centralizados.

## Mediador

Los mediadores actúan como intermediarios entre DIDs de pares. Para que un agente envíe un mensaje a otro, debe conocer los DIDs `to` y `from` de cada mensaje. El remitente y el destinatario forman una conexión criptográfica llamada `DIDPair`. Los mediadores mantienen colas de mensajes para cada `DIDPair`. Si un Edge Agent está sin conexión, el mediador guarda sus mensajes entrantes hasta que el agente vuelve a conectarse y puede recibirlos. Los mediadores pueden entregar mensajes cuando el agente los consulta o enviarlos mediante WebSockets. Deben permanecer en línea y ofrecer alta disponibilidad.

Identus admite de forma oficial su propia implementación de mediador y publica actualizaciones para ella. También existen implementaciones alternativas cercanas a la comunidad Identus. En teoría, cualquier mediador compatible con DIDCommV2 debería funcionar con Identus, por lo que esta lista no pretende incluir todas las opciones.

- [PRISM Mediator](https://github.com/hyperledger-identus/mediator).
- [RootsID Mediator](https://github.com/roots-id/didcomm-mediator).
- [Blocktrust Mediator](https://github.com/bsandmann/blocktrust.Mediator).

RootsID y Blocktrust ofrecen instancias alojadas de sus mediadores con acceso público. Aunque resultan útiles durante el desarrollo, no recomendamos usarlas para despliegues Identus en producción: carecen de garantías de disponibilidad y no escalarán más allá de un pequeño número de usuarios simultáneos. El [capítulo @sec-mediator] explica cómo ejecutar tu propio mediador.

## Registro de datos verificables (VDR)

El registro de datos verificables (Verifiable Data Registry, VDR) aborda el reto de garantizar la autenticidad y la integridad de datos públicos en un entorno que genera y consume datos a una escala sin precedentes. Los sistemas tradicionales suelen depender de autoridades centralizadas. Esa dependencia introduce puntos únicos de fallo y debilita la confianza, en especial cuando varias partes independientes necesitan confiar en los mismos datos.

Los retos principales incluyen permitir la verificación descentralizada, garantizar la integridad y la autenticidad sin supervisión central, ofrecer interoperabilidad entre distintas tecnologías de almacenamiento (bases de datos, cadenas de bloques y cachés en memoria) y facilitar una colaboración escalable sin exigir confianza previa entre las partes.

El sistema VDR aborda estos problemas mediante una API unificada y un marco modular y extensible para almacenar, modificar, obtener y eliminar datos. Separa la capa de aplicación de los mecanismos de almacenamiento mediante dos componentes principales que admiten complementos:

- **Drivers**: son complementos que implementan la lógica real de almacenamiento y obtención de datos. Cada driver se adapta a un backend de almacenamiento concreto, como una base de datos, una cadena de bloques o almacenamiento en memoria. Los drivers ejecutan operaciones de creación, lectura, actualización y eliminación. También generan y verifican pruebas criptográficas (hashes para datos inmutables y firmas digitales para datos modificables) para garantizar la integridad.
- **Administradores de URL**: construyen y resuelven URL que hacen referencia a los datos almacenados. Incluyen los metadatos necesarios, como identificadores de drivers, familias de drivers y pruebas criptográficas, mediante parámetros de consulta estándar. Así, cuando recibe una URL, el sistema VDR puede seleccionar el driver correcto y verificar la integridad de los datos.

Esta abstracción permite que la capa VDR mantenga operaciones de datos coherentes y verificables con independencia de dónde o cómo guarde los datos. Emplea hashes criptográficos para datos inmutables y firmas digitales para datos modificables, incluidos en las URL o disponibles mediante el driver, para garantizar la integridad. Así, varias partes pueden verificar por sí mismas la autenticidad y la integridad sin confiar previamente unas en otras, lo que mejora la seguridad y la colaboración. El diseño del sistema VDR permite adaptarse a distintos backends de almacenamiento y mantener métodos coherentes de verificación y acceso a los datos.
