# Billeteras {#sec-wallets}

## Descripción general

Una billetera SSI es software que almacena y protege el material que necesita un titular, emisor o verificador para actuar en un protocolo de identidad. En el [modelo de datos de credenciales verificables 2.0 de W3C](https://www.w3.org/TR/vc-data-model-2.0/), un repositorio de credenciales almacena las credenciales del titular y protege el acceso a ellas. En [DID Core de W3C](https://www.w3.org/TR/did-1.0/), un DID se resuelve en un documento DID que puede exponer métodos públicos de verificación y puntos de acceso de servicios. Las claves privadas, las credenciales, los registros DIDComm y el estado local de la billetera permanecen en el almacenamiento de la billetera, no en el documento DID.

Identus usa la palabra billetera en dos contextos habituales:

- Una billetera de Edge Agent se ejecuta dentro de una aplicación para el titular, como una aplicación de navegador, móvil o de escritorio. La aplicación incorpora un SDK de Identus Edge y proporciona almacenamiento mediante Pluto.
- Una billetera de Cloud Agent es un contenedor de billetera en el servidor al que se accede mediante la API REST de Cloud Agent. Un controlador o inquilino usa llamadas de API y webhooks en lugar de llamadas directas al SDK dentro del proceso.

No trates un registro distribuido, PRISM Node o un resolvedor DID como una copia de seguridad de la billetera. El estado público del DID puede ayudar a otras partes a resolver las claves del emisor y los puntos de acceso de servicios. No puede reconstruir las credenciales que conserva el titular, las claves privadas de DIDs de pares, el historial de mensajes DIDComm, los registros de conexión ni el estado de las presentaciones. Una billetera de producción necesita un diseño explícito de almacenamiento, cifrado, copias de seguridad y recuperación.

En el ejemplo de billetes de avión de esta sección, la aerolínea ejecuta Cloud Agent como emisor. El viajero usa una billetera de Edge Agent como titular. El personal de seguridad del aeropuerto puede usar Cloud Agent, un SDK de Edge u otra implementación de verificador, según el despliegue.

## Responsabilidades de la billetera

Una billetera tiene más tareas que almacenar credenciales. Administra el estado local necesario para continuar los flujos de protocolo tras reinicios de la aplicación, cambios de dispositivo e interrupciones de red.

La billetera almacena o referencia:

- Registros DID y metadatos DID.
- Claves privadas, semillas de billetera o referencias a secretos que usan las operaciones DID.
- Credenciales verificables recibidas de emisores.
- Registros de presentaciones creadas para verificadores.
- Mensajes DIDComm, registros de conexión y estado del protocolo.
- Configuración del mediador para la entrega sin conexión.
- Referencias al estado de credenciales y a esquemas que necesitan los flujos de verificación posteriores.

La billetera usa ese estado para ejecutar el protocolo. La billetera del titular recibe ofertas de credenciales, solicita el consentimiento del titular, almacena las credenciales emitidas, crea presentaciones y envía respuestas DIDComm. La billetera del emisor firma credenciales y mantiene registros de emisión. La billetera o el agente del verificador recibe presentaciones, comprueba las pruebas criptográficas, resuelve el material del emisor, comprueba el estado de las credenciales y aplica después la política del verificador.

Mantén separadas las capas de verificación. Una comprobación criptográfica determina si la presentación, la prueba de la credencial, la clave del emisor, la vinculación del titular y los datos de estado superan la verificación. Una comprobación de política determina si el verificador acepta ese emisor, tipo de credencial, esquema, sujeto, antigüedad o nivel de garantía para la acción de negocio.

## Billeteras de Edge Agent

Los SDK de Identus Edge Agent permiten que una aplicación ejecute las funciones de billetera y agente cerca del titular. Los repositorios actuales de Identus ofrecen:

- [SDK de TypeScript](https://github.com/hyperledger-identus/sdk-ts) para aplicaciones de navegador y Node.js. La [documentación actual del SDK de TypeScript](https://hyperledger-identus.github.io/docs/sdk-ts/docs/sdk/) indica el paquete `@hyperledger/identus-sdk` y señala el cambio de nombre desde `@hyperledger/identus-edge-agent-sdk`.
- [SDK de Swift](https://github.com/hyperledger-identus/sdk-swift) para plataformas Apple.
- [SDK de Kotlin Multiplatform](https://github.com/hyperledger-identus/sdk-kmp) para aplicaciones Android y JVM.

Los SDK usan el mismo vocabulario de arquitectura:

- Apollo proporciona operaciones criptográficas.
- Castor crea, administra y resuelve DIDs.
- Pollux administra operaciones de credenciales en los SDK de Swift y Kotlin. El SDK actual de TypeScript expone las funciones de credenciales mediante sus módulos y complementos.
- Mercury administra mensajes DIDComm v2 y las tareas relacionadas de mensajería segura.
- Pluto proporciona la interfaz de almacenamiento.
- Agent o EdgeAgent combina los componentes en una API de agente de nivel superior.

Empieza con Agent o EdgeAgent para el código de la aplicación. Las llamadas directas a Apollo, Castor, Mercury, Pollux o Pluto sirven cuando la aplicación necesita una operación de nivel inferior que la API del agente no expone. Por ejemplo, una billetera que implementa un protocolo DIDComm propio puede construir y empaquetar mensajes mediante Mercury y almacenar después los registros de protocolo resultantes mediante la misma capa de almacenamiento de la billetera.

### Almacenamiento Pluto

Pluto es el límite principal de implementación en una billetera de Edge Agent. Identus define la interfaz de almacenamiento, y la aplicación proporciona la implementación que corresponde a su plataforma y modelo de amenazas.

En una billetera de navegador, la implementación puede usar IndexedDB u otra base de datos del navegador. En una billetera móvil, puede usar la base de datos de la plataforma y la protección de claves mediante Keychain o Keystore. En una billetera de prueba en el servidor, un almacén en memoria sirve para una ejecución desechable. El almacenamiento de producción debe cifrar los datos sensibles en reposo, vincular las claves de cifrado al modelo de seguridad de la plataforma cuando sea posible y admitir copias de seguridad y restauración comprobadas.

El diseño del almacenamiento cambia el comportamiento de la billetera. Si la aplicación almacena credenciales pero pierde las claves privadas, el titular puede perder la capacidad de demostrar el control del DID que usó durante la emisión o presentación. Si la aplicación almacena claves pero pierde las credenciales, el titular aún puede controlar el DID pero ya no tiene la carga útil de la credencial. Si la aplicación pierde los registros de protocolo DIDComm, puede no reanudar los flujos de emisión, conexión o prueba después de un reinicio.

Trata los almacenes Pluto de la comunidad como dependencias que debes evaluar, no como opciones predeterminadas de Identus. Comprueba su mantenimiento, comportamiento de cifrado, compatibilidad con migraciones y compatibilidad con la versión del SDK de la aplicación antes de elegir uno.

## Billeteras de Cloud Agent

Cloud Agent se ejecuta en un servidor y expone operaciones de billetera mediante API REST y flujos basados en webhooks. El [README de Cloud Agent](https://github.com/hyperledger-identus/cloud-agent) lo describe como un agente en la nube basado en estándares de W3C y Aries que admite [DIDComm v2](https://identity.foundation/didcomm-messaging/spec/v2.1/), emisión, verificación y tenencia de credenciales, administración de secretos y varios inquilinos.

Una billetera de Cloud Agent sirve para servicios que necesitan operaciones de identidad en el servidor. Una aerolínea puede usar Cloud Agent para administrar DIDs del emisor, firmar credenciales de billetes, enviar ofertas y seguir el estado de emisión. Un verificador del aeropuerto puede usar Cloud Agent para solicitar presentaciones y recibir resultados de verificación. Una empresa puede ejecutar Cloud Agent en modo de varios inquilinos cuando cada cliente o departamento necesita una billetera separada.

La custodia de Cloud Agent tiene un límite de confianza distinto al de una billetera de Edge Agent. El inquilino o controlador del negocio interactúa con la billetera mediante una API. El operador del servicio controla la ejecución, la base de datos, el almacenamiento de secretos, la red y el acceso operativo. Una billetera administrada por el servicio puede servir para organizaciones y billeteras de usuarios delegadas, pero el despliegue debe definir quién puede administrar billeteras, exportar datos, rotar secretos, recuperar semillas e inspeccionar registros.

### Billeteras de varios inquilinos

En el modo de varios inquilinos de Cloud Agent, un administrador crea una billetera, crea una entidad para el inquilino y configura un método de autenticación para esa entidad. La [documentación de incorporación de inquilinos de Identus](https://identus.io/cloud-agent/docs/docusaurus/multitenancy/tenant-onboarding/) describe la billetera como el contenedor de los activos del inquilino, incluidos DIDs, credenciales y conexiones. Después de la configuración, Cloud Agent limita las llamadas de API del inquilino a esa billetera.

Este modelo sirve para billeteras administradas por el servicio. No elimina el riesgo de custodia. La API de administración, la autenticación del inquilino, los identificadores de billetera, las filas de la base de datos, las rutas de Vault y el enrutamiento de webhooks deben preservar la separación entre inquilinos.

### Material de claves de Cloud Agent

Identus Cloud Agent usa vías distintas de administración de claves para los DIDs PRISM y los DIDs de pares.

Para los DIDs PRISM administrados, Cloud Agent deriva el material de claves a partir de una semilla de billetera y una ruta de derivación. La [documentación de administración de DIDs](https://identus.io/documentation/develop/cloud-agent/did-management/) indica que Cloud Agent almacena la ruta de derivación en lugar del propio material de claves PRISM y reconstruye después el material de claves durante la ejecución a partir de la semilla y la ruta.

Para los DIDs de pares administrados, Cloud Agent genera el material de claves y lo almacena en el almacenamiento de secretos. Los DIDs de pares admiten actividades DIDComm, por lo que estas claves protegen las comunicaciones específicas de cada relación.

La [documentación de almacenamiento de secretos de Cloud Agent](https://identus.io/documentation/develop/cloud-agent/secrets-storage/) indica HashiCorp Vault como servicio predeterminado de almacenamiento de secretos. Describe tipos de secretos como semillas, claves privadas, datos de definiciones de credenciales y secretos de vinculación de AnonCreds. En la configuración de varios inquilinos, las rutas de activos de Vault incluyen el ID de la billetera para que los secretos de cada billetera tengan su propia ruta.

## Elegir la custodia Edge o Cloud

Usa una billetera de Edge Agent cuando el titular deba conservar las credenciales y claves en una aplicación bajo su control. Este modelo sirve para billeteras de viajeros, empleados o ciudadanos y para la presentación móvil de pruebas. El equipo de la aplicación debe administrar el almacenamiento, el cifrado, las copias de seguridad, la recuperación, el uso del mediador y el consentimiento local del usuario.

Usa una billetera de Cloud Agent cuando un servidor deba actuar por una organización o un inquilino delegado. Este modelo sirve para emisores, verificadores, billeteras de organizaciones administradas por un servicio y flujos de backend que necesitan API REST y webhooks. El equipo de despliegue debe administrar el almacenamiento de secretos, la autenticación, la separación entre inquilinos, la supervisión y los controles operativos.

Muchos sistemas Identus usan ambos modelos. El emisor y el verificador ejecutan Cloud Agent. El titular usa una billetera de Edge Agent. DIDComm, las credenciales, los DIDs y los protocolos de presentación conectan ambos modelos de custodia sin exigir la misma implementación de billetera en cada parte.

::: {.content-visible when-format="html:js"}
{{< include ../diagrams/d08-wallet-custody.html >}}
:::

::: {.content-visible unless-format="html:js"}
![El equipo de la aplicación implementa el almacenamiento, la protección de claves y la recuperación para la billetera del viajero. El operador del servicio administra la ejecución de Cloud Agent, la base de datos, el almacenamiento de secretos y la recuperación. El enlace DIDComm representa el intercambio de protocolo; la entrega puede usar un mediador.](../diagrams/d08-wallet-custody.svg){fig-alt="Custodia de billeteras Edge y Cloud"}
:::

## Flujo del billete de avión

El ejemplo de billetes de avión usa el límite de la billetera de forma concreta:

1. La aerolínea crea o selecciona una billetera de Cloud Agent para las operaciones de emisión.
2. La aerolínea crea un `did:prism` del emisor con un método de verificación `assertionMethod` y publica el DID cuando los verificadores necesitan una resolución respaldada por el registro distribuido.
3. El viajero abre una billetera de Edge Agent. La billetera crea o selecciona DIDs del titular y almacena el material secreto local mediante Pluto.
4. La aerolínea envía una oferta de credencial de billete a la billetera del viajero mediante DIDComm u otra vía de emisión compatible.
5. La billetera del viajero muestra la oferta, registra el consentimiento del titular, solicita la credencial, la recibe y la almacena.
6. El personal de seguridad del aeropuerto solicita pruebas del estado del billete, el vuelo, la vinculación del pasajero u otro conjunto de afirmaciones aceptado.
7. La billetera del viajero crea una presentación a partir de la credencial almacenada y la envía al verificador.
8. El software del verificador comprueba la prueba de la presentación, la resolución del DID del emisor, el estado de la credencial, la vinculación del titular, el esquema y las reglas específicas del formato.
9. La política de seguridad del aeropuerto decide si el resultado verificado satisface la regla del puesto de control.

Los pasos 8 y 9 son decisiones separadas. El software del verificador puede informar de que la prueba de la credencial es válida, el DID del emisor se resolvió y la credencial está activa. La política del aeropuerto aún puede rechazar la presentación si la fecha del vuelo, el aeropuerto, el emisor, el tipo de credencial o la vinculación del pasajero no corresponde a la regla del puesto de control.

## Notas de implementación

Para billeteras de Edge Agent:

- Elige el SDK para la plataforma anfitriona antes de diseñar el almacenamiento.
- Trata Pluto como un componente obligatorio de la billetera, no como una caché opcional.
- Cifra el contenido de la billetera que incluya credenciales, registros DIDComm, semillas, claves privadas o referencias a secretos.
- Comprueba la restauración en un dispositivo o perfil de navegador limpio.
- Usa `did:peer` para la actividad DIDComm específica de cada relación y `did:prism` para la resolución pública.

Para billeteras de Cloud Agent:

- Decide si el despliegue usa una billetera predeterminada o varios inquilinos antes de incorporar usuarios.
- Mantén separadas las credenciales de administración y las de los inquilinos.
- Guarda las semillas de billetera y el material de claves privadas en el almacenamiento de secretos configurado.
- Crea copias de seguridad de la base de datos de Cloud Agent y del almacenamiento de secretos como una sola unidad de recuperación.
- Audita la entrega de webhooks, las claves de API de los inquilinos, los IDs de billetera y las rutas de Vault durante la incorporación de inquilinos.

Un equipo completa el trabajo de billetera solo después de comprobar la restauración y la reanudación del protocolo. Una billetera que puede emitir o recibir credenciales durante una demostración del flujo normal aún puede fallar en producción si el titular cambia de dispositivo, borra el almacenamiento del navegador, crece la cola del mediador o un inquilino de Cloud Agent necesita recuperarse después de restaurar la base de datos.
