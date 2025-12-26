# Pull Request

## 📝 Descrição
<!-- Descreva o que foi implementado neste PR -->


## 🏷️ Tipo de Mudança
- [ ] 🐛 Bug fix (correção que resolve um issue)
- [ ] ✨ Nova feature (funcionalidade que adiciona valor)
- [ ] 💥 Breaking change (mudança que quebra compatibilidade)
- [ ] 📚 Documentação
- [ ] 🔧 Configuração/Infraestrutura
- [ ] ♻️ Refatoração

## ✅ Checklist

### Código
- [ ] Código compila sem erros (`dotnet build`)
- [ ] Warnings tratados (`dotnet build -WarnAsError`)
- [ ] Testes passam localmente (`dotnet test`)
- [ ] Code coverage mantido ou melhorado
- [ ] Sem código comentado ou debug statements

### Testes
- [ ] Adicionei testes unitários para minhas mudanças
- [ ] Adicionei testes de integração (se aplicável)
- [ ] Testes cobrem casos de erro

### Docker/Kubernetes
- [ ] Docker build funciona (`docker build -f <Dockerfile> .`)
- [ ] Testei no docker-compose local
- [ ] Manifests Kubernetes atualizados (se aplicável)
- [ ] Health checks funcionam

### Documentação
- [ ] Atualizei README.md (se necessário)
- [ ] Atualizei comentários de código
- [ ] Atualizei documentação de API (Swagger)
- [ ] Atualizei CHANGELOG (se houver)

### DevOps
- [ ] GitHub Actions passam (CI/CD)
- [ ] Não quebrei builds existentes
- [ ] Imagens Docker otimizadas (tamanho)

## 🔗 Issues Relacionadas
Closes #
Fixes #
Related to #

## 📸 Screenshots (se aplicável)
<!-- Cole screenshots, logs ou evidências aqui -->


## 🧪 Como Testar

### Localmente
```bash
# Passos para testar localmente
dotnet run --project src/...
```

### Docker Compose
```bash
docker-compose up -d
# Acessar http://localhost:...
```

### Kubernetes
```bash
kubectl apply -f k8s/base/
kubectl get pods -n peo-platform
```

## 📋 Notas Adicionais
<!-- Qualquer informação adicional relevante -->


## 🔍 Review Checklist (para revisores)
- [ ] Código está limpo e legível
- [ ] Lógica de negócio está correta
- [ ] Não há problemas de segurança
- [ ] Performance é adequada
- [ ] Testes são suficientes
- [ ] Documentação é clara

---

**📌 Nota**: Este PR está pronto para review. Por favor, verifique os itens acima antes de aprovar.
