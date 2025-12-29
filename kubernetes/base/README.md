# Kubernetes/OpenShift Manifests para Migration Tracker

Este diretório contém os manifestos YAML necessários para deploy da aplicação Migration Tracker no OpenShift/Kubernetes.

## Estrutura

```
kubernetes/
├── base/                    # Manifests base (comuns a todos os ambientes)
│   ├── configmap.yml        # Configurações da aplicação
│   ├── secret.yml           # Secrets (senhas, tokens)
│   ├── serviceaccount.yml   # ServiceAccount para OpenShift
│   ├── deployment.yml       # Deployment da aplicação API
│   ├── service.yml          # Service para expor a API
│   ├── route.yml            # Route do OpenShift (equivalente a Ingress)
│   ├── postgres-deployment.yml  # Deployment do PostgreSQL
│   ├── postgres-service.yml     # Service do PostgreSQL
│   ├── postgres-pvc.yml         # PersistentVolumeClaim para dados do PostgreSQL
│   ├── init-db-job.yml      # Job opcional para inicializar o banco
│   └── kustomization.yml    # Configuração do Kustomize
└── overleys/
    ├── prod/                # Overlay para produção
    │   ├── kustomization.yml
    │   ├── deployment-patch.yml
    │   └── route-patch.yml
    └── uat/                 # Overlay para UAT (User Acceptance Testing)
        ├── kustomization.yml
        ├── deployment-patch.yml
        └── route-patch.yml
```

## Pré-requisitos

1. Cluster OpenShift/Kubernetes configurado
2. `kubectl` ou `oc` CLI instalado e configurado
3. Acesso ao registry de imagens (quay.io/lagomes/migration-tracker-api)
4. Permissões para criar Deployments, Services, Routes, PVCs, etc.

## Configuração

### 1. Secrets

**IMPORTANTE**: Antes do deploy, ajuste os valores em `secret.yml`:

```yaml
stringData:
  SESSION_SECRET: "uma-string-bem-longa-e-aleatoria-fixa-para-prod"  # Gere um valor seguro
  DB_PASSWORD: "1q2w3e"  # Altere para uma senha forte
```

### 2. ConfigMap

Ajuste os valores em `configmap.yml` conforme necessário, especialmente:
- `DB_HOST`: Nome do service do PostgreSQL (já configurado como `postgres-service`)
- `DB_MAX_OPEN_CONNS` e `DB_MAX_IDLE_CONNS`: Ajuste conforme número de réplicas

### 3. Route (OpenShift)

No overlay de produção (`overleys/prod/route-patch.yml`), ajuste o hostname:

```yaml
spec:
  host: migration-tracker.example.com  # Altere para seu domínio
```

## Deploy

### Opção 1: Usando Kustomize (Recomendado)

```bash
# Deploy base (desenvolvimento)
kubectl apply -k kubernetes/base

# Deploy UAT
kubectl apply -k kubernetes/overleys/uat

# Deploy produção
kubectl apply -k kubernetes/overleys/prod
```

### Opção 2: Usando oc/kubectl diretamente

```bash
# Criar namespace (se não existir)
oc new-project migration-tracker

# Aplicar recursos base
oc apply -f kubernetes/base/configmap.yml
oc apply -f kubernetes/base/secret.yml
oc apply -f kubernetes/base/serviceaccount.yml
oc apply -f kubernetes/base/postgres-pvc.yml
oc apply -f kubernetes/base/postgres-deployment.yml
oc apply -f kubernetes/base/postgres-service.yml

# Aguardar PostgreSQL estar pronto
oc wait --for=condition=ready pod -l app=migration-tracker-postgres --timeout=300s

# Inicializar banco de dados (opcional, mas recomendado na primeira vez)
oc apply -f kubernetes/base/init-db-job.yml
oc wait --for=condition=complete job/migration-tracker-init-db --timeout=300s

# Aplicar aplicação
oc apply -f kubernetes/base/deployment.yml
oc apply -f kubernetes/base/service.yml
oc apply -f kubernetes/base/route.yml
```

