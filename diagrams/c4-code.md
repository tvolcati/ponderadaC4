<div align="center">

# 💻 Nível 4 — Código

**"Como as estruturas de dados e os fluxos críticos se comportam?"**

[![C4 Level](https://img.shields.io/badge/C4%20Model-Nível%204%20%7C%20Code-9C27B0?style=flat-square)](../README.md)
[![Zoom](https://img.shields.io/badge/Zoom-Classes%20e%20sequências-lightgrey?style=flat-square)]()

[← C3: Componentes](./c3-components.md) · **C4: Código** · [↑ README](../README.md)

</div>

---

## 🎯 O que este nível mostra

O nível de código é o mais granular do C4 Model. Aqui saímos dos boxes e setas e descemos ao nível de **classes, entidades e fluxos de execução**.

Dois artefatos documentam este nível:

1. **Modelo de Domínio** — as entidades centrais do sistema e seus relacionamentos. É a linguagem ubíqua: a mesma estrutura que vive no banco, na API e no frontend.

2. **Diagrama de Sequência** — o caminho crítico do sistema: um dado sai do sensor de uma máquina e percorre toda a stack até se tornar um alerta visível para o operador.

---

## 4.1 — Modelo de Domínio

> Cinco entidades principais estruturam o sistema. `Machine` é o ator central — ela tem um perfil de configuração ativo, gera leituras de telemetria e, quando algo sai dos limiares, produz alertas enriquecidos com contexto climático.

```mermaid
classDiagram
    direction TB

    class Machine {
        +String machineId
        +String name
        +MachineType type
        +String ownerId
        +MachineStatus status
        +GPS lastLocation
        +DateTime lastSeenAt
        +ConfigProfile activeConfig
        +updateStatus(status MachineStatus)
        +applyConfig(config ConfigProfile)
    }

    class TelemetryReading {
        +String machineId
        +DateTime timestamp
        +Float rpm
        +Float engineTempCelsius
        +Float fuelLevelPercent
        +Float speedKmh
        +GPS location
        +Map~String, Float~ sensorData
        +isWithinThresholds(profile ConfigProfile) bool
    }

    class ConfigProfile {
        +String profileId
        +String name
        +String version
        +Float maxRpm
        +Float maxEngineTempCelsius
        +Float minFuelLevelPercent
        +Map~String, Float~ customThresholds
        +DateTime createdAt
        +String createdByAdminId
    }

    class Alert {
        +String alertId
        +String machineId
        +AlertSeverity severity
        +AlertType type
        +String description
        +String recommendation
        +Float triggeredValue
        +Float threshold
        +WeatherContext weatherContext
        +AlertStatus status
        +DateTime triggeredAt
        +DateTime acknowledgedAt
        +String acknowledgedByUserId
        +acknowledge(userId String)
    }

    class WeatherContext {
        +Float ambientTempCelsius
        +Float windSpeedKmh
        +Float humidityPercent
        +String condition
        +GPS location
        +DateTime capturedAt
    }

    class User {
        +String userId
        +String email
        +UserRole role
        +String organizationId
    }

    class GPS {
        +Float latitude
        +Float longitude
        +Float altitudeMeters
    }

    class MachineType {
        <<enumeration>>
        TRACTOR
        HARVESTER
        SPRAYER
        DRONE
    }

    class MachineStatus {
        <<enumeration>>
        ONLINE
        OFFLINE
        MAINTENANCE
        ALERT
    }

    class AlertSeverity {
        <<enumeration>>
        WARNING
        CRITICAL
    }

    class AlertType {
        <<enumeration>>
        ENGINE_OVERHEAT
        LOW_FUEL
        RPM_EXCEEDED
        SENSOR_FAILURE
        CONNECTIVITY_LOST
        CUSTOM_THRESHOLD
    }

    class AlertStatus {
        <<enumeration>>
        OPEN
        ACKNOWLEDGED
        RESOLVED
    }

    class UserRole {
        <<enumeration>>
        OPERATOR
        ADMIN
    }

    Machine "1" --> "1"     ConfigProfile  : activeConfig
    Machine "1" --> "1"     GPS            : lastLocation
    Machine "1" --> "0..*"  Alert          : generates
    TelemetryReading "1" --> "1" GPS       : location
    Alert "1" --> "1"       WeatherContext : context
    Machine      --> MachineType
    Machine      --> MachineStatus
    Alert        --> AlertSeverity
    Alert        --> AlertType
    Alert        --> AlertStatus
    User         --> UserRole
```

### Entidades principais

| Entidade | Armazenamento | Descrição |
|---|---|---|
| `Machine` | DynamoDB | Representa uma máquina da frota. Chave primária de quase tudo |
| `TelemetryReading` | Timestream | Uma leitura dos sensores num instante de tempo. Gerada continuamente |
| `ConfigProfile` | DynamoDB | Perfil de comportamento aplicado remotamente a uma máquina |
| `Alert` | DynamoDB | Anomalia detectada, com contexto climático e recomendação de ação |
| `WeatherContext` | Embedded no Alert | Snapshot das condições climáticas no momento do alerta |
| `User` | DynamoDB | Operador ou Administrador com acesso à plataforma |

---

## 4.2 — Fluxo de Telemetria (Caminho Crítico)

> Do sensor ao alerta: um trator com motor superaquecendo publica uma leitura de 98°C. Este diagrama mostra cada passo que o sistema dá até o operador ver o alerta vermelho no dashboard.

```mermaid
sequenceDiagram
    autonumber

    participant M  as Máquina (IoT)
    participant BR as Broker MQTT
    participant IN as Serviço de Ingestão
    participant SQ as Fila SQS
    participant AN as Serviço de Análise
    participant TS as Timestream
    participant DY as DynamoDB
    participant WE as Serv. Meteorológico
    participant WB as Portal Web

    M  ->> BR: PUBLISH machines/{id}/telemetry<br/>{rpm: 2400, temp: 98°C, gps: {...}}
    BR ->> IN: Entrega mensagem MQTT

    IN ->> IN: Valida schema e sanitiza payload
    IN ->> TS: INSERT TelemetryReading
    IN ->> DY: UPDATE Machine.status + lastSeenAt
    IN ->> SQ: SendMessage(telemetryEvent)

    Note over SQ, AN: ⏱️ Processamento assíncrono — a ingestão já terminou

    AN ->> SQ: ReceiveMessage (long polling)
    SQ -->> AN: telemetryEvent

    AN ->> TS: Query últimas 60 leituras da máquina
    TS -->> AN: [ ]TelemetryReading (histórico)

    AN ->> AN: Calcula baseline (média ± desvio padrão)
    AN ->> AN: Compara leitura atual com limiares do ConfigProfile

    alt 🚨 Anomalia detectada (temp 98°C > limiar 90°C)
        AN ->> WE: GET /current?lat={lat}&lon={lon}
        WE -->> AN: WeatherContext {temp: 35°C, vento: 12km/h}

        AN ->> DY: Query alertas ENGINE_OVERHEAT ativos para a máquina
        DY -->> AN: [ ] (nenhum alerta ativo)

        AN ->> DY: PUT Alert {severity: CRITICAL, type: ENGINE_OVERHEAT,<br/>triggeredValue: 98, threshold: 90, recommendation: "Parar máquina"}
        DY -->> AN: OK
        AN ->> DY: UPDATE Machine.status = ALERT

    else ✅ Leituras dentro dos limiares
        AN ->> AN: Nenhuma ação necessária
    end

    Note over WB, DY: 👨‍🌾 Operador abre o portal

    WB ->> DY: GET /alerts?status=OPEN&machineId={id}
    DY -->> WB: [ Alert{severity: CRITICAL, ...} ]
    WB ->> WB: Renderiza alerta vermelho no dashboard

    WB ->> DY: PATCH /alerts/{alertId} {status: ACKNOWLEDGED}
    DY -->> WB: Alert{status: ACKNOWLEDGED}
```

### Fases do fluxo

| Fase | Participantes | O que acontece |
|---|---|---|
| **1. Publicação** | Máquina → Broker | Dado bruto do sensor sai do campo via MQTT/TLS |
| **2. Ingestão** | Broker → Lambda → SQS | Valida, persiste no Timestream e enfileira para análise. Rápido e stateless |
| **3. Análise** | SQS → Fargate | Consome assincronamente, calcula baseline, detecta anomalia com contexto climático |
| **4. Alerta** | Fargate → DynamoDB | Persiste alerta enriquecido somente se não há alerta ativo do mesmo tipo |
| **5. Visualização** | Portal → DynamoDB | Operador consulta alertas abertos e faz acknowledge |

---

<div align="center">

[← C3: Componentes](./c3-components.md) · **C4: Código** · [↑ README](../README.md)

</div>
