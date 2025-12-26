# Kubernetes Deployment Guide

## Estrutura de Diretórios

```
k8s/
├── base/                           # Manifestos base do Kubernetes
│   ├── namespace.yaml             # Namespace peo-platform
│   ├── configmap.yaml             # ConfigMaps com configurações
│   ├── secret.yaml                # Secrets (senhas, chaves)
│   ├── sqlserver-deployment.yaml  # SQL Server
│   ├── identity-api-deployment.yaml
│   ├── gestao-conteudo-api-deployment.yaml
│   ├── gestao-alunos-api-deployment.yaml
│   ├── faturamento-api-deployment.yaml
│   ├── bff-api-deployment.yaml
│   └── web-spa-deployment.yaml
└── README.md                      # Este arquivo
```

## Pré-requisitos

- Kubernetes cluster configurado (minikube, kind, ou cluster cloud)
- `kubectl` instalado e configurado
- Imagens Docker publicadas no Docker Hub

## Configuração Inicial

### 1. Atualizar Secrets

**IMPORTANTE**: Antes de aplicar, edite `k8s/base/secret.yaml` e substitua os valores de exemplo:

```yaml
# Edite estas credenciais:
DB_PASSWORD: "sua-senha-forte-aqui"
JWT_SECRET_KEY: "sua-chave-jwt-de-pelo-menos-32-caracteres"
PAYPAL_CLIENT_ID: "seu-client-id-do-paypal"
PAYPAL_SECRET: "seu-secret-do-paypal"
```

### 2. Atualizar Imagens Docker

Em todos os arquivos `*-deployment.yaml`, substitua `<DOCKER_USERNAME>` pelo seu username do Docker Hub:

```bash
# Linux/Mac
find k8s/base -name "*-deployment.yaml" -exec sed -i 's/<DOCKER_USERNAME>/seu-username/g' {} +

# Windows PowerShell
Get-ChildItem k8s/base/*-deployment.yaml | ForEach-Object {
    (Get-Content $_) -replace '<DOCKER_USERNAME>', 'seu-username' | Set-Content $_
}
```

## Deploy no Kubernetes

### Opção 1: Deploy Completo

```bash
# Criar namespace
kubectl apply -f k8s/base/namespace.yaml

# Aplicar configurações (ConfigMaps e Secrets)
kubectl apply -f k8s/base/configmap.yaml
kubectl apply -f k8s/base/secret.yaml

# Deploy do banco de dados primeiro
kubectl apply -f k8s/base/sqlserver-deployment.yaml

# Aguardar o SQL Server estar pronto
kubectl wait --for=condition=ready pod -l app=sqlserver -n peo-platform --timeout=300s

# Deploy das APIs
kubectl apply -f k8s/base/identity-api-deployment.yaml
kubectl apply -f k8s/base/gestao-conteudo-api-deployment.yaml
kubectl apply -f k8s/base/gestao-alunos-api-deployment.yaml
kubectl apply -f k8s/base/faturamento-api-deployment.yaml

# Deploy do BFF e SPA
kubectl apply -f k8s/base/bff-api-deployment.yaml
kubectl apply -f k8s/base/web-spa-deployment.yaml
```

### Opção 2: Deploy com um único comando

```bash
kubectl apply -f k8s/base/
```

## Verificação

### Verificar todos os recursos

```bash
# Ver todos os recursos no namespace
kubectl get all -n peo-platform

# Ver pods
kubectl get pods -n peo-platform

# Ver services
kubectl get services -n peo-platform

# Ver deployments
kubectl get deployments -n peo-platform
```

### Ver logs de um serviço

```bash
# Logs da Identity API
kubectl logs -f deployment/identity-api -n peo-platform

# Logs do SQL Server
kubectl logs -f deployment/sqlserver -n peo-platform
```

### Verificar health checks

```bash
# Port forward para testar localmente
kubectl port-forward -n peo-platform service/identity-api-service 8080:8080

# Em outro terminal, teste o health check
curl http://localhost:8080/health
curl http://localhost:8080/alive
```

## Acesso aos Serviços

### Usando Port Forward

