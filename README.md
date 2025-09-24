# test-repo
this is a test repo


flowchart TB
  %% Camada Canais
  subgraph C[Experiência / Canais]
    W[Web/App]
    T[Teams/Slack]
    WA[WhatsApp]
    V[Voice/Telefonia]
    A[API Externa]
  end

  %% Camada Copilotos (Domínios)
  subgraph D[Copilotos por Domínio]
    D1[Copiloto Comercial\n- Contexto & Prompt kit\n- Guardrails do domínio\n- Memória curta]
    D2[Copiloto Jurídico]
    D3[Copiloto Dados/Engenharia]
  end

  %% MCP Orquestrador
  subgraph M[MCP Orquestrador (Plataforma)]
    R[Intent Router / Planner]
    P[Policy Engine & Guardrails\n(PII, compliance, limites)]
    MB[Memory Bus\n(curta/longa, por domínio/usuário)]
    Q[Task Queue / Parallel Tools]
    REG[Registry: Agents & Tools]
    OBS[Observability & FinOps\n(Langfuse/Langsmith/traces/custos)]
  end

  %% Especialistas (Agents/Skills)
  subgraph E[Especialistas]
    E1[SQLAgent\n- Tool adapters\n- Skill graph\n- KPIs]
    E2[ChartAgent]
    E3[RetrievalAgent]
    E4[AutomationAgent]
    E5[LegalAgent]
  end

  %% Recursos / Sistemas
  subgraph S[Recursos / Sistemas]
    BQ[(BigQuery)]
    DC[(Dataplex / Policy Tags)]
    AD[(AD/SSO/IAP)]
    API[(APIs Internas CRM/ERP)]
    GCS[(Cloud Storage)]
    PS[(Pub/Sub)]
    SEC[(Secret Manager)]
  end

  %% Fluxos
  W -->|mensagem| D1
  T -->|mensagem| D1
  WA --> D1
  V --> D1
  A --> D1

  D1 -->|intent| R
  R --> P
  P --> MB
  R -->|orquestração| Q
  Q --> REG

  REG --> E1
  REG --> E2
  REG --> E3
  REG --> E4
  REG --> E5

  %% Tools dos especialistas chamando recursos
  E1 -->|query| BQ
  E1 --> DC
  E3 --> GCS
  E4 --> API
  E4 --> PS
  E5 --> AD
  E5 --> SEC

  %% Observabilidade
  R -. trace .-> OBS
  E1 -. métricas .-> OBS
  E3 -. custos .-> OBS
  D1 -. eventos .-> OBS

  %% Retorno
  E1 -->|result| Q --> D1
  D1 -->|resposta| W


sequenceDiagram
  autonumber
  participant User as Usuário (Teams)
  participant Cop as Copiloto Comercial
  participant MCP as MCP Orquestrador
  participant Pol as Policy/Guardrails
  participant Mem as Memory Bus
  participant Reg as Registry
  participant SQL as SQLAgent
  participant BQ as BigQuery
  participant Obs as Observability

  User->>Cop: Pergunta (ex: "vendas por produto no mês")
  Cop->>MCP: Intent + contexto do domínio
  MCP->>Pol: Checagem de política (PII, limites)
  Pol-->>MCP: OK/Redações/Masking
  MCP->>Mem: Recupera memória curta/longa
  MCP->>Reg: Seleciona especialista (SQLAgent)
  MCP->>SQL: Plano de execução + constraints
  SQL->>BQ: Query parametrizada (policy tags/row-level)
  BQ-->>SQL: Resultado tabular
  SQL-->>MCP: Dataset + sumarização
  MCP->>Obs: Trace + custo + latência
  MCP-->>Cop: Resposta + dados + (opcional: gráfico)
  Cop-->>User: Mensagem final formatada