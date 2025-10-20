# Datos

Nombre: Alexis Sebastian Sanchez Ruano

Número de control: 22211659

Fecha: 19 de Octubre del 2025

Nickname: SanchezRuano22211659

# Medición de consumo energético industrial

Proyecto que busca simular la medición del consumo energético de 3 líneas de producción ficticias representadas por tres sensores.

Los datos serán recibidos a través de MQTT y guardados en influxdb para ser representados en Grafana.

Cada sensor medirá los siguientes datos:
- El identificador del sensor
- Hora en la que se envió
- La tensión eléctrica medida
- La corriente eléctrica
- La potencia instantánea
- La energía acumulada a lo largo del tiempo
- La temperatura del equipo

# Arquitectura

## Sensores

Un sensor es representado por un raspberry PI con el código de ejecución para su simulación. El código está trabajado el Rust, generando los datos mencionados y enviando los datos a Mosquitto en su respectivo tópico.

## Mosquitto

Mosquitto es ejecutado en EC2 como el servidor principal, recibiendo los datos de los sensores y posteriormente siendo interceptados por Telegraf.

## Telegraf

Telegraf es encargado de intrceptar los datos publicados por Mosquitto y traducirlos para su inserción en InfluxDB, todo por medios seguros de TLS.

## InfluxDB

InfluxDB guarda todos los datos recibidos de Telegraf de manera consistente y segura.

## Grafana

Es el encargado de usar los datos de InfluxDB para mostrar de manera visual la información, midiendo las métricas de manera casi inmediata.

## Diagrama de la arquitectura general

```mermaid
flowchart LR
    subgraph Sensores["Sensores"]
        S1["Línea de producción A"]
        S2["Línea de producción B"]
        S3["Línea de producción C"]
    end

    subgraph Broker["Mosquitto (EC2)"]
        MQTT["Broker MQTT (Puerto 8883 y TLS)"]
    end

    subgraph Pipeline["Procesamiento de Datos"]
        T["Telegraf (Suscriptor MQTT + TLS)"]
        DB["InfluxDB (Base de datos de series temporales)"]
    end

    subgraph Visualización["Grafana"]
        G["Panel de control - Métricas en tiempo real"]
    end

    S1 -- "Publica datos en tópico industry/energy/line1" --> MQTT
    S2 -- "Publica datos en tópico industry/energy/line2" --> MQTT
    S3 -- "Publica datos en tópico industry/energy/line3" --> MQTT

    MQTT -- "Transmite mensajes MQTT con TLS" --> T
    T -- "Inserta métricas procesadas" --> DB
    DB -- "Provee datos históricos en tiempo real" --> G
```

# Procedimiento
# Pruebas
