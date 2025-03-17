# Documentación de la Configuración del Colector OpenTelemetry

Este documento describe la configuración de una instancia del Colector OpenTelemetry (OTel).


### Aquí están los pasos para configurar el binario de OpenTelemetry como un agente:

Download otel-collector tar.gz para su arquitectura.

```
wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.116.0/otelcol-contrib_0.116.0_linux_amd64.tar.gz
```

Extrae otel-collector tar.gz a la carpeta otelcol-contrib

```
mkdir otelcol-contrib && tar xvzf otelcol-contrib_0.121.0_linux_amd64.tar.gz -C otelcol-contrib
```

Crea el archivo config.yaml en la carpeta otelcol-contrib con el siguiente contenido. Reemplaza API_KEY con la clave proporcionada por New Relic.

## Ejecutar en segundo plano

```
./otelcol-contrib --config ./config.yaml &> otelcol-output.log & echo "$!" > otel-pid
```

Para ver la salida de los registros que has configurado para el proceso que se ejecuta en segundo plano:

```
tail -f -n 50 otelcol-output.log
```


### `health_check`

-   **Propósito:** Habilita un punto de control de salud para el colector. Esto permite a sistemas externos verificar el estado operativo del colector.

## Receptores (Receivers)

### `otlp`

-   **Propósito:** Configura el receptor del Protocolo de Línea Abierta de Telemetría (OTLP), permitiendo al colector recibir trazas, métricas y registros a través de gRPC y HTTP.
-   **Configuración:**
    -   `protocols`:
        -   `grpc`: Habilita OTLP sobre gRPC.
        -   `http`: Habilita OTLP sobre HTTP.

### `hostmetrics`

-   **Propósito:** Configura el receptor de métricas del host, que recopila métricas a nivel del sistema.
-   **Configuración:**
    -   `collection_interval`: Especifica el intervalo en el que se recopilan las métricas (20 segundos).
    -   `scrapers`: Define las métricas específicas a recopilar.
        -   `cpu`:
            -   `metrics`:
                -   `system.cpu.utilization`: Habilita la recopilación de métricas de utilización de la CPU.
        -   `load`: Habilita la recopilación de métricas de carga del sistema.
        -   `memory`:
            -   `metrics`:
                -   `system.memory.utilization`: Habilita la recopilación de métricas de utilización de la memoria.
        -   `disk`: Habilita la recopilación de métricas del disco.
        -   `filesystem`:
            -   `metrics`:
                -   `system.filesystem.utilization`: Habilita la recopilación de métricas de utilización del sistema de archivos.
        -   `network`: Habilita la recopilación de métricas de la red.
        -   `paging`:
            -   `metrics`:
                -   `system.paging.utilization`: Habilita la recopilación de métricas de utilización de la paginación.
        -   `processes`: Habilita la recopilación de métricas generales del proceso.
        -   `process`:
            -   `metrics`:
                -   `process.cpu.utilization`: Habilita la recopilación de métricas de utilización de la CPU por proceso.
                -   `process.cpu.time`: Deshabilita la recopilación de métricas de tiempo de la CPU por proceso.
            -   `mute_process_name_error`: Suprime los errores relacionados con la recuperación del nombre del proceso.
            -   `mute_process_exe_error`: Suprime los errores relacionados con la recuperación del ejecutable del proceso.
            -   `mute_process_io_error`: Suprime los errores relacionados con la recuperación de E/S del proceso.
            -   `mute_process_user_error`: Suprime los errores relacionados con la recuperación del usuario del proceso.
            -   `mute_process_cgroup_error`: Suprime los errores relacionados con la recuperación del cgroup del proceso.

### `filelog`

-   **Propósito:** Configura el receptor de registros de archivos, que lee registros de archivos especificados.
-   **Configuración:**
    -   `include`: Especifica la lista de archivos de registro a monitorear.
        -   `/var/log/alternatives.log`
        -   `/var/log/cloud-init.log`
        -   `/var/log/auth.log`
        -   `/var/log/dpkg.log`
        -   `/var/log/syslog`
        -   `/var/log/messages`
        -   `/var/log/secure`
        -   `/var/log/yum.log`

## Procesadores (Processors)

### `transform/truncate`

-   **Propósito:** Configura un procesador de transformación para truncar los valores de los atributos.
-   **Configuración:**
    -   `trace_statements`:
        -   `context`: `span`
        -   `statements`:
            -   `truncate_all(attributes, 4095)`: Trunca todos los atributos de span a una longitud máxima de 4095 caracteres.
            -   `truncate_all(resource.attributes, 4095)`: Trunca todos los atributos de recurso a una longitud máxima de 4095 caracteres.

### `memory_limiter`

-   **Propósito:** Configura un procesador de limitación de memoria para evitar que el colector consuma memoria excesiva.
-   **Configuración:**
    -   `check_interval`: Especifica el intervalo en el que se verifica el uso de memoria (1 segundo).
    -   `limit_mib`: Establece el límite máximo de uso de memoria (1000 MiB).
    -   `spike_limit_mib`: Establece el pico máximo de memoria permitido (200 MiB).

### `batch`

-   **Propósito:** Configura un procesador de lotes para mejorar la eficiencia de la exportación agrupando datos.

### `resourcedetection`

-   **Propósito:** Configura un procesador de detección de recursos para detectar y agregar automáticamente atributos de recursos.
-   **Configuración:**
    -   `detectors`: Especifica los detectores a utilizar (`env`, `system`).
    -   `system`:
        -   `hostname_sources`: Utiliza el sistema operativo para determinar el nombre de host.
        -   `resource_attributes`:
            -   `host.id`: Habilita la detección del ID del host.

### `resource`

-   **Propósito:** Configura un procesador de recursos para agregar o modificar manualmente atributos de recursos.
-   **Configuración:**
    -   `attributes`:
        -   `key`: `host.id`
        -   `value`: `localhost`
        -   `action`: `insert`: Inserta el atributo `host.id` con el valor `localhost`.

## Exportadores (Exporters)

### `otlphttp`

-   **Propósito:** Configura el exportador OTLP HTTP, que envía trazas, métricas y registros a un punto final OTLP a través de HTTP.
-   **Configuración:**
    -   `endpoint`: Especifica la URL del punto final OTLP (`https://otlp.nr-data.net:4317`).
    -   `headers`:
        -   `api-key`: Especifica la clave API para la autenticación (`2ea0316d15ca51d448716c9a43d8f35bFFFFNRAL`).

## Servicio (Service)

### `pipelines`

-   **`metrics`:**
    -   `receivers`: `hostmetrics`
    -   `processors`: `memory_limiter`, `resourcedetection`, `batch`
    -   `exporters`: `otlphttp`
-   **`traces`:**
    -   `receivers`: `otlp`
    -   `processors`: `memory_limiter`, `transform/truncate`, `resourcedetection`, `batch`
    -   `exporters`: `otlphttp`
-   **`logs`:**
    -   `receivers`: `filelog`
    -   `processors`: `batch`
    -   `exporters`: `resourcedetection`, `otlphttp`

### `extensions`

-   `health_check`: Habilita la extensión de control de salud configurada.
