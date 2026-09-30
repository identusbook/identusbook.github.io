# Mantenimiento {#sec-maintenance}

Dominar Identus implica mantener la aplicación después de lanzarla. El despliegue de producción del [capítulo @sec-installation-production] no es un solo proceso. Es un conjunto de servicios de larga ejecución: Cloud Agent, PRISM Node o NeoPRISM, HashiCorp Vault, PostgreSQL y, en modo Cardano, un nodo Cardano, Cardano Wallet y Cardano DB Sync. Cada uno tiene su propio ciclo de vida, almacenamiento y modos de fallo. Este capítulo cubre el trabajo posterior al lanzamiento para mantener el entorno en buen estado: reiniciarlo correctamente, observarlo, actualizarlo sin romper la compatibilidad y proteger sus secretos y claves.

Este capítulo presupone el despliegue descrito en el capítulo de instalación de producción. Las indicaciones son concretas donde ese despliegue las sustenta. Donde las prácticas operativas de Identus aún son inmaduras o no tienen documentación, la sección incluye un marcador para ampliarla en un borrador posterior.

## Reinicio y limpieza

Los servicios de un despliegue en modo Cardano tienen un orden de dependencias que los reinicios deben respetar. Cardano Wallet y Cardano DB Sync se conectan al nodo Cardano mediante su socket. PRISM Node depende de Cardano Wallet y DB Sync, y Cloud Agent depende de PRISM Node y PostgreSQL. Un reinicio correcto normalmente avanza desde el registro distribuido hacia la aplicación:

1. Inicia o comprueba PostgreSQL y abre Vault. Cloud Agent no puede leer semillas de billetera mientras Vault está sellado.
2. Inicia `cardano-node` y después `cardano-wallet` y `cardano-db-sync` cuando el socket del nodo esté disponible.
3. Espera a que DB Sync se acerque al último bloque de la cadena antes de depender de la publicación o resolución `did:prism`.
4. Inicia PRISM Node, después Cloud Agent y después la aplicación controladora.

Un reinicio no vuelve a anclar los DIDs existentes. Cloud Agent reconstruye el material de claves DID a partir de la semilla de billetera y la ruta de derivación almacenada. Por ello, disponer de la semilla —con Vault abierto y accesible— es el requisito previo para los flujos de emisión y actualización DID tras reiniciar.

La limpieza trata principalmente del disco y los registros. Los servicios Cardano generan registros continuos durante la sincronización. Mantén configurada la rotación para todos los contenedores de larga ejecución. La base de datos PostgreSQL de DB Sync crece continuamente y consume más disco que el resto del entorno. A diferencia de las bases de datos de Cloud Agent y PRISM Node, los datos DB Sync son estado derivado de la cadena: pueden reconstruirse mediante una nueva sincronización o restaurarse desde una instantánea de confianza. Por ello pueden depurarse o reconstruirse en lugar de mantener copias de seguridad a largo plazo.

> TODO: Añadir procedimientos concretos de limpieza: eliminar imágenes y volúmenes Docker, recuperar disco de DB Sync y decidir entre nueva sincronización e instantánea con tiempos medidos en `preprod` y `mainnet`.

## Observabilidad

Operar bien Identus requiere observar la capa de aplicación —Cloud Agent y controlador— y la capa del registro distribuido —servicios Cardano—. Un problema en cualquiera de ellas puede detener la emisión de credenciales mientras Cloud Agent sigue «activo».

### Comprobaciones de estado y servicios

La lista de comprobaciones de producción del [capítulo @sec-installation-production] es el punto de partida para la supervisión continua. Las comprobaciones que conviene ejecutar continuamente, además del lanzamiento, incluyen:

