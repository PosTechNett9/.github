# FIAP Cloud Games - Tech Challenge Fase 3

## 🏗️ Arquitetura

### Microsserviços

O sistema foi dividido em três microsserviços principais:

#### 1. **Users Service**
- Cadastro de usuários
- Autenticação
- Gerenciamento de perfis

#### 2. **Games Service**
- Listagem de jogos
- Busca avançada (integrado com Elasticsearch)
- Sistema de recomendações
- Processamento de compras

#### 3. **Payments Service**
- Processamento de pagamentos
- Consulta de status de transações
- Emissão de eventos de pagamento

### Componentes da Arquitetura

- **API Gateway**: Ponto de entrada único, gerenciando o acesso entre os 3 microsserviços
- **Elasticsearch**: Busca e indexação de jogos com queries avançadas e agregações
- **Funções Serverless**: Lambda de Processamento de Pagamentos
- **Observabilidade**: Logs centralizados e distributed tracing

## 🔄 Fluxo de Comunicação

```
Cliente → API Gateway → Microsserviços
                ↓
         Roteia para serviço apropriado
                ↓
         Games ←→ Elasticsearch (busca)
         Games → Payments (compra)
         Payments → Fila → Lambda
```

![Diagrama de Funcionamento](https://i.imgur.com/sz91AXU.jpeg)

## 🚀 Tecnologias Utilizadas

- **Cloud Provider**: AWS
- **Banco de Dados**: SQL Server / PostgreSQL
- **Serverless**: AWS Lambda
- **API Gateway**: AWS API Gateway
- **Mensageria**: AWS SQS
- **CI/CD**: GitHub Actions
- **Observabilidade**: Grafana / Prometheus

## 📦 Repositórios

- **Users Service**: https://github.com/FiapPosTechNett9/tech-challenge-users
- **Games Service**: https://github.com/FiapPosTechNett9/tech-challenge-games
- **Payments Service**: https://github.com/FiapPosTechNett9/tech-challenge-payments

## 🛠️ Configuração e Instalação

### Pré-requisitos

- .NET8
- Docker e Docker Compose

### Instalação Local

1. Clone os repositórios:
```bash
git clone https://github.com/FiapPosTechNett9/tech-challenge-users
git clone https://github.com/FiapPosTechNett9/tech-challenge-games
git clone https://github.com/FiapPosTechNett9/tech-challenge-payments
```

2. Configure o Docker Compose

Crie um arquivo `docker-compose.yml` na raiz do projeto (fora das pastas dos microsserviços):

```yaml
version: '3.8'

services:
  # Banco de dados do Users Service
  sqlserver-users:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: fcg-sqlserver-users
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=FCG_Password123!
      - MSSQL_PID=Developer
    ports:
      - "1433:1433"
    volumes:
      - sqlserver-users-data:/var/opt/mssql
    networks:
      - fcg-network
    healthcheck:
      test: /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "FCG_Password123!" -Q "SELECT 1" || exit 1
      interval: 10s
      timeout: 3s
      retries: 10
      start_period: 10s

  # Banco de dados do Games Service
  sqlserver-games:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: fcg-sqlserver-games
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=FCG_Password123!
      - MSSQL_PID=Developer
    ports:
      - "1434:1433"
    volumes:
      - sqlserver-games-data:/var/opt/mssql
    networks:
      - fcg-network
    healthcheck:
      test: /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "FCG_Password123!" -Q "SELECT 1" || exit 1
      interval: 10s
      timeout: 3s
      retries: 10
      start_period: 10s

  # Banco de dados do Payments Service
  sqlserver-payments:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: fcg-sqlserver-payments
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=FCG_Password123!
      - MSSQL_PID=Developer
    ports:
      - "1435:1433"
    volumes:
      - sqlserver-payments-data:/var/opt/mssql
    networks:
      - fcg-network
    healthcheck:
      test: /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "FCG_Password123!" -Q "SELECT 1" || exit 1
      interval: 10s
      timeout: 3s
      retries: 10
      start_period: 10s

  # Elasticsearch para busca de jogos
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: fcg-elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    networks:
      - fcg-network
    healthcheck:
      test: curl -s http://localhost:9200 >/dev/null || exit 1
      interval: 30s
      timeout: 10s
      retries: 50

  # LocalStack para simular AWS SQS localmente
  localstack:
    image: localstack/localstack:latest
    container_name: fcg-localstack
    environment:
      - SERVICES=sqs,lambda
      - DEBUG=1
      - DOCKER_HOST=unix:///var/run/docker.sock
      - AWS_DEFAULT_REGION=us-east-1
    ports:
      - "4566:4566"
      - "4571:4571"
    volumes:
      - localstack-data:/var/lib/localstack
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - fcg-network

  # Seq para visualização de logs
  seq:
    image: datalust/seq:latest
    container_name: fcg-seq
    environment:
      - ACCEPT_EULA=Y
    ports:
      - "5341:80"
    volumes:
      - seq-data:/data
    networks:
      - fcg-network

volumes:
  sqlserver-users-data:
  sqlserver-games-data:
  sqlserver-payments-data:
  elasticsearch-data:
  localstack-data:
  seq-data:

networks:
  fcg-network:
    driver: bridge
```

Execute:
```bash
docker-compose up -d
```

3. Criar a Fila SQS no LocalStack

Após o LocalStack estar rodando, crie a fila SQS:

```bash
# Windows (PowerShell)
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name fcg-payments-queue --region us-east-1

# Linux/Mac
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name fcg-payments-queue --region us-east-1
```

A URL da fila será: `http://localhost:4566/000000000000/fcg-payments-queue`

4. Instale as dependências em cada serviço:
```bash
cd tech-challenge-users
npm install 

cd ../tech-challenge-games
npm install 

cd ../tech-challenge-payments
npm install 
```

5. Execute os serviços:
```bash
# Terminal 1
cd tech-challenge-users
npm run dev

# Terminal 2
cd tech-challenge-games
npm run dev

# Terminal 3
cd tech-challenge-payments
npm run dev
```

## 🎥 Vídeo de Demonstração

[Link]

## 👥 Grupo 10

- **Eduarda Vitória Cunha Matias** - RM366476 - @Eduarda M. ✨ RM366476
- **Matheus Soares Camacho** - RM? - @MatFox
- **Pedro Luperini Piza** - RM365457 - @Pedro Luperini - RM365457
- **Rafaela Nascimento Carvalho** - RM366364 - @Rafaela - RM366364

## 🔗 Links Úteis

- [Repositório Users](https://github.com/FiapPosTechNett9/tech-challenge-users)
- [Repositório Games](https://github.com/FiapPosTechNett9/tech-challenge-games)
- [Repositório Payments](https://github.com/FiapPosTechNett9/tech-challenge-payments)
- [Diagrama de Funcionamento](https://miro.com/welcomeonboard/RmF1dzFJYUsyZktpWXgyMTQ3anQ1TGtySnFnVi9vbkl3azY4R1JrYzRIblM3OWJ0ZFYzSVVyQ1I5S0JyK1hmSnVWcjJOZy9lYmVXTnRpWWo3SThUUGs5UDFPa3BjaFVMQ1dWTXFmVTNHT2FyTTF1akI1aCt5d0Z5WFoydXlkbWpBS2NFMDFkcUNFSnM0d3FEN050ekl3PT0hdjE=?share_link_id=870211838882)
