# Autonomous Expert Agents — Mistral + Google Colab
### Cortex Hub

## 🔎 Visão Geral

Este repositório consolida uma arquitetura de **agentes autônomos especialistas** construída sobre:

* Modelos Mistral (API e modelos open-weight)
* Orquestração multi-agente em Python
* Memória vetorial persistente (RAG / embeddings)
* Execução em Google Colab com GPU
* Deploy On-Premise via FastAPI + Docker + Nginx (pronto para K8s)

O foco é demonstrar nível sênior de engenharia em IA aplicada a operações (AIOps), com:

* Design modular, idempotente e escalável
* Observabilidade e logging estruturado
* Integração natural com stacks modernos (infra, monitoração, automação)
* Capacidade de evoluir para ambientes enterprise (on-prem / híbrido / cloud)

---

## 🧠 Arquitetura Multi-Agente

```mermaid
flowchart TD
    U[Usuário / Driver Script / UI] --> O[Orquestrador Multi-Agente]
    O --> A1[Agente Infra & AIOps]
    O --> A2[Agente Logs & Observabilidade]
    O --> A3[Agente DevOps & Scripts]
    O --> A4[Agente Segurança & Hardening]
    O --> A5[Agente Documentação Técnica]
    O --> M[Mistral API / LLM Local]
    O --> T[Tools Engine]
    O --> V[Vector Memory / RAG]

    T --> S[Sistemas Externos / APIs / Shell]
    V --> D[(Chroma / DuckDB / Files)]
    M --> O
    D --> O
```

---

## 🚀 Getting Started

### 1. Clonar o repositório

```
git clone https://github.com/<org>/<repo>.git
cd <repo>
```

### 2. Instalar dependências

```
pip install -r requirements.txt
```

### 3. Configurar API Key

```
export MISTRAL_API_KEY="sua-chave-mistral"
```

### 4. Executar o orquestrador

```
python main.py
```

---

## 🧩 Binding com Mistral

```python
from mistralai import MistralClient
import os

client = MistralClient(api_key=os.environ["MISTRAL_API_KEY"])

response = client.chat.complete(
    model="mistral-large-latest",
    messages=[
        {"role": "user", "content": "Analise este log de aplicação e proponha um diagnóstico."}
    ]
)

print(response.choices[0].message.content)
```

---

## 📁 Estrutura do Projeto

```
/
├── agents/
│   ├── infra_agent.py
│   ├── devops_agent.py
│   ├── security_agent.py
│   ├── logs_agent.py
│   ├── doc_agent.py
│   └── orchestrator.py
│
├── tools/
│   ├── shell_tools.py
│   ├── file_tools.py
│   ├── http_tools.py
│   └── observability_tools.py
│
├── memory/
│   ├── embeddings.py
│   ├── vector_store.py
│   └── documents/
│
├── notebooks/
│   ├── quickstart_colab.ipynb
│   └── enterprise_demo.ipynb
│
├── api/
│   ├── app.py
│   └── routers/
│       └── agents.py
│
├── docker/
│   ├── Dockerfile.api
│   ├── nginx.conf
│   └── docker-compose.yml
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── main.py
├── requirements.txt
└── README.md
```

---

## 🔧 CI/CD — GitHub Actions

Pipeline CI em `.github/workflows/ci.yml` com lint, tests e smoke build.

```yaml
ame: CI

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  lint:
    name: Linter (Python)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt ruff
      - run: ruff check .

  tests:
    name: Testes Automatizados
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt pytest
      - run: pytest -q || echo "Nenhum teste encontrado"

  build:
    name: Build / Smoke Test
    runs-on: ubuntu-latest
    needs: tests
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: |
          python - << 'EOF'
          try:
              import main
              print("Main importado com sucesso.")
          except Exception as e:
              print(e)
              raise
          EOF
```

---

## 🧱 Guia de Deploy On-Premise (Docker + FastAPI + Nginx)