- El estado de Cloud Agent devuelve la versión esperada y la `REST_SERVICE_URL` pública responde mediante HTTPS.
- Las billeteras de titulares y servicios de mediador pueden acceder a `DIDCOMM_SERVICE_URL`.
- Cardano DB Sync está cerca del último bloque de la cadena. Si se retrasa, demora silenciosamente la publicación y resolución de DIDs.
- Cardano Wallet informa de un estado de red correcto (`/v2/network/information`) y, en mainnet, la dirección de pago de PRISM Node tiene suficiente ADA para comisiones.
- Una comprobación de extremo a extremo: una operación de creación `did:prism` alcanza el estado confirmado y después se resuelve en forma corta.

### Administración de nodos y memoria

Cardano DB Sync consume más recursos que el resto del entorno y sus requisitos dominan el dimensionamiento. El capítulo de producción registra cifras mainnet del orden de 64 GB de RAM, 4 o más núcleos de CPU, SSD con 60 000 IOPS o más y varios cientos de GB de disco que crecen con el tiempo. `preprod` es mucho menor. Ejecuta DB Sync y su PostgreSQL cerca uno del otro, idealmente juntos, para reducir la latencia de sincronización. Supervisa las IOPS si separas servicios entre máquinas.

Durante la operación, observa:

- El crecimiento de disco PostgreSQL de DB Sync y el espacio restante.
- La memoria de `cardano-node` y `cardano-db-sync` frente a los límites del host.
- El tamaño y la latencia de consultas de las bases de datos de Cloud Agent y PRISM Node, que afectan al rendimiento de emisión y verificación.

### Pruebas de rendimiento

> TODO: Marcador de posición. El libro aún no define una metodología de pruebas de rendimiento para Identus. Ampliar esta sección con una prueba de carga repetible —rendimiento de emisión, verificación y resolución DID—, métricas, valores objetivo y cómo la profundidad de confirmación Cardano limita la latencia de emisión de extremo a extremo.

### Análisis con BlockTrust Analytics

> TODO: Marcador de posición. BlockTrust Analytics es una herramienta de análisis de terceros para Identus/PRISM. Ampliar esta sección con lo que observa, cómo conectarla a un despliegue en ejecución y qué preguntas operativas responde que las comprobaciones de estado básicas no responden.

## Actualización de agentes

### Selección de versiones y compatibilidad

Trata las notas de versiones de Identus Platform como la primera fuente de compatibilidad. Una versión de plataforma fija componentes que se sabe que funcionan juntos: por ejemplo, versiones de Cloud Agent, Mediator, NeoPRISM y SDK. Pueden existir versiones independientes más recientes, como una nueva imagen de Cloud Agent o PRISM Node, pero requieren comprobar la compatibilidad con la versión fijada de plataforma. Los pares de imágenes Cloud Agent y PRISM Node sin comprobar pueden fallar en las comprobaciones de compatibilidad.

En la práctica, las actualizaciones cambian una o más de estas versiones fijas:

- `AGENT_VERSION` (Cloud Agent).
- `PRISM_NODE_VERSION` (PRISM Node) o la versión del backend NeoPRISM.
- `CARDANO_WALLET_TAG` y su versión asociada de `cardano-node`.
- `CARDANO_DB_SYNC_VERSION`.

Cuando cambies cualquier imagen Identus, vuelve a ejecutar la lista completa de comprobaciones de producción antes de depender del despliegue.

### Procedimiento de actualización y notas de compatibilidad

Las actualizaciones de Cardano DB Sync necesitan especial atención: lee las notas de versión antes de cambiar la etiqueta de imagen. Pueden ejecutar migraciones de esquemas, como una migración de la tabla `epoch` entre versiones secundarias, y la compatibilidad de instantáneas depende de cada versión. Valida una actualización DB Sync en `preprod` antes de aplicarla a `mainnet`. Confirma que la nueva versión aún llega al último bloque de la cadena y que PRISM Node puede publicar y resolver un DID de prueba después.

