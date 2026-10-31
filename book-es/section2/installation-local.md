# Instalación: entorno local {#sec-installation-local}

## Descripción general

Hyperledger Identus mantiene el código de sus componentes en repositorios distintos. El repositorio de Cloud Agent contiene el agente de servidor que usa este capítulo. Una aplicación controladora envía solicitudes HTTP a la API REST de Cloud Agent. Cloud Agent administra DIDs, mensajes DIDComm V2, emisión, verificación y almacenamiento de credenciales, y configuración de inquilinos. El README oficial de Cloud Agent lo describe como un agente en la nube W3C/Aries que se ejecuta en el servidor. Las aplicaciones de billetera de los titulares usan los SDK de Edge Agent.

El entorno local ejecuta Cloud Agent con una configuración de desarrollo. Docker inicia Cloud Agent junto con sus servicios de apoyo: PostgreSQL, HashiCorp Vault, APISIX y PRISM Node. La guía oficial de inicio rápido describe esta configuración como de un solo inquilino, con autenticación por clave de API desactivada y un libro mayor en memoria para almacenar DIDs publicados.

Este capítulo inicia dos instancias locales de Cloud Agent:

- `issuer` en el puerto `8000`, para crear DIDs de emisor, esquemas y ofertas de credenciales.
- `verifier` en el puerto `9000`, que usarás más adelante para los flujos de solicitud de pruebas.

Para el capítulo sobre la API REST, puedes ejecutar solo la instancia del emisor. La instancia del verificador resulta útil cuando el libro llega a los flujos de conexiones, emisión y verificación.

## Selección de versiones

Hyperledger Identus no es un único programa con un único número de versión. Cada componente, como Cloud Agent, Mediator, NeoPRISM y cada SDK de Edge Agent, vive en su propio repositorio, evoluciona a su propio ritmo y publica versiones con su propia numeración. Una **versión de Identus Platform** es un conjunto con nombre de versiones concretas de los componentes que los mantenedores de Identus han probado juntas mediante pruebas de integración de extremo a extremo. El número de versión de la plataforma, como `v2.17`, no coincide con el número de versión de ningún componente.

El contenido de este libro es compatible con **Identus Platform `v2.17`**. En el momento de la publicación, esa versión incluía estas versiones de los componentes:

| Componente | Versión en Identus Platform `v2.17` |
|------------|-------------------------------------|
| Cloud Agent | `2.2.0` |
| Mediator | `1.2.1` |
| NeoPRISM | `0.14.1` |
| SDK-TS (TypeScript) | `8.0.0` |
| SDK-Swift | `8.1.1` |
| SDK-KMP (Kotlin Multiplatform) | `5.0.0` |