```bash
# BFF API
kubectl port-forward -n peo-platform service/bff-api-service 5005:80

# Web SPA
kubectl port-forward -n peo-platform service/web-spa-service 8080:80

# Identity API
kubectl port-forward -n peo-platform service/identity-api-service 5001:8080
```

### Usando LoadBalancer (se disponível)

```bash
# Obter IP externo do BFF
kubectl get service bff-api-service -n peo-platform

# Obter IP externo do SPA
kubectl get service web-spa-service -n peo-platform
```

## Escalabilidade

### Escalar um serviço

```bash
# Escalar Identity API para 3 réplicas
kubectl scale deployment identity-api --replicas=3 -n peo-platform

# Verificar status
kubectl get pods -l app=identity-api -n peo-platform
```

### Auto-scaling (opcional)

```bash
# Criar HPA para Identity API
kubectl autoscale deployment identity-api \
  --cpu-percent=70 \
  --min=2 \
  --max=5 \
  -n peo-platform
```

## Troubleshooting

### Pod não inicia

```bash
# Descrever o pod
kubectl describe pod <pod-name> -n peo-platform

# Ver eventos do namespace
kubectl get events -n peo-platform --sort-by='.lastTimestamp'
```

### Problemas de conectividade

```bash
# Testar DNS interno
kubectl run -it --rm debug --image=busybox --restart=Never -n peo-platform -- nslookup identity-api-service

# Testar conectividade entre serviços
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -n peo-platform -- curl http://identity-api-service:8080/health
```

### Reiniciar um deployment

```bash
kubectl rollout restart deployment/identity-api -n peo-platform
```

## Limpeza

### Remover todos os recursos

```bash
# Remover todas as deployments e services
kubectl delete -f k8s/base/

# OU remover o namespace inteiro
kubectl delete namespace peo-platform
```

## Minikube Local

Se estiver usando Minikube:

```bash
# Iniciar minikube
minikube start --memory=8192 --cpus=4

# Habilitar métricas
minikube addons enable metrics-server

# Acessar serviços
minikube service web-spa-service -n peo-platform
minikube service bff-api-service -n peo-platform

# Abrir dashboard
minikube dashboard
```

## Kind Local

Se estiver usando Kind:

```bash
# Criar cluster
kind create cluster --name peo-platform

# Carregar imagens locais (se não estiverem no Docker Hub)
kind load docker-image peo-identity-api:latest --name peo-platform
kind load docker-image peo-gestao-conteudo-api:latest --name peo-platform
# ... repetir para outras imagens

# Deploy normalmente
kubectl apply -f k8s/base/
```

## Monitoramento

### Recursos do cluster

```bash
# Ver uso de recursos por namespace
kubectl top pods -n peo-platform

# Ver uso de recursos por node
kubectl top nodes
```

### Logs agregados

```bash
# Logs de todos os pods da Identity API
kubectl logs -l app=identity-api -n peo-platform --tail=100 -f

# Logs de todas as APIs
kubectl logs -l tier=backend -n peo-platform --tail=50
```

## Backup do Banco de Dados

```bash
# Fazer backup do SQL Server
kubectl exec -n peo-platform deployment/sqlserver -- \
  /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "<sua-senha>" -C \
  -Q "BACKUP DATABASE [PeoIdentityDb] TO DISK = N'/var/opt/mssql/backup/identity.bak'"

# Copiar backup para local
kubectl cp peo-platform/sqlserver-pod-name:/var/opt/mssql/backup/identity.bak ./identity.bak
```

## Notas Importantes

1. **Secrets**: Nunca commite secrets reais no Git. Use ferramentas como Sealed Secrets ou External Secrets Operator em produção.

2. **Storage**: Em produção, configure um StorageClass apropriado para o PersistentVolumeClaim do SQL Server.

3. **Resources**: Ajuste os limits e requests de CPU/memória conforme necessário para seu ambiente.

4. **Ingress**: Para produção, considere usar um Ingress Controller ao invés de LoadBalancer direto.

5. **TLS**: Configure certificados TLS para comunicação segura em produção.
