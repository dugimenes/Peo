# Estratégia de Branching - PEO Platform

## Branching Model: GitHub Flow Simplificado

Este projeto utiliza uma variação simplificada do GitHub Flow, adequada para desenvolvimento individual com CI/CD.

## Branches Principais

### `main`
- Branch de produção
- Sempre estável e deployável
- Protegida: requer Pull Request
- Deploy automático para produção (quando configurado)

### `develop`
- Branch de desenvolvimento
- Integração contínua de features
- Ambiente de staging/homologação
- Deploy automático no Docker Hub com tag `develop`

## Workflow de Desenvolvimento

### 1. Criar Feature Branch

```bash
# Sempre partir do develop atualizado
git checkout develop
git pull origin develop

# Criar branch de feature
git checkout -b feature/nome-da-feature
# OU
git checkout -b fix/nome-do-bug
```

### 2. Desenvolver e Commitar

```bash
# Fazer mudanças
git add .
git commit -m "tipo: descrição curta

Descrição detalhada do que foi feito e por quê.

Closes #123"

# Tipos de commit:
# - feat: nova funcionalidade
# - fix: correção de bug
# - docs: apenas documentação
# - style: formatação, ponto e vírgula, etc
# - refactor: refatoração de código
# - test: adicionar testes
# - chore: atualizar dependências, build, etc
```

### 3. Push e Pull Request

```bash
# Push da branch
git push origin feature/nome-da-feature

# No GitHub:
# 1. Abrir Pull Request para develop
# 2. Preencher template do PR
# 3. Aguardar CI/CD passar
# 4. Code review (se aplicável)
# 5. Merge após aprovação
```

### 4. Merge para Main

```bash
# Quando develop estiver estável:
# 1. Criar PR de develop → main
# 2. Aguardar todos os testes
# 3. Merge após aprovação
# 4. Tag de versão
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0
```

## Nomenclatura de Branches

### Features
```
feature/auth-jwt
feature/health-checks
feature/kubernetes-manifests
```

### Fixes
```
fix/docker-build-error
fix/health-endpoint
fix/nswag-release-build
```

### Hotfixes (urgentes em produção)
```
hotfix/security-vulnerability
hotfix/critical-bug
```

### Releases
```
release/v1.0.0
release/v1.1.0
```

## Proteção de Branches (Configurar no GitHub)

### Para `main`:
1. GitHub → Settings → Branches → Add rule
2. Branch name pattern: `main`
3. ✅ Require pull request before merging
4. ✅ Require status checks to pass
   - `.NET Build and Test`
   - `Docker Build and Publish`
5. ✅ Require conversation resolution before merging
6. ✅ Do not allow bypassing the above settings

### Para `develop`:
1. Branch name pattern: `develop`
2. ✅ Require pull request before merging
3. ✅ Require status checks to pass
4. ✅ Allow force pushes (apenas para desenvolvedores)

## Pull Request Template

Criar arquivo `.github/pull_request_template.md`:

```markdown
## Descrição
<!-- Descreva o que foi implementado -->

## Tipo de Mudança
- [ ] Bug fix (correção que resolve um issue)
- [ ] Nova feature (funcionalidade que adiciona valor)
- [ ] Breaking change (mudança que quebra compatibilidade)
- [ ] Documentação

## Checklist
- [ ] Código compila sem erros
- [ ] Testes passam localmente
- [ ] Adicionei testes para minhas mudanças
- [ ] Atualizei a documentação
- [ ] Docker build funciona
- [ ] Testei no Kubernetes (se aplicável)

## Screenshots (se aplicável)
<!-- Cole screenshots aqui -->

## Issues Relacionadas
Closes #
```

## Tags e Versionamento Semântico

### Formato: `v{MAJOR}.{MINOR}.{PATCH}`

- **MAJOR**: Mudanças incompatíveis na API
- **MINOR**: Novas funcionalidades (compatíveis)
- **PATCH**: Correções de bugs

### Exemplos:
```bash
# Primeira release
git tag -a v1.0.0 -m "Initial release with Docker and Kubernetes"

# Novo recurso compatível
git tag -a v1.1.0 -m "Add monitoring with Prometheus"

# Correção de bug
git tag -a v1.1.1 -m "Fix health check endpoint"

# Push tags
git push origin --tags
```

## Automação com GitHub Actions

### Triggers Configurados

**Build e Teste** (`.github/workflows/dotnet.yml`):
- `push` em `main` e `develop`
- `pull_request` em `main` e `develop`

**Docker Build e Push** (`.github/workflows/docker-publish.yml`):
- `push` em `main` e `develop`
- `pull_request` em `main`

### Tags das Imagens Docker

| Branch/Evento | Tags Geradas |
|---------------|--------------|
| `main` | `latest`, `v1.0.0`, `main-{sha}` |
| `develop` | `develop`, `develop-{sha}` |
| `feature/*` | Não publica (apenas testa) |

## Comandos Úteis

```bash
# Ver branches
git branch -a

# Deletar branch local
git branch -d feature/nome

# Deletar branch remota
git push origin --delete feature/nome

# Atualizar develop com main
git checkout develop
git merge main
git push origin develop

# Ver histórico visual
git log --oneline --graph --all

# Ver diferenças entre branches
git diff develop..main
```

## Exemplo de Fluxo Completo

```bash
# 1. Começar nova feature
git checkout develop
git pull origin develop
git checkout -b feature/add-redis-cache

# 2. Desenvolver
# ... fazer mudanças ...
git add .
git commit -m "feat: add Redis caching layer

Implements distributed caching using Redis for improved performance.
Includes:
- Redis connection configuration
- Cache service implementation
- Integration with existing APIs

Closes #45"

# 3. Push e PR
git push origin feature/add-redis-cache
# Abrir PR no GitHub: feature/add-redis-cache → develop

# 4. Após merge no develop, testar
# ... testes em staging ...

# 5. Release para produção
git checkout main
git pull origin main
git merge develop
git tag -a v1.2.0 -m "Release 1.2.0: Redis caching"
git push origin main --tags

# 6. Cleanup
git branch -d feature/add-redis-cache
git push origin --delete feature/add-redis-cache
```

## Boas Práticas

1. ✅ **Commits pequenos e frequentes**
2. ✅ **Mensagens descritivas**
3. ✅ **Testar antes de fazer PR**
4. ✅ **Manter branches atualizadas com develop**
5. ✅ **Deletar branches após merge**
6. ✅ **Usar Pull Requests sempre**
7. ✅ **Code review (quando possível)**
8. ✅ **Resolver conflitos localmente**

## Resolução de Conflitos

```bash
# Atualizar feature branch com develop
git checkout feature/minha-feature
git fetch origin
git rebase origin/develop

# Se houver conflitos:
# 1. Resolver conflitos nos arquivos
# 2. git add <arquivos-resolvidos>
# 3. git rebase --continue

# Force push (apenas em feature branches!)
git push origin feature/minha-feature --force-with-lease
```

---

**Nota**: Este é um projeto individual de MBA, mas as práticas aqui descritas são profissionais e aplicáveis em ambientes de equipe.
