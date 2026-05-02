<div align="center">

<br/>

# 🌾 AgriMonitor

### Sistema Integrado de Monitoramento e Gestão de Configuração para Máquinas Agrícolas

<br/>

[![C4 Model](https://img.shields.io/badge/Arquitetura-C4%20Model-4CAF50?style=for-the-badge)](https://c4model.com)
[![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![IoT](https://img.shields.io/badge/Protocol-MQTT%20%2F%20TLS-6A1B9A?style=for-the-badge)](https://mqtt.org)
[![Next.js](https://img.shields.io/badge/Frontend-Next.js-000000?style=for-the-badge&logo=next.js)](https://nextjs.org)
[![Mermaid](https://img.shields.io/badge/Diagramas-Mermaid-FF3670?style=for-the-badge)](https://mermaid.js.org)

<br/>

> **Arquitetura documentada com o C4 Model** — do contexto estratégico ao código, em 4 níveis progressivos de detalhe.

<br/>

</div>

---

## 📖 Sobre o Sistema

O **AgriMonitor** é uma plataforma cloud-native que conecta frotas de máquinas agrícolas — tratores, colheitadeiras, pulverizadores e drones — a um sistema central de monitoramento e controle remoto.

O sistema permite:

- 📡 **Monitoramento em tempo real** — telemetria de sensores (temperatura, RPM, GPS, combustível) transmitida continuamente via MQTT
- 🚨 **Detecção inteligente de anomalias** — análise estatística com baseline histórico e correlação com dados climáticos, eliminando falsos positivos
- ⚙️ **Configuração remota** — perfis de configuração versionados aplicados remotamente nas máquinas do campo
- 🌦️ **Integração meteorológica** — dados climáticos externos para suporte à manutenção preventiva e planejamento operacional

---

## 🗂️ Estrutura do Repositório

```
📁 ponderadaC4/
│
├── 📄 README.md                   ← você está aqui
│
└── 📁 diagrams/
    ├── 🌍 c1-context.md           ← Nível 1: Contexto
    ├── 📦 c2-containers.md        ← Nível 2: Containers
    ├── 🔩 c3-components.md        ← Nível 3: Componentes
    └── 💻 c4-code.md              ← Nível 4: Código
```

---

## 🗺️ O que é o C4 Model?

O **C4 Model** (criado por Simon Brown) é uma abordagem para documentar arquitetura de software em quatro níveis de zoom progressivo — como um Google Maps da sua arquitetura.

```
        ╔═══════════════════════════════════════╗
        ║  🌍  C1 — CONTEXTO                    ║  ← Visão estratégica
        ║  Quem usa? Com o que se integra?       ║     Qualquer stakeholder entende
        ╠═══════════════════════════════════════╣
        ║  📦  C2 — CONTAINERS                  ║  ← Visão técnica de alto nível
        ║  Quais serviços? Que tecnologias?      ║     Tech leads e arquitetos
        ╠═══════════════════════════════════════╣
        ║  🔩  C3 — COMPONENTES                 ║  ← Visão de design
        ║  O que há dentro de cada container?   ║     Desenvolvedores
        ╠═══════════════════════════════════════╣
        ║  💻  C4 — CÓDIGO                      ║  ← Visão de implementação
        ║  Classes, entidades, sequências        ║     Quem vai codar
        ╚═══════════════════════════════════════╝
```

---

## 📐 Os 4 Níveis — Navegação

<table>
  <thead>
    <tr>
      <th>Nível</th>
      <th>Nome</th>
      <th>Pergunta central</th>
      <th>Público</th>
      <th>Link</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><b>C1</b></td>
      <td>🌍 Contexto</td>
      <td>Quem usa o sistema e com quem ele conversa?</td>
      <td>Todos os stakeholders</td>
      <td><a href="./diagrams/c1-context.md">Ver diagrama →</a></td>
    </tr>
    <tr>
      <td align="center"><b>C2</b></td>
      <td>📦 Containers</td>
      <td>Como o sistema é dividido em partes executáveis?</td>
      <td>Tech leads, arquitetos</td>
      <td><a href="./diagrams/c2-containers.md">Ver diagrama →</a></td>
    </tr>
    <tr>
      <td align="center"><b>C3</b></td>
      <td>🔩 Componentes</td>
      <td>O que há dentro de cada container?</td>
      <td>Desenvolvedores</td>
      <td><a href="./diagrams/c3-components.md">Ver diagrama →</a></td>
    </tr>
    <tr>
      <td align="center"><b>C4</b></td>
      <td>💻 Código</td>
      <td>Como as entidades e fluxos críticos se comportam?</td>
      <td>Quem vai implementar</td>
      <td><a href="./diagrams/c4-code.md">Ver diagrama →</a></td>
    </tr>
  </tbody>
</table>

---

## 🛠️ Stack Tecnológica

| Camada | Tecnologia | Por quê |
|---|---|---|
| **Frontend** | Next.js / Vercel | SSR, edge caching global, deploy sem infra |
| **API** | AWS API Gateway + Lambda | Serverless, escala automática, pay-per-request |
| **Ingestão IoT** | AWS Lambda | Event-driven, stateless, escala por invocação |
| **Análise** | AWS Fargate | Stateful, long-running, mantém baseline em memória |
| **Mensageria** | AWS SQS | Buffer entre ingestão e análise, exactly-once delivery |
| **Série Temporal** | Amazon Timestream | TTL automático, compressão nativa, queries otimizadas |
| **Operacional** | Amazon DynamoDB | Lookups por chave, latência de 1 dígito em milissegundos |
| **Protocolo IoT** | MQTT / TLS | Leve, confiável em redes instáveis, publish-subscribe nativo |
| **Meteorologia** | API REST externa | Integração somente leitura, substituível |

---

## 🔑 Princípios de Design

**Desacoplamento por fila** — o Serviço de Ingestão nunca chama o Serviço de Análise diretamente. O SQS absorve picos e garante que nenhuma leitura seja perdida mesmo se o analisador estiver sobrecarregado.

**Banco por finalidade** — Timestream para séries temporais, DynamoDB para entidades de negócio. Cada banco faz o que foi criado para fazer, sem comprometer performance com dados heterogêneos.

**Middleware substituível** — o broker MQTT é externo ao sistema. Pode ser Mosquitto local, AWS IoT Core ou HiveMQ sem impactar nenhuma linha de código do backend.

**Alertas sem duplicação** — o Gerador de Alertas verifica se já existe um alerta ativo do mesmo tipo para a máquina antes de criar outro, evitando spam no dashboard do operador.

---

## 👥 Atores do Sistema

| Ator | Tipo | Interação |
|---|---|---|
| 👨‍🌾 **Operador Agrícola** | Usuário primário | Monitora dashboards, visualiza alertas em tempo real, faz acknowledge |
| 🔧 **Administrador de Configuração** | Usuário técnico | Cria perfis de configuração, aplica remotamente nas máquinas, gerencia usuários |
| 🚜 **Máquinas com IoT** | Sistema externo | Publicam telemetria via MQTT, recebem comandos de configuração |
| 📡 **Middleware MQTT** | Sistema externo | Roteia mensagens entre campo e nuvem |
| 🌦️ **Serviço Meteorológico** | Sistema externo | Fornece contexto climático para análise inteligente |

---

<div align="center">

<br/>

**Comece pelo** [🌍 Diagrama de Contexto](./diagrams/c1-context.md) **e vá descendo os níveis.**

<br/>

Documentado com [C4 Model](https://c4model.com) · Diagramas em [Mermaid](https://mermaid.js.org)

</div>
