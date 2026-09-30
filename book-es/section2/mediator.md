# Mediador {#sec-mediator}

## Descripción general

Hyperledger Identus Mediator es un servicio de enrutamiento DIDComm V2. Ofrece a las billeteras móviles, las billeteras de navegador y otros agentes con conectividad variable un punto de acceso estable donde otros agentes pueden enviar mensajes DIDComm cifrados.

En un flujo DIDComm con mediación, la billetera del titular establece primero la mediación con el mediador. El mediador devuelve información de enrutamiento. La billetera del titular anuncia después esa información en las entradas de servicio DIDComm y los registros de conexión posteriores. Un emisor o verificador que envía un mensaje DIDComm a ese titular cifra el mensaje para el titular, lo envuelve en un mensaje DIDComm `forward` para el mediador y envía el mensaje envuelto al punto de acceso del mediador. El mediador puede descifrar la envoltura externa de enrutamiento, pero no puede leer el cuerpo cifrado del mensaje del titular.

El mediador almacena los mensajes pendientes del titular en MongoDB. La billetera del titular usa después DIDComm Message Pickup para consultar el estado de la cola, solicitar la entrega y confirmar los mensajes recibidos. Este modelo permite que un emisor o verificador envíe mensajes cuando la billetera del titular está desconectada, sin revelar la dirección del dispositivo del titular a cada conexión.

El mediador no es un registro de datos verificables y no publica DIDs. Administra el transporte y el almacenamiento DIDComm. PRISM Node administra las operaciones `did:prism` respaldadas por el registro distribuido. Cloud Agent y los SDK de Edge Agent usan el mediador cuando su contraparte DIDComm necesita un punto de retransmisión estable.

## Protocolos disponibles

El repositorio actual de Mediator documenta la compatibilidad con estos protocolos DIDComm:

- `https://didcomm.org/routing/2.0`
- `https://didcomm.org/coordinate-mediation/2.0`
- `https://didcomm.org/messagepickup/3.0`
- `https://didcomm.org/trust-ping/2.0`
- `https://didcomm.org/report-problem/2.0`
- `https://didcomm.org/basicmessage/2.0`

Para la configuración, los protocolos importantes son Coordinate Mediation, Routing y Message Pickup. Coordinate Mediation permite que una billetera receptora solicite mediación y registre claves de destinatario. Routing define el mensaje `forward` que envía tráfico a través del mediador. Message Pickup permite que la billetera receptora recupere los mensajes en cola.

::: {.content-visible when-format="html:js"}
{{< include ../diagrams/d07-mediation-delivery.html >}}
:::

::: {.content-visible unless-format="html:js"}
![El mediador conserva los mensajes en cola hasta que la billetera confirma sus identificadores. Las solicitudes de recogida usan return_route: all para que el mediador pueda responder mediante la conexión de la billetera. El diagrama muestra la entrega después de un período sin conexión.](../diagrams/d07-mediation-delivery.svg){fig-alt="Configuración de mediación y entrega tras una desconexión"}
:::

## Selección de versión

Consulta ambas líneas de versiones antes de fijar una imagen de mediador. La versión actual `v2.16` de Identus Platform incluye Mediator `1.2.0`. El repositorio independiente de Mediator indica `v1.2.1` como la versión estable más reciente de Mediator.

Este capítulo usa `MEDIATOR_VERSION=1.2.1` para la configuración del mediador independiente. Si reproduces el conjunto completo de componentes de Identus Platform `v2.16`, usa `MEDIATOR_VERSION=1.2.0`.

## Requisitos previos

Instala Git, Docker y Docker Compose antes de ejecutar el entorno del mediador.

```bash
git --version
docker --version
docker compose version
```

Cada comando debe mostrar una versión. Si un comando falla, instala o repara esa herramienta antes de iniciar el mediador.

## Clonar el repositorio del mediador

Clona el repositorio actual:

```bash
git clone https://github.com/hyperledger-identus/mediator identus-mediator
cd identus-mediator
```

El material antiguo de Identus puede usar `https://github.com/hyperledger/identus-mediator`. Los repositorios actuales de Identus pertenecen a la organización `hyperledger-identus` de GitHub.

## Archivos de configuración local

El repositorio oficial ya incluye `docker-compose.yml` e `initdb.js`. No necesitas crear ninguno de los dos archivos cuando ejecutas el repositorio clonado. Lee ambos archivos antes de cambiar el despliegue del mediador. Esos archivos definen la imagen, la base de datos, las claves de identidad, los puntos de acceso públicos DIDComm y la comprobación de estado.

El archivo Compose local actual tiene esta estructura:

```yaml
services:
  mongo:
    image: mongo:6.0
    ports:
      - "27017:27017"
    command: ["--auth"]
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=admin
      - MONGO_INITDB_DATABASE=mediator
    volumes:
      - ./initdb.js:/docker-entrypoint-initdb.d/initdb.js

  identus-mediator:
    image: docker.io/hyperledgeridentus/identus-mediator:${MEDIATOR_VERSION:-latest}
    ports:
      - "8080:8080"
    environment:
      - KEY_AGREEMENT_D=Z6D8LduZgZ6LnrOHPrMTS6uU2u5Btsrk1SGs4fn8M7c
      - KEY_AGREEMENT_X=Sr4SkIskjN_VdKTn0zkjYbhGTWArdUNE4j_DmUpnQGw
      - KEY_AUTHENTICATION_D=INXCnxFEl0atLIIQYruHzGd5sUivMRyQOzu87qVerug
      - KEY_AUTHENTICATION_X=MBjnXZxkMcoQVVL21hahWAw43RuAG-i64ipbeKKqwoA
      - SERVICE_ENDPOINTS=${SERVICE_ENDPOINTS:-http://localhost:8080;ws://localhost:8080/ws}
      - MONGODB_USER=admin
      - MONGODB_PASSWORD=admin
      - MONGODB_PROTOCOL=mongodb
      - MONGODB_HOST=mongo
      - MONGODB_PORT=27017
      - MONGODB_DB_NAME=mediator
    depends_on:
      - mongo
    healthcheck:
      test: curl --fail http://localhost:8080/health || exit 1
      interval: 30s
      timeout: 8s
      retries: 3
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

El servicio `mongo` almacena cuentas de mediación y mensajes en cola. El archivo local expone MongoDB en el puerto `27017` del host para depuración. Las billeteras y los Cloud Agents no necesitan ese puerto del host. Los despliegues de producción deben mantener MongoDB dentro de la red privada del despliegue.

El servicio `identus-mediator` ejecuta la imagen del mediador de Docker Hub. `MEDIATOR_VERSION` selecciona la etiqueta de la imagen. La asignación `ports` expone el servicio HTTP y WebSocket del mediador en el puerto `8080` del host. El mediador usa `SERVICE_ENDPOINTS` cuando construye su DID de pares y su invitación fuera de banda. Por tanto, ese valor debe indicar un punto de acceso al que pueda conectarse la billetera del titular.

Los valores de clave de demostración del archivo Compose local crean una identidad estable del mediador para el desarrollo local. No reutilices esas claves para un mediador compartido o de producción. Genera un nuevo par de claves X25519 para el acuerdo de claves y un nuevo par de claves Ed25519 para la autenticación. Proporciona después los cuatro campos JWK mediante la configuración del despliegue.

El archivo de inicialización de Mongo crea el usuario de base de datos, las colecciones, los índices y la regla TTL:

```js
db.createUser({
    user: "admin",
    pwd: "admin",
    roles: [
        { role: "readWrite", db: "mediator" }
    ]
});

const database = 'mediator';
const collectionDidAccount = 'user.account';
const collectionMessages = 'messages';
const collectionMessagesSend = 'messages.outbound';

use(database);

db.createCollection(collectionDidAccount);
db.createCollection(collectionMessages);
db.createCollection(collectionMessagesSend);

db.getCollection(collectionDidAccount).createIndex({ 'did': 1 }, { unique: true });
db.getCollection(collectionDidAccount).createIndex({ 'alias': 1 }, { unique: true, partialFilterExpression: { "alias.0": { $exists: true } } });
db.getCollection(collectionDidAccount).createIndex({ "messagesRef.hash": 1, "messagesRef.recipient": 1 });

