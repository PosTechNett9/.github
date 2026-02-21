# 🎮 FIAP Cloud Games - Tech Challenge Fase 4
## Mensageria, Docker e Kubernetes

[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4)](https://dotnet.microsoft.com/)
[![AWS](https://img.shields.io/badge/AWS-SNS%20%7C%20SQS%20%7C%20Lambda%20%7C%20EKS-FF9900)](https://aws.amazon.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-326CE5)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-Alpine-2496ED)](https://www.docker.com/)

---

## 🎯 Sobre o Projeto

O **FIAP Cloud Games** é uma plataforma de jogos online que enfrentava problemas de escalabilidade. Muitos usuários não conseguiam fazer login e ficavam em fila para iniciar jogos devido ao alto volume de requisições.

Esta **Fase 4** implementa uma solução completa usando:
- **Mensageria assíncrona** (AWS SNS/SQS) para desacoplar microsserviços
- **Containerização otimizada** (Docker Alpine) reduzindo o tamanho das imagens
- **Orquestração** (Kubernetes + HPA) para escalabilidade automática
- **Serverless** (AWS Lambda) para processamento assíncrono de pagamentos

### Problema Resolvido

❌ **Antes:**
- Serviços acoplados, falha em cascata
- Sem escalabilidade horizontal
- Custos fixos independente da carga

✅ **Depois:**
- Comunicação assíncrona, serviços desacoplados
- Escala automaticamente de 2 a 15 pods por serviço
- Custos reduzem automaticamente em baixa demanda

---

## 🏗️ Arquitetura

### Visão Geral

```
┌─────────────┐
│   Cliente   │
└──────┬──────┘
       │
       ▼
┌────────────────────────────────────────────┐
│     AWS EKS - Kubernetes Cluster           │
│                                            │
│  ┌──────────┐  ┌─────────┐  ┌────────────┐ │
│  │  Games   │  │  Users  │  │  Payments  │ │
│  │ (2-15)   │  │ (2-10)  │  │  (2-10)    │ │
│  └────┬─────┘  └────┬────┘  └─────┬──────┘ │
└───────┼─────────────┼─────────────┼────────┘
        │             │             │
        ▼             ▼             ▼
┌───────────────────────────────────────────┐
│       AWS SNS/SQS (Mensageria)            │
│  • auth-requests / auth-responses         │
│  • payment-events                         │
└───────────────┬───────────────────────────┘
                │
                ▼
          ┌──────────┐
          │  Lambda  │
          │ Payments │
          └──────────┘
```
![Fluxo de Comunicação Assíncrona](https://i.imgur.com/qiZV4na.jpeg)

### Microsserviços

1. **Users Service** - Autenticação
2. **Games Service** - API Principal
3. **Payments Service** - Pagamentos

---

## 🛠️ Tecnologias Utilizadas

- **.NET 8, ASP.NET Core, Entity Framework Core**
- **AWS SNS, SQS, Lambda, EKS, RDS**
- **Kubernetes, Docker, HPA**

---

## ✅ Funcionalidades Implementadas

### Requisitos Obrigatórios

- [x] Comunicação Assíncrona (SNS/SQS)
- [x] Eventos para autenticação e pagamentos
- [x] Docker otimizado (Alpine, multi-stage)
- [x] Kubernetes com HPA
- [x] ConfigMaps e Secrets
- [x] APM e monitoramento

--- 
## Kubernets

## ARQUITETURA KUBERNETES (EKS) - AWS

<img width="8192" height="4604" alt="AWS EKS Cluster User-2026-02-21-122521" src="https://github.com/user-attachments/assets/121ea0f3-2196-4557-a4fa-677f08c6c831" />

---

## 📊 LEGENDA DE COMPONENTES

### 🌐 Camada de Internet
- **Usuários:** Clientes web e mobile acessando as APIs

### ☁️ AWS Cloud
- **Região:** us-east-1 (Norte da Virgínia)
- **VPC:** Rede privada virtual isolada

### 🔒 Segurança e Rede
- **Security Group:** Firewall controlando acesso às portas
- **ALB:** Application Load Balancer distribuindo tráfego

### ⚙️ Amazon EKS
- **Control Plane:** Gerenciado pela AWS (API Server, Scheduler, etc)
- **Worker Nodes:** 2× EC2 t3.medium rodando os Pods
- **Namespace production:** Isolamento lógico dos recursos

### 📋 Configurações
- **ConfigMaps (3):** Variáveis de ambiente não-sensíveis
- **Secrets (3):** Credenciais e dados sensíveis (encrypted)

### 🌐 Kubernetes Services
- **NodePort:** Expõe aplicações nas portas 30080, 30081, 30082
- **Load Balancing:** Distribui tráfego entre réplicas dos Pods

### 🔹 Pods (6 total)
- **Users:** 2 réplicas (Ports 30080)
- **Payments:** 2 réplicas (Port 30081)
- **Games:** 2 réplicas (Port 30082)

### 📈 Auto-Scaling
- **HPA:** Escala horizontalmente baseado em CPU (70%) e Memory (80%)
- **Min:** 2 réplicas | **Max:** 10 réplicas por serviço

### 💾 Amazon RDS
- **Users DB:** SQL Server (Port 1433)
- **Payments DB:** SQL Server (Port 1433)
- **Games DB:** PostgreSQL (Port 5432)

### 📬 Mensageria
- **SNS:** Pub/Sub para eventos assíncronos
- **SQS:** Filas para processamento assíncrono
- **Lambda:** Processamento serverless de autenticação

### 🔄 CI/CD Pipeline
- **GitHub:** Repositório de código
- **GitHub Actions:** Automação de build, test e deploy
- **Docker Hub:** Registry de imagens de containers

---

## 🔄 FLUXO DE DADOS

### 1️⃣ Requisição do Usuário
```
Usuário → ALB → Security Group → Service (NodePort) → Pod → RDS
```

### 2️⃣ Resposta ao Usuário
```
RDS → Pod → Service → ALB → Usuário
```

### 3️⃣ Auto-Scaling
```
HPA monitora Pods → Detecta CPU/Memory > threshold → 
Instrui Control Plane → Scheduler cria novos Pods
```

### 4️⃣ Mensageria (Payments)
```
Payments Pod → SNS (payment-events) → Consumidores externos
```

### 5️⃣ Autenticação Assíncrona (Games)
```
Games Pod → SNS (auth-requests) → Lambda → SQS (auth-responses) → Games Pod
```

### 6️⃣ CI/CD
```
Git Push → GitHub Actions → Build/Test → Docker Hub → 
Deploy K8s → Control Plane → Scheduler → Novos Pods
```

---

## 📈 MÉTRICAS E CAPACIDADE

### Recursos por Pod:
| Recurso | Request | Limit |
|---------|---------|-------|
| CPU | 250m (0.25 core) | 500m (0.5 core) |
| Memory | 256Mi | 512Mi |

### Capacidade Total:

**Estado Atual (6 pods):**
- CPU: 1.5 cores (request) / 3 cores (limit)
- Memory: 1.5 GB (request) / 3 GB (limit)

**Máximo Teórico (30 pods):**
- CPU: 7.5 cores (request) / 15 cores (limit)
- Memory: 7.5 GB (request) / 15 GB (limit)

### Worker Nodes:
- **2× EC2 t3.medium**
- 2 vCPU cada = **4 vCPU total**
- 4 GB RAM cada = **8 GB total**

---

## 🎯 CARACTERÍSTICAS DA ARQUITETURA

### ✅ Alta Disponibilidade
- Múltiplas réplicas de cada serviço
- Distribuição em 2 Worker Nodes
- Health checks automáticos

### ✅ Escalabilidade
- Auto-scaling horizontal (HPA)
- Suporta de 6 a 30 pods
- Baseado em métricas reais

### ✅ Segurança
- Secrets encrypted no etcd
- Security Groups controlando acesso
- RDS em subnet privada
- VPC isolada

### ✅ Resiliência
- Self-healing (pods reiniciam automaticamente)
- Load balancing automático
- Rollback em caso de falha

### ✅ Observabilidade
- Health checks (liveness, readiness)
- Métricas de CPU/Memory
- Logs estruturados (Serilog)

---

## 🔐 SEGURANÇA IMPLEMENTADA

1. **Network Security:**
   - VPC isolada
   - Security Groups com regras específicas
   - RDS em subnet privada (sem acesso direto da internet)

2. **Secrets Management:**
   - Secrets encrypted no etcd
   - Nunca commitados no Git
   - Rotação via CI/CD

3. **Container Security:**
   - Images escaneadas com Trivy
   - Alpine Linux (menor superfície de ataque)
   - Non-root user

4. **AWS IAM:**
   - Service accounts com permissões mínimas
   - Session tokens temporários


---

## 👥 Equipe

- **Pedro Luperini Piza** - RM365457
    Discord: @Pedro Luperini - RM365457
- **Rafaela Nascimento Carvalho** - RM366364
    Discord: @Rafaela - RM366364

---

## 🔗 Links

- [Vídeo Demonstração](https://drive.google.com/drive/u/0/folders/1mwYAxmiwkTvTuq2JkvvudlPThPR87KFl)
- [Repositório Users](https://github.com/PosTechNett9/tech-challenge-users)
- [Repositório Games](https://github.com/PosTechNett9/tech-challenge-games)
- [Repositório Payments](https://github.com/PosTechNett9/tech-challenge-payments)

---

**FIAP Pós Tech - Fase 4** 🚀