Para Cloud Agent y PRISM Node, confirma que el par es una combinación probada. Después actualiza y ejecuta un flujo completo del emisor —crear DID, emitir, revocar o suspender si corresponde, presentar y verificar— antes de cambiar el tráfico.

### Reducir el tiempo de inactividad

> TODO: Marcador de posición. Documentar una estrategia de actualización con poco tiempo de inactividad. Resolver antes estas preguntas: si Cloud Agent permite ejecutar versiones antiguas y nuevas en paralelo con una base de datos compartida, cómo aplica las migraciones durante una actualización y cómo terminar los flujos DIDComm y de emisión en curso. Hasta entonces, planificar una ventana de mantenimiento.

## HashiCorp Vault y administración de claves

### Administración de claves

En producción, Cloud Agent almacena material de semillas de billetera en Vault (`SECRET_STORAGE_BACKEND=vault`), bajo rutas específicas como `/secret/<wallet-id>/seed`, junto con rutas de claves de DIDs de pares y otros secretos. La semilla de billetera del inquilino permite volver a derivar claves DID después de reiniciar o desplegar de nuevo. Perderla puede inutilizar DIDs existentes para actualizaciones, desactivación o emisión futuras. Por tanto, Vault es el componente más importante que debes proteger y respaldar en el despliegue.

Aspectos operativos esenciales del capítulo de producción:

- Usa AppRole (`VAULT_APPROLE_ROLE_ID` / `VAULT_APPROLE_SECRET_ID`) para despliegues automáticos. Reserva `VAULT_TOKEN` para laboratorio o emergencias.
- Ejecuta Vault con refuerzo de producción: TLS, usuario de ejecución sin privilegios, una decisión explícita sobre bloqueo de memoria o intercambio cifrado, separación de almacenamiento y rutas de red restringidas. Vault con `server -dev` es solo almacenamiento de laboratorio.
- Crea copias de seguridad de Vault, comprueba la restauración y documenta el procedimiento de apertura o claves de recuperación. Tras cualquier reinicio de Vault, debes abrir el almacén antes de que Cloud Agent lea semillas.

### Copias de seguridad y restauración

Crea copias de seguridad según el tipo de datos, porque los servicios requieren tratamientos distintos:

- **Las bases de datos de Cloud Agent y PRISM Node** contienen estado de aplicación. Crea copias de seguridad y comprueba las restauraciones.
- **El almacenamiento Vault** contiene semillas de billetera y material de claves. Crea copias de seguridad, comprueba la restauración y protege las claves de apertura y recuperación.
- **La frase mnemónica y contraseña de gasto de Cardano Wallet**, junto con la dirección de pago de PRISM Node, pertenecen al sistema de secretos de producción. Nunca deben estar en archivos `.env`, registros de CI, historial de la terminal o el repositorio.
- **La base de datos Cardano DB Sync** contiene estado derivado de la cadena. Puede reconstruirse mediante sincronización o restaurarse desde una instantánea de confianza. Su copia de seguridad tiene menor prioridad que el estado de aplicación.

### Rotación de claves y credenciales

La rotación abarca dos tipos de secretos. Las credenciales operativas —`ADMIN_TOKEN`, claves de API de inquilinos, contraseñas de bases de datos, credenciales de Vault, secretos de cliente Keycloak y contraseña de Cardano Wallet— protegen el acceso a los servicios. Se rotan mediante el mismo proceso que el resto de secretos de producción. Las claves DID son distintas: son material a nivel de protocolo del que dependen los verificadores, y rotarlas cambia lo que otras partes pueden verificar. Los procedimientos siguientes se basan en Cloud Agent 2.1.0. Comprueba las notas de la versión al actualizar, porque el comportamiento de selección de claves aún evoluciona en el agente.

**Credenciales operativas.** La mayoría se rotan con un reinicio, pero debes conocer algunos comportamientos antes de empezar:

