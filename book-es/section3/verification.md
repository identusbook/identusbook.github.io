# Verificación {#sec-verification}

## Presentar una prueba

La verificación en Identus comienza cuando un verificador pide a un titular que presente pruebas de una o más credenciales. Para flujos DIDComm, Identus Cloud Agent usa el protocolo Present Proof. El Cloud Agent del verificador envía una solicitud de presentación mediante una conexión DIDComm existente. El Cloud Agent del titular devuelve una presentación y el del verificador comprueba el material de prueba recibido.

La [guía Present Proof de Identus](https://github.com/hyperledger-identus/cloud-agent/blob/main/docs/docusaurus/credentials/didcomm/present-proof.md) describe el flujo de API DIDComm:

1. El controlador del verificador llama a `POST /present-proof/presentations` en su Cloud Agent. La solicitud identifica la conexión DIDComm y describe la prueba que quiere el verificador.
2. El controlador del titular lee los registros de presentaciones pendientes desde `GET /present-proof/presentations`.
3. El controlador del titular acepta una solicitud mediante `PATCH /present-proof/presentations/{id}` con `action: "request-accept"`. Para credenciales JWT y SD-JWT, proporciona IDs de registros de credenciales mediante `proofId`. Para AnonCreds, proporciona la correspondencia de pruebas de credenciales en `anoncredPresentationRequest`.
4. El Cloud Agent del titular genera la presentación y la envía al Cloud Agent del verificador.
5. El Cloud Agent del verificador verifica la presentación. Un registro correcto del lado del verificador pasa por `PresentationReceived` y `PresentationVerified`.
6. El controlador del verificador acepta la presentación verificada mediante `PATCH /present-proof/presentations/{id}` con `action: "presentation-accept"`, o la rechaza en la lógica de la aplicación.

La estructura exacta de la solicitud depende del formato de credencial. Las solicitudes de presentación JWT y SD-JWT pueden incluir `options.challenge` y `options.domain` para que el titular firme material vinculado a la solicitud del verificador. Las solicitudes AnonCreds usan un objeto `anoncredPresentationRequest` con un nonce, atributos solicitados, predicados solicitados, restricciones e intervalos opcionales de no revocación.

El [protocolo DIDComm Present Proof 3.0](https://didcomm.org/present-proof/3.0/) es independiente del formato. Transporta una solicitud de presentación y una presentación, pero el formato de credencial determina las comprobaciones criptográficas que se ejecutan.

::: {.content-visible when-format="html:js"}
{{< include ../diagrams/d14-presentation-verification.html >}}
:::

::: {.content-visible unless-format="html:js"}
![El verificador proporciona la vinculación de la solicitud para el formato elegido: challenge y domain para JWT o SD-JWT con vinculación de clave, o un nonce de AnonCreds. El controlador del titular selecciona credenciales y llama a request-accept. Después de PresentationVerified, el controlador del verificador comprueba su política y llama a presentation-accept solo si supera las comprobaciones. Un rechazo por política detiene la acción de la aplicación. Este diagrama sigue el flujo correcto y omite respuestas REST.](../diagrams/d14-presentation-verification.svg){fig-alt="Presentar una prueba y aceptar el resultado"}
:::

## Verificación y aceptación

La verificación criptográfica y la aceptación del verificador son decisiones separadas.

El Cloud Agent del verificador puede comprobar si la presentación tiene una estructura correcta para su formato, está vinculada a la solicitud del verificador y cuenta con el material criptográfico esperado. Para JWT-VC, comprueba las firmas JWT y las claves resueltas mediante DID. Para SD-JWT-VC, comprueba la firma del emisor, los resúmenes de las afirmaciones divulgadas y la vinculación de clave del titular cuando la credencial contiene una clave `cnf`. Para AnonCreds, comprueba pruebas de igualdad, de predicados, agregadas y las pruebas de no revocación que solicitó el verificador.

El controlador del verificador aún decide si debe aceptar la prueba para la acción de negocio. El [modelo de datos de credenciales verificables 2.0 de W3C](https://www.w3.org/TR/vc-data-model-2.0/) separa verificación y validación: la verificación comprueba el mecanismo de protección y la validación aplica reglas de negocio a un uso específico de la credencial.

Después de que Cloud Agent informe de una presentación verificada, el controlador del verificador debe comprobar:

1. El verificador confía en el DID del emisor para el tipo y esquema de credencial.
2. La credencial usa el esquema o definición de credencial que indica la solicitud del verificador.
3. Las afirmaciones divulgadas satisfacen la política del verificador.
4. El período de validez y los datos de estado o revocación corresponden al momento que interesa al verificador.
5. El verificador acepta que el titular o presentador tiene autorización para este caso de uso.

Una prueba válida aún puede no superar la aceptación. El verificador de un local puede recibir una credencial de edad criptográficamente válida y rechazarla si el emisor no está en su lista de emisores de confianza o si la afirmación revelada no cumple la regla de edad del local.

## Verificación directa de credenciales

Cloud Agent expone un punto de acceso de verificación directa en `POST /verification/credential`. Úsalo cuando el software del verificador tiene una credencial JWT codificada y necesita resultados explícitos de comprobación fuera de un intercambio DIDComm Present Proof.

El punto de acceso acepta una lista de credenciales codificadas y comprobaciones de verificación. La implementación actual del servicio incluye comprobaciones como verificación de firma y algoritmo, identificación del emisor, caducidad, fecha mínima de validez, audiencia, validación de esquema, validación del esquema del sujeto y análisis semántico de afirmaciones.

El punto de acceso de verificación directa no sustituye una solicitud de presentación. Verifica los datos de credencial que recibe. No demuestra que el titular actual controla la clave del sujeto de la credencial para un desafío del verificador, ni decide si la aplicación debe aceptar un emisor o una afirmación.

## Políticas de verificación

Identus usa el término `verification policy` para reglas almacenadas del verificador. El código de Cloud Agent expone puntos de acceso `/verification/policies` para crear, actualizar, obtener, listar y eliminar esas políticas. El modelo actual contiene una `CredentialSchemaAndTrustedIssuersConstraint` con:

1. `schemaId`, el esquema que espera el verificador.
2. `trustedIssuers`, los DIDs de emisores que acepta para ese esquema.

La descripción del punto de acceso en el código de Cloud Agent indica que estas políticas se aplican a credenciales verificables W3C en formato JWT y actualmente usan las restricciones `schemaId` y `trustedIssuers`.

El controlador del verificador puede usar una política antes de enviar una solicitud de presentación, después de recibir una presentación verificada o en ambos momentos. Antes de la solicitud, la política puede ayudarlo a pedir el esquema que acepta. Después de la verificación, puede ayudarlo a rechazar una credencial de un emisor que no es de confianza. El resultado de verificación criptográfica de Cloud Agent debe contribuir a esta decisión de política, no representar toda la decisión.

Si el despliegue usa un registro de confianza externo, mantén la consulta del registro separada de la verificación de pruebas. El verificador o su controlador consulta el registro, resuelve el conjunto de emisores de confianza para el esquema solicitado e incorpora el resultado a su decisión de política.

## Divulgación selectiva

La divulgación selectiva permite que el titular revele solo las afirmaciones necesarias para la solicitud del verificador. Identus admite presentaciones con divulgación selectiva mediante SD-JWT y AnonCreds, pero ambos formatos usan mecanismos distintos.

La presentación JWT-VC simple expone la carga útil de la credencial necesaria para las comprobaciones del verificador. El formato sirve cuando este necesita la credencial completa, como una puerta de embarque que comprueba toda la tarjeta de embarque. Resulta poco adecuado cuando necesita un atributo o un predicado.

SD-JWT usa hashes con sal. El emisor firma una carga útil JWT que contiene resúmenes de las afirmaciones que permiten divulgación selectiva. Durante la presentación, el titular envía el SD-JWT firmado por el emisor y las divulgaciones seleccionadas para ese verificador. El verificador calcula el hash de cada valor divulgado con su sal y comprueba que el resumen aparece en la carga útil firmada. SD-JWT proporciona divulgación selectiva para afirmaciones JSON; no es un sistema de pruebas de predicados de conocimiento cero.

Identus añade un detalle de vinculación de clave para las presentaciones SD-JWT. Si el titular acepta una oferta SD-JWT con `keyId`, la credencial emitida puede incluir una vinculación de clave `cnf`. Una presentación posterior del titular puede firmar el `challenge` y el `domain` del verificador. Si la credencial SD-JWT no tiene clave `cnf`, la guía Present Proof de Identus indica que el titular no puede crear una presentación que firme el `challenge` y el `domain` del verificador.

::: {.content-visible when-format="html:js"}
{{< include ../diagrams/d15-selective-disclosure.html >}}
:::

::: {.content-visible unless-format="html:js"}
![Este ejemplo usa dos propiedades de objeto que permiten divulgación selectiva. Cada Disclosure codifica \[salt, name, value\] como base64url. El verificador calcula el hash de esa cadena codificada y compara el resumen base64url con la carga útil firmada por el emisor, según RFC 9901, sección 4.2.3. Las etiquetas d1 y d2 representan resúmenes completos. Una credencial con cnf puede admitir vinculación de clave del titular a la solicitud del verificador.](../diagrams/d15-selective-disclosure.svg){fig-alt="Divulgación selectiva en una credencial SD-JWT"}
:::

AnonCreds usa pruebas de conocimiento cero. La solicitud de presentación del verificador puede pedir atributos revelados, predicados como `age >= 18`, restricciones como `schema_id` o `cred_def_id` e intervalos de no revocación. El titular selecciona credenciales que satisfacen la solicitud y crea una prueba. El verificador resuelve los esquemas, definiciones de credenciales y datos de revocación necesarios, y verifica la presentación.

AnonCreds permite que el titular demuestre un predicado sin revelar el atributo original. Para verificar la edad, un emisor público puede emitir una credencial que contiene un atributo de edad derivado de una fecha. Un local con restricción de edad puede solicitar un predicado AnonCreds para `age >= 21`. El titular presenta una prueba que satisface el predicado y el verificador comprueba la prueba, la restricción de emisor, la definición de credencial y el intervalo de revocación sin recibir la fecha completa de nacimiento del titular.

La divulgación selectiva no elimina la política del verificador. Este aún comprueba la confianza en el emisor, la correspondencia del esquema, el estado de la credencial, la vinculación de la solicitud y las reglas locales de aceptación de las afirmaciones o predicados divulgados.