: Versiones de los componentes en Identus Platform `v2.17` en el momento de la publicación {#tbl-platform-versions}

La versión de la plataforma no fija una versión de PRISM Node. Los ejemplos de este libro combinan Cloud Agent `2.2.0` con PRISM Node `2.6.1`.

Los componentes siguen publicando versiones entre una versión de la plataforma y la siguiente, así que el repositorio de un componente puede mostrar una versión más reciente que la de la tabla anterior. Antes de pasar a una versión más reciente, lee las [notas de versión de Identus Platform](https://github.com/hyperledger-identus/hyperledger-identus/releases) y las notas de versión de ese componente. Las notas de la versión v2.17 también indican que SDK-KMP `5.0.0` no funciona y que se omitieron sus pruebas de integración, así que comprueba el estado actual del SDK de Kotlin antes de construir sobre él.

Este capítulo usa Cloud Agent `2.2.0` y PRISM Node `2.6.1`. Es posible que la guía oficial de inicio rápido todavía muestre un par de versiones anterior. Trata `AGENT_VERSION` y `PRISM_NODE_VERSION` como un par de versiones probado. Si cambias uno de los valores, consulta las notas de versión del componente y repite las comprobaciones de estado al final de este capítulo.

## Requisitos previos

Instala Git, Docker y Docker Compose antes de ejecutar los servicios locales de Cloud Agent. Docker Desktop incluye la CLI de Docker y el complemento Compose en macOS y Windows. En Linux, puedes usar Docker Engine con el complemento Compose.

```bash
git --version
docker --version
docker compose version
```

Cada comando debe mostrar una versión. Si alguno falla, instala o repara esa herramienta antes de iniciar Cloud Agent.

En Windows, usa WSL 2 y ejecuta los comandos de Linux dentro de la distribución WSL. La guía de Microsoft sobre WSL explica cómo instalar una distribución, enumerar las distribuciones instaladas y comprobar si una distribución usa WSL 1 o WSL 2.

## Clonar el repositorio de Cloud Agent

Clona el repositorio actual de Cloud Agent y conserva el nombre de directorio local que usa la guía oficial de inicio rápido:

```bash
git clone https://github.com/hyperledger-identus/cloud-agent identus-cloud-agent
cd identus-cloud-agent
```

Los materiales antiguos de Identus pueden usar `https://github.com/hyperledger/identus-cloud-agent`. Identus Platform `v2.15` documenta el traslado del repositorio a `hyperledger-identus/cloud-agent`.

## Configurar las instancias locales del agente

El directorio `infrastructure/local` contiene los scripts y los archivos de entorno para las ejecuciones locales. El README local indica que los scripts descargan imágenes remotas, `.env` controla las versiones de las imágenes, `run.sh` inicia instancias con nombre y `stop.sh` las detiene.

Crea la configuración del emisor:

```bash
cat > ./infrastructure/local/.env-issuer <<'EOF'
API_KEY_ENABLED=false
AGENT_VERSION=2.2.0
PRISM_NODE_VERSION=2.6.1
PORT=8000
NETWORK=identus
VAULT_DEV_ROOT_TOKEN_ID=root
PG_PORT=5432
EOF
```

Crea la configuración del verificador:

```bash
cat > ./infrastructure/local/.env-verifier <<'EOF'
API_KEY_ENABLED=false
AGENT_VERSION=2.2.0
PRISM_NODE_VERSION=2.6.1
PORT=9000
NETWORK=identus
VAULT_DEV_ROOT_TOKEN_ID=root
PG_PORT=5433
EOF
```

::: {.callout-warning}
`API_KEY_ENABLED=false` desactiva la autenticación por clave de API. Usa esta configuración solo para desarrollo local.
:::

Los valores de `PORT` exponen las dos API REST en puertos distintos del equipo anfitrión. Los valores de `PG_PORT` impiden que los dos contenedores PostgreSQL usen el mismo puerto del anfitrión. Conservamos `NETWORK=identus` de la configuración oficial de inicio rápido; el archivo Compose local actual crea la red de ejecución a partir del nombre del proyecto Compose.

::: {.content-visible when-format="html:js"}
{{< include ../diagrams/d05-local-agent-stacks.html >}}
:::

::: {.content-visible unless-format="html:js"}
![El controlador selecciona una instancia mediante su puerto de API en el anfitrión. Cada proyecto Compose inicia su propio agente, base de datos y servicios de apoyo. PostgreSQL expone su puerto del anfitrión en la interfaz de bucle local; el agente usa el puerto interno de la base de datos.](../diagrams/d05-local-agent-stacks.svg){fig-alt="Dos instancias locales de Cloud Agent"}
:::

Las versiones actuales de Hyperledger Identus publican la imagen de Cloud Agent en Docker Hub como `docker.io/hyperledgeridentus/identus-cloud-agent`. El README local antiguo menciona `GITHUB_TOKEN` y `ghcr.io`; esa nota precede al cambio de registro Docker que documenta Identus Platform `v2.15`.

## Iniciar el Cloud Agent del emisor

Ejecuta el emisor desde la raíz del repositorio. Usa el comando de tu sistema operativo.

macOS:

```bash
./infrastructure/local/run.sh -n issuer -b -w -e ./infrastructure/local/.env-issuer -p 8000 -d "$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}'))"
```

Linux:

```bash
./infrastructure/local/run.sh -n issuer -b -w -e ./infrastructure/local/.env-issuer -p 8000 -d "$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)"
```

La primera ejecución descarga imágenes de contenedores y crea volúmenes Docker. Espera a que Docker indique que los contenedores funcionan correctamente. Después consulta el punto de acceso de estado del emisor:

```bash
curl http://localhost:8000/cloud-agent/_system/health
```

La respuesta debe incluir la versión de Cloud Agent que configuraste:

```json
{"version":"2.2.0"}
```

Cloud Agent sirve la API REST del emisor en `http://localhost:8000/cloud-agent/`. Puedes acceder a Swagger UI en `http://localhost:8000/apidocs/` y al documento OpenAPI en `http://localhost:8000/docs/cloud-agent/api/docs.yaml`.

## Iniciar el Cloud Agent del verificador

Inicia el verificador cuando necesites un segundo actor Cloud Agent para los flujos de solicitud de pruebas y verificación.

macOS:

```bash
./infrastructure/local/run.sh -n verifier -b -w -e ./infrastructure/local/.env-verifier -p 9000 -d "$(ipconfig getifaddr $(route get default | grep interface | awk '{print $2}'))"
```

Linux:

```bash
./infrastructure/local/run.sh -n verifier -b -w -e ./infrastructure/local/.env-verifier -p 9000 -d "$(ip addr show $(ip route show default | awk '/default/ {print $5}') | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)"
```

Consulta el punto de acceso de estado del verificador:

```bash
curl http://localhost:9000/cloud-agent/_system/health
```

Respuesta esperada:

```json
{"version":"2.2.0"}
```

Cloud Agent sirve la API REST del verificador en `http://localhost:9000/cloud-agent/`. Puedes acceder a Swagger UI en `http://localhost:9000/apidocs/` y al documento OpenAPI en `http://localhost:9000/docs/cloud-agent/api/docs.yaml`.

## Detener las instancias locales

Detén cada instancia con el mismo valor de `-n` que usaste al iniciarla:

```bash
./infrastructure/local/stop.sh -n issuer
./infrastructure/local/stop.sh -n verifier
```

Para eliminar los volúmenes Docker locales de una instancia, añade `-d`:

```bash
./infrastructure/local/stop.sh -n issuer -d
./infrastructure/local/stop.sh -n verifier -d
```

Usa `-d` cuando quieras empezar con un estado local limpio. Omítelo cuando quieras conservar las billeteras, los DIDs, los esquemas, los registros de credenciales y otros datos de desarrollo entre reinicios.

## Modelo SSI local

Cada instancia de Cloud Agent actúa como un actor SSI en este entorno local. El emisor controla los DIDs de emisión y firma credenciales. El verificador crea solicitudes de prueba y verifica presentaciones. La billetera del titular entra en el flujo más adelante mediante un SDK de Edge Agent o una aplicación de billetera.

El PRISM Node local ofrece a Cloud Agent un VDR de desarrollo para las operaciones `did:prism`. El libro mayor en memoria permite un entorno rápido que puedes desechar. Los despliegues de producción usan otra configuración para la persistencia, el aislamiento de inquilinos, la autenticación de API, los secretos y el acceso a la red Cardano.
