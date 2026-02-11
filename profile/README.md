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

## 👥 Equipe

- **Pedro Luperini Piza** - RM365457
    Discord: @Pedro Luperini - RM365457
- **Rafaela Nascimento Carvalho** - RM366364
    Discord: @Rafaela - RM366364

---

## 🔗 Links

- [Vídeo Demonstração](https://youtube.com/...)
- [Repositório Users](https://github.com/FiapPosTechNett9/tech-challenge-users)
- [Repositório Games](https://github.com/FiapPosTechNett9/tech-challenge-games)
- [Repositório Payments](https://github.com/FiapPosTechNett9/tech-challenge-payments)

---

**FIAP Pós Tech - Fase 4** 🚀
