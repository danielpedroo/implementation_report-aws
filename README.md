# RELATÓRIO DE IMPLEMENTAÇÃO DE SERVIÇOS AWS

**Data:** 6 de março de 2026

**Empresa:** Abstergo Industries

**Responsável:** Daniel Pedro

---

## 1. Introdução

Este relatório resume a proposta de implementação de três (3) serviços AWS com foco em **redução de custos imediata** e melhoria da eficiência operacional. As recomendações são práticas, de rápida implementação e pensadas para um ambiente corporativo típico.

## 2. Objetivo

* Identificar 3 serviços AWS que gerem redução de custos de curto prazo.
* Descrever o uso, passos de implementação e riscos mínimos.
* Entregar um diagrama simples da arquitetura proposta.

## 3. Serviços escolhidos (resumo)

1. **Amazon S3 (Intelligent‑Tiering + Lifecycle Policies)** — redução de custo de armazenamento.
2. **Amazon EC2 (Spot Instances + Auto Scaling)** — redução de custo de computação para cargas tolerantes a interrupção.
3. **AWS Lambda (serverless) + API Gateway** — migração de jobs/funcionalidades intermitentes para pagar por execução em vez de instâncias permanentes.

---

## 4. Detalhamento dos serviços

### 4.1 Amazon S3 — Intelligent‑Tiering + Lifecycle

* **Foco:** reduzir custos de armazenamento de dados menos acessados sem impactar disponibilidade.
* **Caso de uso:** arquivos de logs, backups, imagens antigas e objetos raramente acessados.
* **Passos de implementação:**

  1. Criar buckets novos ou aplicar políticas aos buckets existentes.
  2. Ativar S3 Intelligent‑Tiering (camadas automática entre frequent/rare/Archive Instant Access).
  3. Configurar Lifecycle rules: transição para S3 Glacier Flexible Retrieval ou Deep Archive após X dias (ex.: 90/180 dias), com exclusão após política definida.
  4. Habilitar versioning apenas quando necessário e configurar expiração para versões antigas.
* **Impacto esperado:** 30–70% de redução em custos de armazenamento para dados frios (varia conforme padrão de acesso).

### 4.2 Amazon EC2 — Spot Instances + Auto Scaling

* **Foco:** reduzir custo de instâncias para workloads tolerantes a interrupção (batch jobs, processamento assíncrono, workers).
* **Caso de uso:** processamento em lote, filas (SQS), workers de imagem, CI/CD agents.
* **Passos de implementação:**

  1. Identificar workloads migráveis para modelos tolerantes a interrupção.
  2. Criar Auto Scaling Groups com mistura de Spot + On‑Demand (ex.: 70% Spot, 30% On‑Demand) e políticas de fallback.
  3. Usar Spot Fleet/Capacity‑optimized allocation strategy para estabilidade.
  4. Integrar com Amazon SQS e Amazon ECS/EKS (se aplicável) para orquestração de tarefas.
  5. Habilitar termination notices handling (2‑minute drain) nos agentes.
* **Impacto esperado:** redução de custo de 50–90% para capacidade Spot; combinação com Auto Scaling reduz ociosidade.

### 4.3 AWS Lambda + API Gateway

* **Foco:** substituir serviços baseados em instâncias para trabalhos esporádicos e APIs de baixa/média demanda.
* **Caso de uso:** endpoints internos/externos com baixa carga, processamento de eventos (S3, SNS), tarefas programadas (EventBridge).
* **Passos de implementação:**

  1. Identificar funções que executam por curtos períodos (<15 min) e com baixa carga concorrente.
  2. Reescrever como funções serverless (Node/Python/Java) e expor via API Gateway ou EventBridge.
  3. Definir limites de memória e tempo de execução otimizados para reduzir custo por execução.
  4. Monitorar latência e custos com CloudWatch; ajustar provisioned concurrency somente se necessário.
* **Impacto esperado:** redução de custo para workloads intermitentes (paga-se por execução). Pode eliminar instâncias dedicadas de baixo uso.

---

## 5. Plano de ação (por etapa)

### Etapa 1 — S3 (Cronograma: 1 semana)

* Tarefas:

  * Inventariar buckets e tipos de dados (responsável: Daniel)
  * Definir políticas de lifecycle (90/180/365 dias)
  * Aplicar Intelligent‑Tiering
  * Validar recuperação de objetos
* Entregáveis: políticas aplicadas, estimativa de custo antes/depois

### Etapa 2 — EC2 Spot + Auto Scaling (Cronograma: 2 semanas)

* Tarefas:

  * Mapear workloads migráveis
  * Configurar ASG com mix Spot/On‑Demand
  * Testar cenário de interrupção e fallback
  * Atualizar runbooks de operação
* Entregáveis: ASG configurado, custos projetados

### Etapa 3 — Lambda (Cronograma: 1–2 semanas)

* Tarefas:

  * Priorizar 2–3 funções candidatas
  * Implementar e testar (staging)
  * Migrar tráfego gradualmente
* Entregáveis: funções em produção, métricas de custo

---

## 6. Requisitos e riscos

* **Requisitos:** conta AWS com permissões IAM para criar recursos; backups antes de alterar lifecycle; ambiente de teste.
* **Riscos:** perda de dados por lifecycle mal configurado (mitigar com testes); interrupções de Spot (mitigar com mix On‑Demand); cold starts no Lambda (mitigar com otimização de pacotes e, se necessário, provisioned concurrency).

---

## 7. Métricas de sucesso (KPIs)

* Redução percentual do custo mensal da conta AWS (meta inicial: −30% no 1º mês para workloads identificadas).
* % de armazenamento movido para camadas frias.
* % de capacidade EC2 migrada para Spot.
* Número de funções / endpoints migrados para Lambda.

---

## 8. Diagrama de arquitetura (visão simplificada)

```mermaid
flowchart LR
  subgraph Front
    API[API Gateway]
    User[Usuários/Clients]
  end
  subgraph Serverless
    Lambda[Functions (Lambda)]
    Event[EventBridge/S3 Events]
  end
  subgraph Compute
    ASG[Auto Scaling Group]
    Spot[EC2 Spot Instances]
    OnDemand[EC2 On‑Demand]
  end
  subgraph Storage
    S3[S3 (Intelligent‑Tiering + Lifecycle)]
    Glacier[Glacier/Deep Archive]
  end

  User --> API --> Lambda
  Event --> Lambda
  Lambda --> S3
  ASG --> Spot
  ASG --> OnDemand
  Spot --> S3
  S3 --> Glacier
  API --> ASG
```

> Observação: o diagrama acima mostra os componentes principais e os fluxos de dados: tráfego de API pode ser atendido por Lambda (serverless) ou por serviços em ASG (Spot/On‑Demand) para cargas que precisam de instância dedicada.

---

## 9. Conclusão

A combinação das três soluções propostas oferece uma estratégia imediata e prática para reduzir custos: otimização de armazenamento (S3), uso eficiente de capacidade de computação (Spot + Auto Scaling) e migração de cargas esporádicas para serverless (Lambda). Recomenda‑se começar pela análise de uso de S3 (impacto mais rápido) e, em seguida, migrar workloads para Spot/Lambda conforme teste.

---

## 10. Anexos (sugestões)

* Runbook de rollout para Spot interruptions
* Planilha de levantamento de buckets e padrões de acesso
* Scripts Terraform/CloudFormation para automatizar as políticas

---

**Assinatura do Responsável pelo Projeto:**

Daniel Pedro
