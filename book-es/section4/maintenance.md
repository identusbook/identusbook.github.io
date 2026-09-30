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

Rota las credenciales operativas mediante el mismo proceso que el resto de secretos de producción: `ADMIN_TOKEN`, claves de API de inquilinos, contraseñas de bases de datos, credenciales de Vault, secretos de cliente Keycloak y contraseña de Cardano Wallet.

La rotación de claves DID es una operación distinta a nivel de protocolo. Un controlador `did:prism` rota claves de verificación al publicar una operación DID firmada de **actualización** mediante PRISM Node. Esto cambia los métodos y relaciones de verificación del documento DID en la cadena, como explica el [capítulo @sec-did-and-diddocuments]. Como la emisión y las actualizaciones DID dependen del material de claves derivado de la semilla de billetera, la rotación de claves y la custodia de semillas están estrechamente vinculadas.

> TODO: Marcador de posición para un procedimiento concreto de rotación. Ampliar con pasos y alcance de sus efectos: rotar la semilla de billetera y explicar qué ocurre con los DIDs existentes, rotar claves de verificación `did:prism` mediante actualizaciones sin invalidar credenciales ya emitidas y rotar credenciales AppRole de Vault sin tiempo de inactividad.