| Credencial | Procedimiento | Efecto |
|---|---|---|
| `ADMIN_TOKEN` | Genera un valor nuevo, actualiza el almacén de secretos y reinicia Cloud Agent. | El agente acepta un solo token de administrador. La automatización administrativa falla hasta que use el nuevo valor; las llamadas de API de los inquilinos no se ven afectadas. |
| Claves de API de inquilinos | Registra una clave nueva con `POST /iam/apikey-authentication`, cambia la aplicación del inquilino para que la use y después elimina el registro de la clave antigua con `DELETE /iam/apikey-authentication` y el mismo cuerpo `entityId`/`apiKey`. | Ninguno si sigues ese orden: una entidad puede tener varias claves de API a la vez. |
| `API_KEY_SALT` | No lo rotes como mantenimiento rutinario. | Cloud Agent almacena las claves de API como hashes con sal. Cambiar la sal invalida todas las claves de API registradas a la vez. Trátalo como un nuevo aprovisionamiento de todos los inquilinos. |
| `secret_id` de AppRole de Vault | Consulta el procedimiento siguiente. | Ninguno si el `secret_id` antiguo sigue siendo válido hasta que todas las instancias del agente se hayan reiniciado. |

Genera siempre un valor aleatorio nuevo para una clave de API de reemplazo. Cloud Agent conserva un registro de las claves cuyo registro se ha eliminado y rechaza registrar el mismo valor de nuevo. Si alguien presenta una clave ya registrada para una entidad *distinta*, el agente la considera comprometida y la desactiva.

**Rotar el `secret_id` de AppRole de Vault.** Cloud Agent inicia sesión en Vault con `VAULT_APPROLE_ROLE_ID` y `VAULT_APPROLE_SECRET_ID` al arrancar, y vuelve a iniciar sesión con el mismo par antes de que caduque cada concesión de token. Revocar el `secret_id` antiguo mientras un agente aún lo usa no causa un fallo inmediato. El fallo ocurre en el siguiente inicio de sesión, cuando el agente ya no puede leer las semillas de billetera. Rota en este orden:

1. Genera un `secret_id` nuevo para el rol y registra su accessor:

   ```bash
   vault write -f auth/approle/role/cloud-agent/secret-id
   ```

2. Actualiza `VAULT_APPROLE_SECRET_ID` en el almacén de secretos de producción y reinicia o sustituye de forma gradual todas las instancias de Cloud Agent.
3. Confirma que cada instancia está en buen estado y puede acceder al material de billetera, por ejemplo al enumerar los DIDs de un inquilino.
4. Destruye el `secret_id` antiguo mediante su accessor:

   ```bash
   vault write auth/approle/role/cloud-agent/secret-id-accessor/destroy \
     secret_id_accessor=replace-with-old-accessor
   ```

Si el rol establece `secret_id_ttl` o `secret_id_num_uses`, recuerda que los inicios de sesión periódicos del agente los consumen. Un `secret_id` que caduca o agota sus usos mientras el agente está en ejecución tiene el mismo efecto que revocarlo.

**Rotar claves `did:prism`.** Un controlador `did:prism` rota claves de verificación al publicar una operación DID firmada de **actualización** mediante PRISM Node. Esto cambia los métodos y relaciones de verificación del documento DID en la cadena, como explica el [capítulo @sec-did-and-diddocuments]. En Cloud Agent, la operación es `POST /did-registrar/dids/{didRef}/updates`, con acciones `ADD_KEY` y `REMOVE_KEY`. El agente deriva la clave nueva de la semilla de billetera y registra su ruta de derivación.

La restricción principal afecta al verificador. Cuando Cloud Agent verifica una credencial JWT, resuelve el documento DID *actual* del emisor y comprueba la firma con las claves incluidas en `assertionMethod`. Por tanto, eliminar una clave impide verificar todas las credenciales que firmó esa clave, no solo las futuras. Rota una clave de emisión con un período de solapamiento:

1. **Añade la clave nueva.** Envía una actualización que añada una clave con el mismo propósito y curva que la clave que sustituyes:

   ```bash
   curl -X POST "https://agent.example.test/cloud-agent/did-registrar/dids/$ISSUER_DID/updates" \
     -H "apikey: replace-with-tenant-api-key" \
     -H "Content-Type: application/json" \
     -d '{
       "actions": [
         {
           "actionType": "ADD_KEY",
           "addKey": { "id": "assertion-2", "purpose": "assertionMethod", "curve": "secp256k1" }
         }
       ]
     }'
   ```

   El agente acepta una sola actualización pendiente por DID. Espera hasta que la operación se confirme en la cadena y el DID se resuelva con la clave nueva antes de enviar otra actualización.

2. **Indica la clave de firma en cada solicitud de emisión.** Mientras el DID tenga dos claves `assertionMethod`, Cloud Agent rechaza elegir una: falla la emisión cuando firma una credencial JWT para una oferta que no estableció un ID de clave de emisión. La búsqueda de la clave ocurre al firmar, por lo que esto también afecta a ofertas creadas antes de añadir la clave nueva que aún esperan la solicitud del titular. Establece `issuingKid` en `jwtVcPropertiesV1`, por ejemplo `"issuingKid": "assertion-2"`, en cada oferta. Actualiza la aplicación del emisor *antes* de que se confirme el paso 1 y deja que las ofertas en curso terminen primero. En esta versión, la emisión OID4VCI no acepta un ID de clave, por lo que falla mientras el DID tenga más de una clave `assertionMethod`. Si la usas, planifica el solapamiento con esta restricción.
3. **Vuelve a emitir las credenciales** que deban seguir siendo verificables después de eliminar la clave antigua, y fírmalas con la nueva. Puedes conservar sin cambios las credenciales que caduquen o se revoquen antes de la transición.
4. **Elimina la clave antigua** mediante una acción `REMOVE_KEY` (`"removeKey": { "id": "assertion-1" }`) cuando ninguna credencial que aún necesites dependa de ella. Después de eliminarla, confirma que una credencial recién emitida se verifica y que la credencial de lista de estado aún se verifica. El agente firma las credenciales de listas de estado con la primera clave `assertionMethod` que encuentra. Por tanto, una lista de estado cuya última firma usó la clave eliminada puede necesitar una actualización —una revocación o suspensión— para firmarse de nuevo.

La clave `master0` del DID, que firma las propias operaciones de actualización, está reservada. Cloud Agent rechaza acciones `ADD_KEY` y `REMOVE_KEY` que la nombren, por lo que no puede rotarse mediante la API.

**La semilla de billetera no puede rotarse.** La semilla se fija al crear la billetera. Cloud Agent no tiene un punto de acceso para sustituirla, y todas las claves PRISM de la billetera, incluida `master0`, se derivan de ella. «Rotar» una semilla implica crear una billetera nueva con una semilla nueva, crear y publicar DIDs nuevos en ella, volver a emitir credenciales desde esos DIDs y trasladar a los inquilinos y las partes que confían en el resultado a los nuevos DIDs. Planifícalo como una migración.

Esto también define el alcance de una exposición de la semilla. Cualquiera que tenga la semilla puede derivar `master0` y publicar operaciones de actualización o desactivación para cada DID PRISM de esa billetera. Por tanto, la respuesta es urgente: desactiva los DIDs afectados, crea una billetera y DIDs nuevos y vuelve a emitir las credenciales. Al desactivar un DID, los verificadores ya no pueden resolver claves utilizables para él, por lo que dejan de verificar todas las credenciales que emitió. Ese es el resultado previsto cuando ya no se puede confiar en las claves de firma, pero debes comunicarlo a los titulares y las partes que confían en el resultado antes de que ocurra, cuando sea posible. Por eso, la semilla y el almacenamiento Vault que la conserva son los elementos más importantes que debes proteger y respaldar en el despliegue.