### Opção 3: Preview com Kustomize

Para ver os YAMLs gerados antes de aplicar:

```bash
kubectl kustomize kubernetes/base > base-rendered.yaml
kubectl kustomize kubernetes/overleys/uat > uat-rendered.yaml
kubectl kustomize kubernetes/overleys/prod > prod-rendered.yaml
```

## Inicialização do Banco de Dados

Na primeira vez, é necessário inicializar o schema do banco:

```bash
# Opção 1: Usar o Job
oc apply -f kubernetes/base/init-db-job.yml
oc logs -f job/migration-tracker-init-db

# Opção 2: Executar manualmente em um pod
oc run migration-tracker-init --image=quay.io/lagomes/migration-tracker-api:latest \
  --restart=Never --rm -it -- \
  ./admin-cli -init_db

# Opção 3: Entrar em um pod da aplicação e executar
oc exec -it deployment/migration-tracker-api -- ./admin-cli -init_db
```

## Verificação

```bash
# Verificar status dos pods
oc get pods -n migration-tracker

# Verificar logs da aplicação
oc logs -f deployment/migration-tracker-api

# Verificar logs do PostgreSQL
oc logs -f deployment/migration-tracker-postgres

# Verificar rotas
oc get route migration-tracker-api

# Testar endpoints de health
oc get route migration-tracker-api -o jsonpath='{.spec.host}'
curl http://$(oc get route migration-tracker-api -o jsonpath='{.spec.host}')/healthz
curl http://$(oc get route migration-tracker-api -o jsonpath='{.spec.host}')/readyz
```

## Health Checks

A aplicação expõe os seguintes endpoints para monitoramento:

- `/healthz` - Liveness probe (verifica se o container está vivo)
- `/readyz` - Readiness probe (verifica se pode receber tráfego, testa conexão com DB)
- `/metrics` - Métricas Prometheus

## Troubleshooting

### Pod não inicia

```bash
# Ver eventos
oc describe pod <pod-name>

# Ver logs
oc logs <pod-name>

# Ver logs anteriores (se o pod reiniciou)
oc logs <pod-name> --previous
```

### Problemas de conexão com banco

1. Verifique se o PostgreSQL está rodando: `oc get pods -l app=migration-tracker-postgres`
2. Verifique o service: `oc get svc postgres-service`
3. Teste a conectividade: `oc exec -it deployment/migration-tracker-api -- ping postgres-service`

### PVC não é criado

No OpenShift, pode ser necessário ajustar o `storageClassName` no `postgres-pvc.yml` conforme seu cluster.

### Route não funciona

Verifique se o hostname está configurado corretamente e se você tem permissões para criar Routes no projeto.

## Segurança

- **Nunca** commite secrets reais no repositório
- Use Secrets do Kubernetes/OpenShift para valores sensíveis
- Para produção, considere usar um Secret externo ou um sistema de gerenciamento de secrets
- O SESSION_SECRET deve ser único e seguro em produção

## Ambientes Disponíveis

### Base (Desenvolvimento)
- Namespace: `migration-tracker`
- Réplicas: 2
- Recursos: Padrão (128Mi/100m request, 512Mi/500m limit)

### UAT (User Acceptance Testing)
- Namespace: `migration-tracker-uat`
- Réplicas: 2
- Recursos: Padrão (128Mi/100m request, 512Mi/500m limit)
- Prefixo: `uat-`
- Hostname: `migration-tracker-uat.example.com` (ajustar em `overleys/uat/route-patch.yml`)

### Produção
- Namespace: `migration-tracker-prod`
- Réplicas: 3
- Recursos: Aumentados (256Mi/200m request, 1Gi/1000m limit)
- Prefixo: `prod-`
- Hostname: `migration-tracker.example.com` (ajustar em `overleys/prod/route-patch.yml`)

## Customização

Para adicionar novos ambientes (staging, dev, etc.), crie novos diretórios em `overleys/` seguindo o padrão de `prod/` ou `uat/`.