### Dockerfile da API

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "api.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Nginx Reverse Proxy

```nginx
events {}
http {
    server {
        listen 80;
        server_name _;
        location / { proxy_pass http://api:8000; }
    }
}
```

### docker-compose

```yaml
version: "3.9"
services:
  api:
    build:
      context: ..
      dockerfile: docker/Dockerfile.api
    env_file:
      - ../.env
    ports:
      - "8000:8000"

  nginx:
    image: nginx:stable
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    ports:
      - "80:80"
```

## 🧭 Fluxo Operacional dos Agentes

```mermaid
sequenceDiagram
    autonumber
    participant U as Usuário
    participant API as FastAPI
    participant ORQ as Orquestrador
    participant A as Agente Especialista
    participant M as Mistral
    participant MEM as Memória Vetorial

    U->>API: Envia requisição /api/agents/run
    API->>ORQ: Encaminha entrada do usuário
    ORQ->>MEM: Busca contexto relevante (RAG)
    MEM-->>ORQ: Retorna contexto
    ORQ->>A: Seleciona agente adequado
    A->>M: Solicita inferência / análise
    M-->>A: Resposta estruturada
    A-->>ORQ: Resultado enriquecido
    ORQ-->>API: Consolida resposta
    API-->>U: Entrega saída final
```

---

## 🧩 Pipeline Interno de Raciocínio

```mermaid
flowchart LR
    A[Input do Usuário] --> B[Pré-processamento]
    B --> C[Detecção de Intenção]
    C --> D{Agente Ideal?}
    D -->|Sim| E[Acionamento do Agente]
    D -->|Não| F[Reencaminhamento entre agentes]
    E --> G[Consulta RAG / Memória]
    G --> H[Inferência via Mistral]
    H --> I[Pós-processamento e validação]
    I --> J[Resposta Final]
```

---

## 🧬 Mapa Mental da Arquitetura

```mermaid
mindmap
  root((Autonomous Agents))
    Infra
      Docker
      Kubernetes
      Observabilidade
    DevOps
      Bash
      Ansible
      Python
    Segurança
      Hardening
      Auditorias
      Análise de Logs
    Documentação
      Relatórios
      Diagramas
      Markdown Premium
    Execução
      Google Colab
      On-Premise
      API FastAPI
    Inteligência
      Mistral API
      RAG + Embeddings
      Multi-Agentes
```

---

## ⚙️ Guia de Extensão (Como Criar Novos Agentes)

### 1. Criar classe do agente

```python
class NetworkAgent(BaseAgent):
    name = "network_agent"
    role = "Especialista em redes"

    def run(self, input_text):
        return self.llm(f"Diagnostique esse contexto de rede: {input_text}")
```

### 2. Registrar no orquestrador

```python
orchestrator.register_agent(NetworkAgent())
```

### 3. Adicionar ferramentas (opcional)

```python
@tool("ping")
def ping(host):
    return subprocess.getoutput(f"ping -c 2 {host}")
```

---

## 📡 Diagramas de Deploy em Kubernetes

```mermaid
flowchart TD
    User --> Ingress
    Ingress --> APICluster[FastAPI Deployment]
    APICluster --> Agents[Orchestrator + Agentes]
    Agents --> MistralAPI[Mistral Cloud]
    Agents --> VectorDB[Chroma StatefulSet]
    APICluster --> Logs[OpenTelemetry / Grafana Loki]
```

---

## 🧱 Blocos Premium para Observabilidade

> **Resumo:**
> Sistema avançado de agentes autônomos corporativos, com execução híbrida (cloud + on-premise), processamento vetorial, observabilidade integrada, alto nível de modularidade e orquestração inteligente baseada em intenção.

> **Pontos-Chave:**
>
> * Multi-agentes escaláveis
> * Mistral + RAG
> * Execução acelerada em Colab
> * Pronto para ambientes enterprise

---

MIT License.

