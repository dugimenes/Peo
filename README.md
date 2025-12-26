# PEO - Plataforma de Educação Online

[![.NET Build and Test](https://github.com/dugimenes/Peo/actions/workflows/dotnet.yml/badge.svg)](https://github.com/dugimenes/Peo/actions/workflows/dotnet.yml)
[![Docker Build and Publish](https://github.com/dugimenes/Peo/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/dugimenes/Peo/actions/workflows/docker-publish.yml)
[![.NET Version](https://img.shields.io/badge/.NET-9.0-512BD4)](https://dotnet.microsoft.com/download/dotnet/9.0)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](docs/BRANCHING-STRATEGY.md)

> Plataforma educacional distribuída desenvolvida com arquitetura de microsserviços, DDD, CQRS e DevOps completo com Docker, Kubernetes e CI/CD.

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Arquitetura](#-arquitetura)
- [Tecnologias](#-tecnologias)
- [Começando](#-começando)
  - [Pré-requisitos](#pré-requisitos)
  - [Quick Start - Docker Compose](#quick-start---docker-compose-recomendado)
  - [Desenvolvimento Local](#desenvolvimento-local---aspire)
  - [Deploy Kubernetes](#deploy-kubernetes)
- [Documentação](#-documentação)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Health Checks](#-health-checks)
- [Testes](#-testes)
- [Troubleshooting](#-troubleshooting)
- [Contribuindo](#-contribuindo)
- [Autor](#-autor)
- [Licença](#-licença)

---

## 🎯 Sobre o Projeto

**PEO** (Plataforma de Educação Online) é um sistema completo de gerenciamento educacional desenvolvido como projeto do **MBA DevXpert Full Stack .NET - Módulo 5** da [Desenvolvedor.IO](https://desenvolvedor.io).

### Objetivos do Projeto

- Implementar uma **plataforma educacional distribuída** com arquitetura de microsserviços
- Aplicar **Domain-Driven Design (DDD)** com bounded contexts bem definidos
- Implementar **CQRS** e **Event-Driven Architecture**
- Criar um **ecossistema DevOps completo** com:
  - ✅ Containerização com **Docker**
  - ✅ Orquestração com **Kubernetes**
  - ✅ Pipeline **CI/CD** com **GitHub Actions**
  - ✅ Observabilidade e **Health Checks**
  - ✅ Resiliência com **Polly**

---

## 🏗️ Arquitetura

### Diagrama de Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                   Web SPA (Blazor WebAssembly)                  │
│                       http://localhost:8080                      │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│              BFF - Backend for Frontend (Gateway)               │
│                     http://localhost:5005                       │
│                    [Orquestração de Chamadas]                   │
└──┬──────────────┬─────────────────┬─────────────────┬──────────┘
   │              │                 │                 │
   │ HTTP         │ HTTP            │ HTTP            │ HTTP
   ▼              ▼                 ▼                 ▼
┌────────┐   ┌──────────┐   ┌────────────┐   ┌──────────────┐
│Identity│   │ Gestão   │   │  Gestão    │   │ Faturamento  │
│  API   │   │Conteúdo  │   │  Alunos    │   │     API      │
│ :5001  │   │   API    │   │    API     │   │    :5004     │
│        │   │  :5002   │   │   :5003    │   │              │
│[Auth/  │   │[Cursos/  │   │[Matrícula/ │   │[Pagamentos/  │
│Users]  │   │ Aulas]   │   │Progresso]  │   │  Eventos]    │
└────┬───┘   └────┬─────┘   └─────┬──────┘   └──────┬───────┘
     │            │               │                  │
     └────────────┴───────────────┴──────────────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │   SQL Server         │
                  │   :1433              │
                  │ [4 Databases]        │
                  └──────────────────────┘
```

### Bounded Contexts (DDD)

| Context | Responsabilidade | Porta | Banco |
|---------|------------------|-------|-------|
| **Identity** | Autenticação, Usuários, JWT | 5001 | PeoIdentityDb |
| **Gestão Conteúdo** | Cursos, Aulas, Materiais | 5002 | PeoGestaoConteudoDb |
| **Gestão Alunos** | Matrículas, Progresso, Certificados | 5003 | PeoGestaoAlunosDb |
| **Faturamento** | Pagamentos, PayPal Integration | 5004 | PeoFaturamentoDb |
| **BFF** | Orquestração, Gateway | 5005 | N/A |
| **Web SPA** | Interface Blazor WASM | 8080 | N/A |

---

## 🚀 Tecnologias

### Backend

| Categoria | Tecnologias |
|-----------|-------------|
| **Framework** | .NET 9.0, ASP.NET Core Web API |
| **Frontend** | Blazor WebAssembly, MudBlazor |
| **Banco de Dados** | SQL Server (prod), SQLite (dev/test) |
| **ORM** | Entity Framework Core 9.0 |
| **Mensageria** | MassTransit + RabbitMQ |
| **Patterns** | CQRS (MediatR), DDD, Repository, Unit of Work |
| **Autenticação** | ASP.NET Identity, JWT Bearer |
| **Resiliência** | Polly (Retry, Circuit Breaker) |
| **Documentação** | Swagger/OpenAPI, NSwag |
| **Observabilidade** | OpenTelemetry, Health Checks |

### DevOps & Infraestrutura

| Categoria | Tecnologias |
|-----------|-------------|
| **Containerização** | Docker, Docker Compose |
| **Orquestração** | Kubernetes (Kind/Minikube/Cloud) |
| **CI/CD** | GitHub Actions |
| **Versionamento** | Git, GitHub Flow |
| **Qualidade** | dotnet format, dotCover, xUnit |
| **Testes** | xUnit, FluentAssertions, NetArchTest |

---

## 🏁 Começando

### Pré-requisitos

Certifique-se de ter instalado:

- ✅ [.NET 9.0 SDK](https://dotnet.microsoft.com/download/dotnet/9.0) ou superior
- ✅ [Docker Desktop](https://www.docker.com/products/docker-desktop) (recomendado)
- ✅ [Git](https://git-scm.com/)
- ⚠️ *Opcional:* [Visual Studio 2022](https://visualstudio.microsoft.com/) ou [VS Code](https://code.visualstudio.com/)
- ⚠️ *Opcional:* [Kubernetes](https://kubernetes.io/docs/setup/) (kubectl, minikube ou kind)

### Quick Start - Docker Compose (Recomendado)

A forma mais rápida de executar toda a plataforma:

```bash
# 1. Clone o repositório
git clone https://github.com/dugimenes/Peo.git
cd Peo

# 2. Inicie todos os serviços com Docker Compose
docker-compose up -d

# 3. Aguarde os containers iniciarem (1-2 minutos)
docker-compose logs -f

# 4. Acesse a aplicação
# Web SPA:        http://localhost:8080
# BFF Swagger:    http://localhost:5005/swagger
# Identity API:   http://localhost:5001/swagger
# Conteúdo API:   http://localhost:5002/swagger
# Alunos API:     http://localhost:5003/swagger
# Faturamento:    http://localhost:5004/swagger
```

**Credenciais de Teste:**
- **Admin:** `admin@admin.com` / `@dmin!`
- **Aluno:** Registre um novo usuário na interface

**Parar os serviços:**
```bash
docker-compose down
# Ou para remover volumes também:
docker-compose down -v
```

### Desenvolvimento Local - Aspire

Para desenvolvimento com hot-reload e debugging:

```bash
# 1. Certifique-se que o Docker está rodando
docker ps

# 2. Inicie apenas o SQL Server
docker-compose up -d sqlserver

# 3. Execute o Aspire AppHost
cd src/Peo.AppHost
dotnet run --launch-profile https

# 4. Acesse o Aspire Dashboard
# URL será exibida no console (geralmente https://localhost:17005)
```

**Portas do Aspire:**
- Dashboard: `https://localhost:17005`
- BFF: `https://localhost:7276`
- Web SPA: `https://localhost:7031`

### Deploy Kubernetes

Para deploy em cluster Kubernetes:

```bash
# 1. Configure secrets e imagens
# Edite k8s/base/secret.yaml e substitua valores
# Edite k8s/base/*-deployment.yaml e substitua <DOCKER_USERNAME>

# 2. Aplique os manifests
kubectl apply -f k8s/base/

# 3. Verifique o deploy
kubectl get all -n peo-platform

# 4. Acesse via port-forward
kubectl port-forward -n peo-platform service/web-spa-service 8080:80

# Ou via LoadBalancer (se disponível)
kubectl get service web-spa-service -n peo-platform
```

📖 **Guia completo:** [k8s/README.md](k8s/README.md)

---

## 📚 Documentação

### Documentação Técnica

- 📘 [Guia de Deploy Kubernetes](k8s/README.md)
- 📗 [Estratégia de Branching](docs/BRANCHING-STRATEGY.md)
- 📙 [Documentação Completa](docs/README.md)
- 📕 [Feedback e Avaliações](FEEDBACK.md)

### APIs - Swagger/OpenAPI

Todas as APIs expõem documentação interativa via Swagger:

| API | Swagger URL (Docker Compose) |
|-----|------------------------------|
| Identity | http://localhost:5001/swagger |
| Gestão Conteúdo | http://localhost:5002/swagger |
| Gestão Alunos | http://localhost:5003/swagger |
| Faturamento | http://localhost:5004/swagger |
| BFF | http://localhost:5005/swagger |

---

## 🔄 CI/CD Pipeline

### Workflows GitHub Actions

#### 1. `.NET Build and Test`

Executa em **push** e **PR** para `main` e `develop`:

```yaml
✅ Restore dependencies
✅ Build (Release, -WarnAsError)
✅ Lint (dotnet format)
✅ Run tests
✅ Generate code coverage (dotCover)
✅ Upload coverage artifact
```

#### 2. `Docker Build and Publish`

Executa em **push** para `main` e `develop`:

```yaml
✅ Build and test (job anterior)
✅ Build Docker images (6 serviços)
✅ Push to Docker Hub
✅ Tag: latest, branch-name, sha
```

### Configurar Secrets

Para o CI/CD funcionar, configure no GitHub:

1. **Settings** → **Secrets and variables** → **Actions**
2. Adicione os secrets:
   - `DOCKER_USERNAME`: seu username do Docker Hub
   - `DOCKER_PASSWORD`: token de acesso do Docker Hub

### Status dos Workflows

[![.NET Build](https://github.com/dugimenes/Peo/actions/workflows/dotnet.yml/badge.svg)](https://github.com/dugimenes/Peo/actions/workflows/dotnet.yml)
[![Docker Build](https://github.com/dugimenes/Peo/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/dugimenes/Peo/actions/workflows/docker-publish.yml)

---

## 📁 Estrutura do Projeto

```
Peo/
├── .github/
│   ├── workflows/              # GitHub Actions CI/CD
│   │   ├── dotnet.yml         # Build, test, lint
│   │   └── docker-publish.yml # Docker build/push
│   └── pull_request_template.md
│
├── docs/                       # Documentação
│   ├── README.md
│   └── BRANCHING-STRATEGY.md
│
├── k8s/                        # Kubernetes manifests
│   ├── base/
│   │   ├── namespace.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   ├── sqlserver-deployment.yaml
│   │   ├── *-api-deployment.yaml (6 serviços)
│   │   └── *.yaml
│   └── README.md
│
├── src/                        # Código-fonte
│   ├── Peo.AppHost/           # .NET Aspire orchestrator
│   ├── Peo.ServiceDefaults/   # Health checks, telemetria
│   │
│   ├── Peo.Core/              # Shared kernel
│   ├── Peo.Core.Web/
│   ├── Peo.Core.Infra.Data/
│   ├── Peo.Core.Infra.ServiceBus/
│   │
│   ├── Peo.Identity.Domain/        # Identity BC
│   ├── Peo.Identity.Application/
│   ├── Peo.Identity.Infra.Data/
│   ├── Peo.Identity.WebApi/
│   │   └── Dockerfile
│   │
│   ├── Peo.GestaoConteudo.Domain/  # Conteúdo BC
│   ├── Peo.GestaoConteudo.Application/
│   ├── Peo.GestaoConteudo.Infra.Data/
│   ├── Peo.GestaoConteudo.WebApi/
│   │   └── Dockerfile
│   │
│   ├── Peo.GestaoAlunos.Domain/    # Alunos BC
│   ├── Peo.GestaoAlunos.Application/
│   ├── Peo.GestaoAlunos.Infra.Data/
│   ├── Peo.GestaoAlunos.WebApi/
│   │   └── Dockerfile
│   │
│   ├── Peo.Faturamento.Domain/     # Faturamento BC
│   ├── Peo.Faturamento.Application/
│   ├── Peo.Faturamento.Infra.Data/
│   ├── Peo.Faturamento.Integrations.Paypal/
│   ├── Peo.Faturamento.WebApi/
│   │   └── Dockerfile
│   │
│   ├── Peo.Web.Bff/           # Backend for Frontend
│   │   └── Dockerfile
│   └── Peo.Web.Spa/           # Blazor WebAssembly
│       └── Dockerfile
│
├── tests/
│   ├── Peo.Tests.UnitTests/
│   ├── Peo.Tests.IntegrationTests/
│   └── Peo.Tests.ArchitectureTests/
│
├── docker-compose.yml          # Orquestração local
├── .dockerignore
├── README.md                   # Este arquivo
├── FEEDBACK.md
├── LICENSE
└── Peo.sln
```

---

## ❤️ Health Checks

Todos os serviços implementam health checks para Kubernetes e Docker:

### Endpoints

- **`/health`**: Verifica saúde completa da aplicação
- **`/alive`**: Liveness probe (aplicação está viva)

### Testando Health Checks

```bash
# Docker Compose
curl http://localhost:5001/health  # Identity API
curl http://localhost:5002/health  # Gestão Conteúdo
curl http://localhost:5003/health  # Gestão Alunos
curl http://localhost:5004/health  # Faturamento
curl http://localhost:5005/health  # BFF

# Kubernetes
kubectl exec -n peo-platform <pod-name> -- curl http://localhost:8080/health
```

### Configuração Kubernetes

Cada deployment tem:
```yaml
livenessProbe:
  httpGet:
    path: /alive
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

---

## 🧪 Testes

### Executar Testes Localmente

```bash
# Todos os testes
dotnet test

# Com verbosidade
dotnet test --logger "console;verbosity=detailed"

# Apenas testes unitários
dotnet test --filter FullyQualifiedName~UnitTests

# Apenas testes de arquitetura
dotnet test --filter FullyQualifiedName~ArchitectureTests
```

### Code Coverage

```bash
# Gerar relatório de cobertura
dotnet tool install --global JetBrains.dotCover.CommandLineTools
dotCover cover-dotnet --output report.html --ReportType HTML --Filters="+:module=Peo.*" -- test

# Abrir relatório
start report.html  # Windows
open report.html   # macOS
xdg-open report.html  # Linux
```

### Coverage via CI/CD

O GitHub Actions gera automaticamente:
1. Acesse **Actions** → último workflow
2. Baixe artifact **code-coverage-report**
3. Extraia e abra `report.html`

### Tipos de Testes

| Projeto | Tipo | Cobertura |
|---------|------|-----------|
| `Peo.Tests.UnitTests` | Testes Unitários | 57 testes |
| `Peo.Tests.ArchitectureTests` | Testes de Arquitetura | 4 testes |
| `Peo.Tests.IntegrationTests` | Testes de Integração | Em desenvolvimento |

---

## 🔧 Troubleshooting

### Docker Compose não inicia

```bash
# Verificar logs
docker-compose logs

# Recriar containers
docker-compose down -v
docker-compose up -d --build

# Verificar portas em uso
netstat -an | findstr "5001 5002 5003 5004 5005 8080 1433"
```

### Kubernetes pods não iniciam

```bash
# Ver logs do pod
kubectl logs -n peo-platform <pod-name>

# Descrever pod (eventos)
kubectl describe pod -n peo-platform <pod-name>

# Ver todos os eventos
kubectl get events -n peo-platform --sort-by='.lastTimestamp'

# Restart deployment
kubectl rollout restart deployment/<name> -n peo-platform
```

### Erro de conexão com banco de dados

```bash
# Verificar se SQL Server está rodando
docker ps | grep sqlserver
# OU
kubectl get pods -n peo-platform | grep sqlserver

# Testar conexão
docker exec -it peo-sqlserver /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P "Peo@2025!Strong" -C -Q "SELECT 1"
```

### Health checks falhando

```bash
# Testar diretamente
curl -v http://localhost:5001/health

# Verificar se MapDefaultEndpoints está no Program.cs
grep "MapDefaultEndpoints" src/Peo.*.WebApi/Program.cs
```

### NSwag falhando no build

O NSwag está configurado para rodar apenas em modo Debug (desenvolvimento local):

```bash
# Build local (Debug) - NSwag roda
dotnet build

# Build Release (Docker/CI) - NSwag não roda
dotnet build --configuration Release
```

---

## 🤝 Contribuindo

Este é um projeto acadêmico individual do MBA DevXpert Full Stack .NET.

### Para Feedback

- 📝 Abra uma [Issue](https://github.com/dugimenes/Peo/issues)
- 📧 Entre em contato com o autor

### Branching Strategy

O projeto segue **GitHub Flow simplificado**. Leia o guia completo:
📗 [Estratégia de Branching](docs/BRANCHING-STRATEGY.md)

Branches principais:
- `main`: produção (estável)
- `develop`: desenvolvimento (integração)
- `feature/*`: novas funcionalidades
- `fix/*`: correções

---

## 👨‍💻 Autor

**Eduardo Gimenes**

- GitHub: [@dugimenes](https://github.com/dugimenes)
- LinkedIn: [Eduardo Gimenes](https://linkedin.com/in/eduardogimenes)
- Email: dugimenes@gmail.com

### Contexto Acadêmico

- **Instituição**: [Desenvolvedor.IO](https://desenvolvedor.io)
- **Curso**: MBA DevXpert Full Stack .NET
- **Módulo**: 5 - DevOps, Docker e Kubernetes
- **Período**: Novembro - Dezembro 2025

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 🙏 Agradecimentos

- **[Desenvolvedor.IO](https://desenvolvedor.io)** - MBA DevXpert Full Stack .NET
- **Eduardo Pires** - Instrutor e mentor
- **Comunidade .NET Brasil** - Suporte e inspiração
- **Microsoft** - Ferramentas e documentação excelentes

---

## 📊 Estatísticas do Projeto

- **Linguagem Principal**: C# (.NET 9.0)
- **Microsserviços**: 6 (Identity, Conteúdo, Alunos, Faturamento, BFF, SPA)
- **Linhas de Código**: ~15.000+
- **Testes**: 61+ (unitários + arquitetura)
- **Cobertura de Código**: 70%+ (objetivo)
- **Imagens Docker**: 6 (otimizadas com Alpine)
- **Manifests K8s**: 10+ (completos)

---

## 🗺️ Roadmap

### ✅ Concluído (Módulo 5)

- [x] Containerização com Docker
- [x] Docker Compose para dev local
- [x] Manifests Kubernetes completos
- [x] Pipeline CI/CD com GitHub Actions
- [x] Health checks em todos os serviços
- [x] Documentação completa
- [x] Resiliência com Polly
- [x] Observabilidade básica

### 🎯 Futuro (Opcional)

- [ ] Monitoramento com Prometheus + Grafana
- [ ] Logging centralizado (ELK Stack / Loki)
- [ ] Service Mesh (Istio / Linkerd)
- [ ] API Gateway com Ingress
- [ ] Certificados SSL/TLS automáticos
- [ ] Autoscaling (HPA/VPA)
- [ ] GitOps com ArgoCD
- [ ] Testes de carga (k6)
- [ ] Análise de segurança (OWASP)
- [ ] Feature Flags

---

<div align="center">

**[⬆ Voltar ao topo](#peo---plataforma-de-educação-online)**

Desenvolvido com ❤️ por [Eduardo Gimenes](https://github.com/dugimenes)

</div>