const expireAfterSeconds = 7 * 24 * 60 * 60;
db.getCollection(collectionMessages).createIndex(
    { ts: 1 },
    {
        name: "message-ttl-index",
        partialFilterExpression: { "message_type" : "Mediator" },
        expireAfterSeconds: expireAfterSeconds
    }
)
```

`user.account` almacena registros de cuentas de mediación con el DID como clave. `messages` almacena mensajes de protocolo del mediador y referencias a cargas útiles de usuarios en cola. `messages.outbound` almacena registros de mensajes salientes. El índice TTL selecciona registros cuyo `message_type` es `Mediator`; los mensajes de usuarios en cola permanecen hasta que el destinatario los recupera y confirma su recepción.

Un archivo `.env` local puede fijar la imagen del mediador y sus puntos de acceso públicos:

```bash
MEDIATOR_VERSION=1.2.1
SERVICE_ENDPOINTS=http://localhost:8080;ws://localhost:8080/ws
```

Usa una dirección IP de la LAN o un dominio público en `SERVICE_ENDPOINTS` cuando una billetera móvil deba conectarse desde fuera de la máquina anfitriona. Mantén el esquema de URL y el puerto en correspondencia con el punto de acceso que expones mediante Docker, un proxy inverso o un balanceador de carga.

## Iniciar el mediador

El repositorio incluye el archivo local `docker-compose.yml`. Este inicia dos servicios:

- `mongo`, la base de datos MongoDB para cuentas y mensajes en cola.
- `identus-mediator`, el servicio de mediador JVM expuesto en el puerto `8080`.

El archivo Compose actual asigna el puerto `8080` del host al contenedor del mediador. Comprueba que ningún otro servicio local use ese puerto antes del inicio:

```bash
docker ps --format '{{.Names}} {{.Ports}}' | grep 8080
```

Si el comando muestra otro contenedor que usa `8080`, detén ese servicio o adapta la asignación de puertos de Compose antes de iniciar el mediador. Mantén el puerto expuesto del host en correspondencia con `SERVICE_ENDPOINTS`, porque las billeteras leen `SERVICE_ENDPOINTS` de la invitación DIDComm del mediador.

Ejecuta el comando para tu sistema operativo desde la raíz del repositorio.

macOS:

```bash
MEDIATOR_VERSION=1.2.1 SERVICE_ENDPOINTS="http://$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}')):8080;ws://$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}')):8080/ws" docker compose up -d
```

Linux:

```bash
MEDIATOR_VERSION=1.2.1 SERVICE_ENDPOINTS="http://$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1):8080;ws://$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1):8080/ws" docker compose up -d
```

El valor de `SERVICE_ENDPOINTS` pasa a formar parte de las entradas de servicio del DID de pares y de la invitación fuera de banda del mediador. Usa una dirección a la que pueda conectarse la billetera. Una billetera de navegador que se ejecuta en el mismo host puede usar `localhost`; una billetera móvil en la misma red necesita la dirección IP de la LAN de la máquina anfitriona o un dominio público. Para despliegues de producción, usa puntos de acceso HTTPS y WSS.

## Comprobar el mediador en ejecución

Comprueba los contenedores:

```bash
docker compose ps
```

El servicio del mediador define una comprobación de estado de Docker contra `/health`. Después del inicio, Docker debe indicar que `identus-mediator` está en buen estado.

Comprueba el punto de acceso de estado:

```bash
curl -i http://localhost:8080/health
```

Resultado esperado:

```text
HTTP/1.1 200 OK
```

Comprueba la versión del mediador en ejecución:

```bash
curl http://localhost:8080/version
```

Resultado esperado:

```text
1.2.1
```

Obtén el DID de pares del mediador:

```bash
curl http://localhost:8080/did
```

La respuesta debe comenzar con `did:peer:`. El software de la billetera del titular usa este DID como DID del mediador cuando inicia la mediación.

Obtén la invitación de mediación:

```bash
curl http://localhost:8080/invitation
```

La respuesta es una invitación DIDComm fuera de banda. Su campo `from` contiene el DID de pares del mediador. Su cuerpo debe incluir `goal_code` con el valor `request-mediate` y `accept` con `didcomm/v2`.

Para obtener la invitación codificada para URL que usan los flujos con códigos QR, llama a:

```bash
curl http://localhost:8080/invitationOOB
```

La respuesta debe contener un parámetro `_oob=`.

## Notas de configuración

El archivo Compose local incluye claves de identidad de demostración. No las uses fuera del desarrollo local. Un mediador de producción necesita su propia clave de acuerdo de claves X25519 y su clave de autenticación Ed25519. El repositorio de Mediator incluye una guía para generar los valores `KEY_AGREEMENT_D`, `KEY_AGREEMENT_X`, `KEY_AUTHENTICATION_D` y `KEY_AUTHENTICATION_X` en un formato compatible con JWK.

Puedes configurar MongoDB con variables individuales:

```text
MONGODB_PROTOCOL
MONGODB_HOST
MONGODB_PORT
MONGODB_USER
MONGODB_PASSWORD
MONGODB_DB_NAME
```

Mediator `1.2.0` añadió `MONGODB_CONNECTION_STRING`. Si está presente, esta cadena de conexión completa tiene prioridad sobre las variables individuales de MongoDB.

El mediador almacena dos categorías de mensajes. Los mensajes `Mediator` cubren la configuración de mediación, las operaciones de listas de claves, las solicitudes de recogida y otro tráfico de protocolo del mediador. Estos registros pueden caducar mediante el índice TTL que configura `initdb.js`. Los mensajes `User` son cargas útiles cifradas que esperan la recogida del titular. El mediador los conserva hasta que el destinatario los recupera y confirma su recepción mediante Message Pickup.

## Detener el mediador

Detén el entorno sin eliminar el volumen de la base de datos:

```bash
docker compose down
```

Elimina los contenedores y los datos locales de MongoDB:

```bash
docker compose down -v
```

Usa `-v` cuando quieras un estado limpio del mediador local. Omítelo cuando quieras conservar las cuentas de mediación registradas y los mensajes de desarrollo en cola entre reinicios.
