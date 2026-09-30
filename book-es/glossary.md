# Glosario {#sec-glossary .unnumbered}

Este glosario reúne los términos de identidad autosoberana (SSI), identidad descentralizada y específicos de Identus que usa el libro. Las definiciones proporcionan al desarrollador información suficiente para seguir los capítulos, con referencias a los estándares que definen formalmente cada término. Los términos aparecen en orden alfabético.

[ADA]{#ada}

: La criptomoneda nativa de la cadena de bloques [Cardano](#cardano). PRISM Node gasta ADA en comisiones cuando ancla operaciones `did:prism` en la cadena: ADA de prueba en redes como `preprod` y ADA real en `mainnet`.

[Afirmación]{#claim}

: Una declaración o atributo individual sobre un [sujeto](#subject) dentro de una credencial, por ejemplo un nombre, fecha de nacimiento o categoría de licencia. Una [credencial verificable](#verifiable-credential) es un conjunto de afirmaciones de un [emisor](#issuer).

[Almacenamiento de secretos]{#secret-storage}

: El almacén configurable de Cloud Agent para semillas de billetera y claves privadas. Un backend `postgres` guarda secretos sin cifrar y sirve solo para laboratorio. Se recomienda [HashiCorp Vault](#hashicorp-vault) para producción.

[Anclaje en el registro distribuido]{#ledger-anchoring}

: Registrar operaciones DID como transacciones en una cadena de bloques, como [Cardano](#cardano), para que las alteraciones sean detectables y permitan resolución pública.

[AnonCreds (credenciales anónimas)]{#anoncreds}

: Un formato de [credencial verificable](#verifiable-credential) que preserva la privacidad y procede del ecosistema Hyperledger Indy. Usa [pruebas de conocimiento cero](#zero-knowledge-proof) y [pruebas de predicados](#predicate-proof) para que un titular demuestre hechos, como «edad superior a 21», sin revelar los valores originales de los atributos. Identus Cloud Agent lo admite.

[Apollo]{#apollo}

: El [componente básico](#building-blocks) de Identus que proporciona primitivas criptográficas —hashing, firmas digitales, cifrado y generación de claves— usadas en toda la plataforma para garantizar integridad, autenticidad y confidencialidad de datos.

[AppRole]{#approle}

: Un método de autenticación de [HashiCorp Vault](#hashicorp-vault), formado por un ID de rol y un ID secreto, que se prefiere a un token raíz estático para despliegues automáticos de producción de Cloud Agent.

[Aries (Hyperledger Aries)]{#aries}

: Una familia de protocolos de interoperabilidad Hyperledger (RFC) para agentes SSI que cubre intercambio de DIDs, invitaciones fuera de banda y flujos de emisión de credenciales y presentación de pruebas. Identus Cloud Agent busca la compatibilidad con W3C y Aries.

[assertionMethod]{#assertion-method}

: Una [relación de verificación](#verification-relationship) en un [documento DID](#did-document) que enumera las claves aprobadas para hacer afirmaciones, incluida la firma o emisión de credenciales verificables. El verificador comprueba que la clave firmante de la credencial aparece aquí.

[authentication]{#authentication}

: Una [relación de verificación](#verification-relationship) en un [documento DID](#did-document) que enumera las claves aprobadas para demostrar el control del DID mediante autenticación de desafío y respuesta.

[Autoridad de certificación (CA)]{#certificate-authority}

: El modelo centralizado tradicional de confianza digital, donde una autoridad central garantiza identidades. El libro lo compara con un [registro de confianza](#trust-registry), que registra la autoridad dentro de un ecosistema con gobernanza, pero no emite ni conserva credenciales.

[Autoridad de gobernanza]{#governance-authority}

: El organismo que define las reglas de un [ecosistema](#ecosystem) y mantiene o supervisa su [registro de confianza](#trust-registry).

[Billetera]{#wallet}

: En SSI, software que almacena y protege los DIDs, claves, [conexiones](#connection) y credenciales de una parte, y crea presentaciones. En Identus Cloud Agent, también es la unidad de aislamiento del [inquilino](#tenant). La billetera del titular suele ser un [Edge Agent](#edge-agent); una billetera de servidor se ejecuta en [Cloud Agent](#cloud-agent). Es distinta de [Cardano Wallet](#cardano-wallet).

[Cardano]{#cardano}

: La cadena de bloques pública que Identus usa como [registro de datos verificables](#vdr) de producción. Ancla operaciones `did:prism` para que cualquiera pueda resolver un DID PRISM publicado. El anclaje requiere [ADA](#ada) para pagar comisiones.

[Cardano DB Sync]{#cardano-db-sync}

: Un componente Cardano que sigue la cadena e indexa bloques en PostgreSQL para que PRISM Node lea datos del registro distribuido de forma eficiente. Normalmente consume más recursos que el resto de un entorno Cardano alojado por el propio operador.

[Cardano Wallet]{#cardano-wallet}

: El servicio del ecosistema Cardano que PRISM Node usa para administrar fondos y enviar transacciones Cardano al publicar operaciones DID. Es distinto de una [billetera](#wallet) SSI.

[Castor]{#castor}

: El [componente básico](#building-blocks) de Identus que crea, administra y resuelve [identificadores descentralizados](#did). Actualmente admite los métodos `did:prism` y `did:peer`.

[Clave de API]{#api-key}

: Una credencial por [inquilino](#tenant), enviada en la cabecera `apikey`, que limita las llamadas de API de Cloud Agent a la [billetera](#wallet) de ese inquilino. Las operaciones privilegiadas de administración usan un token de administrador.

[Claves de destinatario]{#recipient-keys}

: Las claves que una billetera registra con un [mediador](#mediator) para que sepa qué mensajes entrantes debe encolar para ese destinatario.

[Cloud Agent]{#cloud-agent}

: El agente de Identus en el servidor, escrito en Scala, que administra billeteras, DIDs y mensajería DIDComm, y puede emitir, conservar y verificar credenciales verificables. Una aplicación [controladora](#controller) lo dirige mediante su API REST. Se espera que esté siempre conectado.

[cnf (clave de confirmación)]{#cnf}

: Un elemento de vinculación de clave incorporado en una credencial [SD-JWT](#sd-jwt). Vincula la credencial a una clave del titular para que este demuestre después su control al firmar el [desafío](#challenge) y el [dominio](#domain) del verificador.

[Componentes básicos]{#building-blocks}

: El conjunto de bibliotecas modulares especializadas de Identus que administran un área de funciones SSI cada una: [Apollo](#apollo), criptografía; [Castor](#castor), DIDs; [Pollux](#pollux), credenciales; [Mercury](#mercury), DIDComm; y [Pluto](#pluto), almacenamiento. Cloud Agent y los SDK de Edge Agent implementan los mismos componentes básicos.

[Conexión]{#connection}

: Una relación [DIDComm](#didcomm) establecida, segura y continua entre dos agentes. Normalmente usa [DIDs de pares](#peer-did) y cada parte la mantiene como registro de conexión en su billetera.

[Controlador (aplicación)]{#controller}

: La aplicación que dirige un [Cloud Agent](#cloud-agent) mediante su API REST y callbacks webhook. El controlador contiene la lógica de negocio y el agente administra los estándares SSI. No debe confundirse con un [controlador DID](#did-controller).

[Controlador DID]{#did-controller}

: La entidad autorizada según las reglas del método DID para cambiar un [documento DID](#did-document): rotar claves, añadir servicios o desactivar. Puede ser el mismo [sujeto DID](#did-subject) u otra entidad que actúe en su nombre.

[Coordinate Mediation]{#coordinate-mediation}

: El protocolo DIDComm `coordinate-mediation/2.0` que una billetera usa para solicitar [mediación](#mediation) a un [mediador](#mediator) y registrar las [claves de destinatario](#recipient-keys) para las que debe aceptar mensajes.

[Correlación]{#correlation}

: La vinculación de la actividad de una persona u organización entre contextos mediante un identificador compartido o una clave reutilizada. Los DIDs públicos y el material reutilizado de documentos DID crean riesgo de correlación; los [DIDs específicos de cada relación](#pairwise-did) lo reducen.

[COSE]{#cose}

: La familia de estándares criptográficos basada en CBOR para firma y cifrado, equivalente binario de [JOSE](#jose), que referencia la recomendación W3C VC «Securing with JOSE/COSE».

[Credencial verificable (VC)]{#verifiable-credential}

: Un conjunto de [afirmaciones](#claim) de un [emisor](#issuer), protegido con un mecanismo criptográfico para que el software detecte alteraciones y confirme la autoría. El modelo de datos de credenciales verificables de [W3C](#w3c) la define. El modelo de datos es independiente del formato: [JWT-VC](#jwt-vc), [SD-JWT-VC](#sd-jwt) o [AnonCreds](#anoncreds).

[credentialStatus]{#credential-status}

: La propiedad de una credencial revocable que apunta a su entrada en la [lista de estado](#status-list). Permite que el verificador descubra si está [revocada o suspendida](#revocation). El modelo de datos VC de W3C la define.

[DID de pares]{#peer-did}

: Un DID del método `did:peer` que las partes de una relación comparten de forma privada y no publican en ningún registro público. Identus lo usa para conexiones [DIDComm](#didcomm) y [mediación](#mediation), donde los identificadores específicos de cada relación reducen la correlación.

[DID del emisor]{#issuer-did}

: Un DID bajo el control de un [emisor](#issuer) que usa para firmar sus credenciales, de modo que los verificadores resuelvan sus claves y comprueben la autoría. Normalmente es público, como los [DIDs PRISM](#prism-did).

[DID específico de una relación]{#pairwise-did}

: Un DID único creado para una sola relación, de modo que cada relación use un identificador distinto con seudónimo. Reduce la [correlación](#correlation) entre contextos. Los [DIDs de pares](#peer-did) suelen usarse de esta forma.

[DID PRISM]{#prism-did}

: Un DID que usa el método `did:prism` de Identus y cuyas operaciones de creación, actualización y desactivación se anclan al registro [Cardano](#cardano) mediante [PRISM Node](#prism-node). Sirve para identidades públicas de emisores y verificadores que necesitan resolución respaldada por el registro distribuido. Consulta también [forma larga / forma corta](#long-short-form).

[DID publicado]{#published-did}

: Un DID cuyas operaciones se han anclado al registro distribuido para que cualquier parte lo resuelva. Se diferencia de un DID no publicado: por ejemplo, normalmente el titular no necesita publicarlo y el emisor sí.

[`did:peer`]{#did-peer}

: Consulta [DID de pares](#peer-did).

[`did:prism`]{#did-prism}

: Consulta [DID PRISM](#prism-did).

[DIDComm (DIDComm Messaging V2)]{#didcomm}

: Un protocolo de mensajería seguro, cifrado e independiente del transporte para comunicar agentes identificados por DIDs. DIDComm V2, la versión de Identus, oculta el contenido a todos salvo los destinatarios autorizados, demuestra el remitente en modo autenticado y funciona mediante cualquier transporte, como HTTP, WebSockets o Bluetooth.

[DIDPair]{#didpair}

: El emparejamiento criptográfico de un DID remitente y un DID destinatario que juntos identifican un canal DIDComm. Un [mediador](#mediator) mantiene una cola de mensajes por DIDPair.

[DIF (Decentralized Identity Foundation)]{#dif}

: Una organización del sector que desarrolla estándares de identidad descentralizada, incluidos DIDComm y Presentation Exchange.

[DIF Presentation Exchange]{#presentation-exchange}

: Una especificación DIF para expresar qué prueba requiere un verificador y cómo responde la billetera del titular. Identus Cloud Agent la implementa para solicitudes y entregas de credenciales.

[Divulgación (SD-JWT)]{#disclosure}

: En una credencial [SD-JWT](#sd-jwt), el valor de afirmación con sal que el titular revela durante la presentación para que el verificador calcule su hash y lo compare con el resumen firmado en la credencial. Las afirmaciones no divulgadas permanecen ocultas.

[Divulgación selectiva]{#selective-disclosure}

: La capacidad del titular de revelar solo las afirmaciones que necesita el verificador, en lugar de toda la credencial. Por ejemplo, demostrar «más de 21» sin revelar toda la fecha de nacimiento. Depende del formato: [SD-JWT](#sd-jwt) y [AnonCreds](#anoncreds) la admiten; [JWT-VC](#jwt-vc) simple no. Apoya la minimización de datos.

[Documento DID]{#did-document}

: El documento en el que se resuelve un DID. Publica el material público necesario para interactuar con el sujeto DID: [métodos de verificación](#verification-method), [relaciones de verificación](#verification-relationship) y [puntos de acceso de servicios](#service-endpoint). Nunca debe contener claves privadas, secretos o un perfil personal.

[Ecosistema]{#ecosystem}

: Una comunidad de participantes que opera según un [marco de gobernanza](#governance-framework) compartido, donde se definen relaciones de confianza y autorizaciones. Un [registro de confianza](#trust-registry) responde preguntas sobre la autoridad en un ecosistema específico.

[Ed25519]{#ed25519}

: Un algoritmo de firma digital y tipo de clave de curva elíptica de uso común para autenticación y firma de credenciales. Por ejemplo, Identus lo requiere para las claves del emisor y titular de SD-JWT.

[Edge Agent]{#edge-agent}

: Un agente Identus que se ejecuta dentro de una aplicación para usuarios, web, móvil o de escritorio, implementado como [SDK](#edge-sdk). A diferencia de [Cloud Agent](#cloud-agent), no se puede suponer que siempre está conectado y depende de un [mediador](#mediator) para enviar y recibir mensajes. Normalmente actúa como [billetera](#wallet) del usuario.

[Edge SDK]{#edge-sdk}

: Las bibliotecas cliente Identus, disponibles en TypeScript, Swift y Kotlin Multiplatform, que incorporan funciones de [Edge Agent](#edge-agent) en una aplicación. Implementan los mismos [componentes básicos](#building-blocks) que Cloud Agent.

[Emisión de credenciales]{#credential-issuance}

: El flujo de protocolo por el que un emisor crea, firma y entrega una [credencial verificable](#verifiable-credential) en la billetera del titular. En Identus sigue el protocolo Issue Credential mediante DIDComm, u [OpenID4VCI](#openid4vci).

[Emisor]{#issuer}

: El rol que hace [afirmaciones](#claim) sobre un [sujeto](#subject), las empaqueta como [credencial verificable](#verifiable-credential) y la firma criptográficamente. Su autoridad procede del contexto: ley, acreditación, contrato o gobernanza.

[Emisores de confianza]{#trusted-issuers}

: El conjunto de [DIDs de emisores](#issuer-did), normalmente limitado a un [esquema de credencial](#credential-schema), que un verificador acepta como autoridades. En Identus se configura actualmente en una [política de verificación](#verification-policy). Un [registro de confianza](#trust-registry) generaliza este concepto.

[Entidad]{#entity}

: En Cloud Agent, el objeto administrativo que representa a un [inquilino](#tenant) y se vincula a una [billetera](#wallet). La autenticación, mediante [clave de API](#api-key) o permiso Keycloak, se asocia a una entidad. En modo de un solo inquilino, usa automáticamente una entidad predeterminada con ID de ceros.

[Esquema de credencial]{#credential-schema}

: Una definición registrada y compartida de la estructura de un tipo de credencial: campos, tipos y atributos obligatorios. Los emisores emiten conforme a ella, y los verificadores y sus [políticas de verificación](#verification-policy) se limitan a ella. Identus admite JSON Schema para JWT y SD-JWT y esquemas AnonCreds.

[Forma larga / forma corta de DID PRISM]{#long-short-form}

: Dos codificaciones de un [DID PRISM](#prism-did). La forma larga incorpora todo el estado inicial del DID en el identificador y se puede resolver antes de publicarlo. La forma corta es el identificador compacto que se usa después de [publicarlo](#published-did) en el registro distribuido.

[HashiCorp Vault]{#hashicorp-vault}

: Un servicio de administración de secretos que Cloud Agent puede usar como backend de [almacenamiento de secretos](#secret-storage) de producción para semillas de billetera y material de claves, con rutas por billetera en despliegues de varios inquilinos.

[Hyperledger Indy]{#hyperledger-indy}

: Un ecosistema Hyperledger de cadena de bloques y SSI del que procede el formato de credencial [AnonCreds](#anoncreds).

[Identidad autosoberana (SSI)]{#ssi}

: Un modelo de identidad digital basado en credenciales bajo custodia del usuario y pruebas criptográficas, en lugar de un proveedor central de identidad. Un sujeto recibe una credencial de un [emisor](#issuer), la almacena como [titular](#holder) y presenta pruebas a un [verificador](#verifier) que aplica su propia política. SSI cambia la custodia y el intercambio de datos de identidad; la gobernanza, la ley y la reputación aún determinan la confianza.

[Identificador descentralizado (DID)]{#did}

: Un identificador único global que una entidad controla sin un registrador central. La especificación DID Core de [W3C](#w3c) lo define como un URI de tres partes: el esquema `did:`, un nombre de [método](#did-method) y un identificador específico del método. Se resuelve en un [documento DID](#did-document).

[Identus (Hyperledger Identus)]{#identus}

: La plataforma de identidad descentralizada de código abierto que explica este libro, alojada bajo el marco de confianza descentralizada de Linux Foundation. Proporciona [Cloud Agent](#cloud-agent), [PRISM Node](#prism-node), [SDK de Edge](#edge-sdk) y [Mediator](#mediator) para emitir, conservar y verificar credenciales.

[Inquilino]{#tenant}

: Un usuario lógico u organización que atiende [Cloud Agent](#cloud-agent), representado por una [entidad](#entity) y respaldado por una [billetera](#wallet) aislada. Consulta el [modo de varios inquilinos](#multi-tenancy).

[Invitación fuera de banda (OOB)]{#oob-invitation}

: Una invitación DIDComm autónoma, que suele entregarse como URL o código QR con un parámetro de consulta `_oob`. Permite que una nueva parte inicie una [conexión](#connection), [mediación](#mediation) o emisión o presentación sin conexión previa, sin un canal existente.

[JOSE]{#jose}

: La familia de estándares criptográficos basada en JSON —JSON Web Signature, JSON Web Encryption, JSON Web Key y JSON Web Token— que suele proteger credenciales. Consulta también [COSE](#cose).

[JSON Web Key (JWK)]{#jwk}

: Un formato JSON para representar una clave criptográfica. Los documentos DID y el material de claves DIDComm suelen expresarse como JWK.

[JWT-VC]{#jwt-vc}

: Un formato de [credencial verificable](#verifiable-credential) que empaqueta afirmaciones en un JSON Web Token firmado. Normalmente se presenta completo y revela todas sus afirmaciones, sin [divulgación selectiva](#selective-disclosure).

[keyAgreement]{#key-agreement}

: Una [relación de verificación](#verification-relationship) en un [documento DID](#did-document) que enumera claves aprobadas para el acuerdo de claves; por ejemplo, establecer el cifrado de [DIDComm](#didcomm). Normalmente usa una clave [X25519](#x25519).

[Keycloak]{#keycloak}

: Un servidor de administración de identidad y acceso que Identus puede integrar para que los [inquilinos](#tenant) se autentiquen mediante [OIDC](#oidc) y reciban permisos de billetera mediante [UMA](#uma), en lugar de claves estáticas de API.

[Lista de estado]{#status-list}

: Un mecanismo de revocación publicado y eficiente en espacio, como Bitstring Status List o StatusList2021, donde cada bit representa el estado de [revocación o suspensión](#revocation) de una credencial. El verificador lee el bit que referencia [credentialStatus](#credential-status). En Identus lo proporciona [Pollux](#pollux).

[Marco de gobernanza]{#governance-framework}

: Las reglas que publica una [autoridad de gobernanza](#governance-authority) para describir quién puede hacer qué en un [ecosistema](#ecosystem). Un [registro de confianza](#trust-registry) las expresa durante la ejecución y permite consultarlas. La autoridad procede del marco de gobernanza, no de la tecnología del registro.

[Mediación]{#mediation}

: El acuerdo que establece la billetera del titular con un [mediador](#mediator) para que reciba y retransmita mensajes DIDComm en su nombre. Produce información de enrutamiento que la billetera anuncia después en su documento DID.

[Mediador]{#mediator}

: Un servicio DIDComm V2 que actúa como retransmisor estable y siempre conectado para agentes, como billeteras móviles, que no siempre son accesibles. Encola mensajes cifrados por [DIDPair](#didpair) y los entrega cuando el destinatario consulta o se conecta, sin poder leer el contenido cifrado. *No* es un [VDR](#vdr).

[Mensaje `forward`]{#forward-message}

: Un mensaje de enrutamiento DIDComm del protocolo `routing/2.0` que envuelve un mensaje ya cifrado para el destinatario final en una envoltura externa dirigida a un [mediador](#mediator). Así puede enrutarlo sin leer el contenido interno.

[Mercury]{#mercury}

: El [componente básico](#building-blocks) de Identus que proporciona la interfaz [DIDComm](#didcomm) V2 para mensajería segura entre agentes, independiente del transporte subyacente.

[Message Pickup]{#message-pickup}

: El protocolo DIDComm `messagepickup/3.0` que una billetera usa para consultar mensajes en cola de un [mediador](#mediator), solicitar su entrega y confirmar su recepción.

[Método de verificación]{#verification-method}

: Una entrada de un [documento DID](#did-document) que especifica una clave pública, por ejemplo como [JWK](#jwk), su tipo y su controlador. Las [relaciones de verificación](#verification-relationship) lo referencian para fines específicos.

[Método DID]{#did-method}

: El esquema —la parte posterior a `did:`, como `prism` o `peer`— que define cómo crear, resolver, actualizar y desactivar una clase de DID. W3C define el modelo de datos compartido; cada especificación de método define su propio registro, operaciones y ciclo de vida.

[Modo de varios inquilinos]{#multi-tenancy}

: Un modo de [Cloud Agent](#cloud-agent) en el que una instancia aloja muchos [inquilinos](#tenant) aislados, cada uno con su propia [billetera](#wallet), DIDs, credenciales y conexiones. El modo de un solo inquilino atiende una billetera predeterminada.

[NeoPRISM]{#neoprism}

: Un backend de nodo DID más reciente para Identus, recomendado para producción, que puede sustituir al [PRISM Node](#prism-node) heredado para publicar y resolver operaciones `did:prism`.

[Oferta de credencial]{#credential-offer}

: Un mensaje de un [emisor](#issuer) que propone emitir una credencial específica a un [titular](#holder) e inicia el flujo de [emisión de credenciales](#credential-issuance).

[OIDC (OpenID Connect)]{#oidc}

: Un protocolo estándar de autenticación sobre OAuth 2.0 que, mediante [Keycloak](#keycloak), autentica inquilinos y emite tokens de acceso.

[OpenID4VCI (OID4VCI)]{#openid4vci}

: OpenID for Verifiable Credential Issuance: un protocolo basado en OAuth 2.0 para que una billetera obtenga credenciales desde el punto de acceso del emisor. Es una vía alternativa de emisión a DIDComm.

[OpenID4VP]{#openid4vp}

: OpenID for Verifiable Presentations: un protocolo basado en OAuth 2.0 para solicitar y presentar credenciales. Es una vía alternativa de presentación a DIDComm.

[Parte que confía en el resultado]{#relying-party}

: La parte que consume un resultado de verificación y aplica políticas de negocio y confianza —emisores, esquemas y tipos de credenciales aceptados— para decidir si actúa. A menudo es el mismo actor que el [verificador](#verifier).

[Pluto]{#pluto}

: El [componente básico](#building-blocks) de Identus que define la interfaz de almacenamiento de datos de identidad: DIDs, claves, credenciales y estado de conexión. La aplicación proporciona la implementación concreta del almacenamiento.

[Política de verificación]{#verification-policy}

: La configuración del lado del verificador en Identus que expresa qué [esquemas de credenciales](#credential-schema) y [emisores de confianza](#trusted-issuers) acepta. Se aplica después de las comprobaciones criptográficas y es la forma práctica de la política de aceptación o [validación](#validation).

[Pollux]{#pollux}

: El [componente básico](#building-blocks) y subsistema de Cloud Agent responsable de operaciones de [credenciales verificables](#verifiable-credential): emisión, verificación, [divulgación selectiva](#selective-disclosure) y [estado de credenciales](#status-list).

[Presentación verificable (VP)]{#verifiable-presentation}

: Datos derivados de una o más [credenciales verificables](#verifiable-credential) que se comparten con un [verificador](#verifier) específico como respuesta a una solicitud. Según el formato, una presentación puede incluir una credencial completa, partes de una —[divulgación selectiva](#selective-disclosure)— o datos de varias.

[PRISM Node]{#prism-node}

: El componente Identus que implementa `did:prism` y actúa como nodo de segunda capa sobre el registro distribuido. Publica y resuelve DIDs PRISM y mantiene un estado interno indexado, sincronizado con la cadena subyacente. Sirve como [VDR](#vdr) para DIDs PRISM y se espera que esté siempre conectado.

[Profundidad de confirmación]{#confirmation-depth}

: El número de bloques que deben añadirse después de una transacción DID antes de tratar su estado como final y seguro para resolver. Es un parámetro ajustable al anclar DIDs en un registro distribuido.

[Protocolo Issue Credential]{#issue-credential}

: El protocolo DIDComm, Issue Credential 3.0 en Identus, que regula el intercambio de mensajes de oferta, solicitud y emisión de credenciales entre emisor y titular.

[Protocolo Present Proof]{#present-proof}

: El protocolo DIDComm independiente del formato, Present Proof 3.0 en Identus, que transporta una [solicitud de presentación](#presentation-request) del verificador y una [presentación](#verifiable-presentation) de respuesta del titular.

[Prueba de conocimiento cero (ZKP)]{#zero-knowledge-proof}

: Una técnica criptográfica que demuestra que una declaración es verdadera sin revelar los datos subyacentes. Es la base de las [pruebas de predicados](#predicate-proof) [AnonCreds](#anoncreds), como «más de 21» sin divulgar una fecha de nacimiento.
[Prueba de predicado]{#predicate-proof}

: Una prueba de que un atributo satisface una condición, como `age >= 21`, sin divulgar el valor original. Es una función esencial de [AnonCreds](#anoncreds) y las [pruebas de conocimiento cero](#zero-knowledge-proof).

[Punto de acceso de servicio]{#service-endpoint}

: Una entrada de un [documento DID](#did-document) que anuncia dónde y cómo interactuar con el sujeto DID. Por ejemplo, un punto de acceso `DIDCommMessaging` que indica la URL, el perfil DIDComm aceptado y las claves de enrutamiento.

[Registro de confianza]{#trust-registry}

: Un registro oficial, con gobernanza y legible por máquina de las entidades autorizadas para cada rol de un [ecosistema](#ecosystem). El verificador lo consulta para confirmar la *autoridad* del emisor, una pregunta separada de la *validez* criptográfica de la credencial. No es una [autoridad de certificación](#certificate-authority) ni un [VDR](#vdr).

[Registro de datos verificables (VDR)]{#vdr}

: Un sistema, a menudo una cadena de bloques, pero también una base de datos, red distribuida o almacén en memoria, donde se publican y resuelven DIDs y datos relacionados. Permite que partes independientes verifiquen autenticidad e integridad sin autoridad central. Identus accede a VDR mediante drivers intercambiables, como los de [PRISM Node](#prism-node) y [NeoPRISM](#neoprism). Un [mediador](#mediator) no es un VDR.

[Relación de verificación]{#verification-relationship}

: La asociación en un [documento DID](#did-document) entre un [método de verificación](#verification-method) y un fin permitido: [authentication](#authentication), [assertionMethod](#assertion-method), [keyAgreement](#key-agreement), `capabilityInvocation` o `capabilityDelegation`. Estas relaciones evitan reutilizar una clave entre distintos fines de prueba.

[Resolución DID]{#did-resolution}

: El proceso de buscar un DID y devolver su [documento DID](#did-document) junto con metadatos de resolución. Un [resolvedor](#resolver) que comprende el método DID correspondiente lo ejecuta.

[Resolvedor]{#resolver}

: Software que realiza la [resolución DID](#did-resolution): recibe un DID y devuelve su [documento DID](#did-document). Un [resolvedor universal](#universal-resolver) dirige muchos métodos a drivers específicos. Elegir un resolvedor es una decisión de confianza.

[Resolvedor universal]{#universal-resolver}

: Un servicio público de [resolución](#resolver) que puede resolver muchos [métodos DID](#did-method) al dirigir cada uno a un driver específico.

[Revocación / suspensión]{#revocation}

: Invalidar una credencial emitida antes de su caducidad natural, de forma permanente —revocación— o temporal —suspensión—. El cambio aparece en una [lista de estado](#status-list) que los verificadores consultan mediante [credentialStatus](#credential-status).

[Ruta de derivación]{#derivation-path}

: La ruta determinista que, junto con una [semilla de billetera](#wallet-seed), regenera las claves de un DID específico mediante derivación jerárquica determinista. Cloud Agent almacena la ruta en lugar de las claves derivadas.

[SD-JWT / SD-JWT-VC]{#sd-jwt}

: Selective Disclosure JWT: una variante JWT que usa hashes o resúmenes con sal para que el titular revele solo afirmaciones seleccionadas mientras el verificador compara cada valor divulgado con el resumen firmado. SD-JWT-VC es el formato de credencial verificable basado en ella. Identus lo admite. Consulta [divulgación](#disclosure) y [cnf](#cnf).

[secp256k1]{#secp256k1}

: Un algoritmo de clave y firma de curva elíptica, también usado por Bitcoin y Cardano, que Identus admite para ciertas pruebas de credenciales y listas de estado.

[Secreto de vinculación]{#link-secret}

: Un secreto bajo el control del titular que [AnonCreds](#anoncreds) usa para vincular credenciales emitidas al titular y permitir pruebas de conocimiento cero entre varias credenciales.

[Semilla de billetera]{#wallet-seed}

: El material secreto de clave raíz del que una billetera deriva determinísticamente sus claves DID mediante una [ruta de derivación](#derivation-path) almacenada. Es un secreto de gran valor: perderlo puede impedir de forma permanente usar o actualizar DIDs existentes.

[Solicitud de presentación]{#presentation-request}

: Un mensaje, también llamado solicitud de prueba, en el que el [verificador](#verifier) especifica qué prueba necesita del titular. El protocolo Present Proof lo transporta.

[Sujeto]{#subject}

: La persona, organización, dispositivo, cuenta o cosa que describen las [afirmaciones](#claim) de una credencial. A menudo es el [titular](#holder), pero no necesariamente: un progenitor puede conservar una credencial de su hijo, o un administrador de flota una de un dispositivo.

[Titular]{#holder}

: El rol que recibe [credenciales verificables](#verifiable-credential), las almacena normalmente en una [billetera](#wallet) y crea [presentaciones](#verifiable-presentation) en respuesta a solicitudes del verificador. A menudo es el [sujeto](#subject) de la credencial, pero no siempre.

[ToIP (Trust over IP Foundation)]{#toip}

: Un organismo de Linux Foundation que desarrolla estándares de confianza digital descentralizada, incluido ToIP Stack —un modelo por capas de DIDs, DIDComm, protocolos de intercambio de datos y ecosistemas de aplicaciones— y [Trust Registry Query Protocol](#trqp).

[Triángulo de confianza]{#triangle-of-trust}

: Una forma común de describir los tres roles SSI —[emisor](#issuer), [titular](#holder) y [verificador](#verifier)— y los flujos de credenciales y presentaciones entre ellos. Los roles dependen de la interacción: una entidad puede emitir una credencial, conservar otra y verificar una tercera.

[TRQP (Trust Registry Query Protocol)]{#trqp}

: Un protocolo de solo lectura de [ToIP](#toip) para preguntar a un [registro de confianza](#trust-registry) si una entidad tiene una autorización según la gobernanza de un ecosistema. En términos simples: «¿La entidad X tiene la autorización Y según el marco de gobernanza Z?». A veces se describe como «DNS para registros de confianza».

[UMA (User-Managed Access)]{#uma}

: Un estándar de autorización basado en OAuth 2.0 que [Keycloak](#keycloak) usa para conceder a un sujeto permisos sobre un recurso de [billetera](#wallet) específico de Cloud Agent. Emite un token de parte solicitante (RPT) que el inquilino presenta al agente.

[Validación]{#validation}

: La comprobación de reglas de negocio del verificador para determinar si una credencial es *adecuada* para una decisión; por ejemplo, si el emisor resulta aceptable según una política de contratación. Es distinta de la [verificación](#verification): una credencial puede ser criptográficamente válida y no superar la validación.

[Verificación]{#verification}

: La comprobación técnica de que una credencial o presentación es criptográficamente correcta: firmas válidas, claves del emisor resueltas correctamente, uso adecuado de claves según las [relaciones de verificación](#verification-relationship), [estado](#status-list) actual y [vinculación del titular](#holder-binding). Es distinta de la [validación](#validation), que aplica políticas de negocio.

[Verificador]{#verifier}

: El rol que solicita una [presentación](#verifiable-presentation) al titular y decide si la acepta, mediante [verificación](#verification) criptográfica y [validación](#validation) de política de negocio. Consulta también [parte que confía en el resultado](#relying-party).

[Vinculación de clave]{#key-binding}

: Vincular criptográficamente una credencial a una clave del titular para que demuestre su control al presentarla. Consulta también [vinculación del titular](#holder-binding) y [cnf](#cnf).

[Vinculación del titular]{#holder-binding}

: Prueba criptográfica de que la parte que presenta una credencial controla la clave asociada a su sujeto. Consulta también [vinculación de clave](#key-binding).

[Webhook]{#webhook}

: El mecanismo de callback HTTP por el que [Cloud Agent](#cloud-agent) notifica al [controlador](#controller) cambios de estado, como una conexión recibida, una credencial emitida o recibida, o una presentación verificada.

[X25519]{#x25519}

: Un tipo de clave de curva elíptica para el acuerdo de claves Diffie-Hellman que establece el cifrado de [DIDComm](#didcomm). Aparece en los documentos DID bajo la relación [keyAgreement](#key-agreement).
