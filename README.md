# Keycloak - Deployment Kubernetes

## Descrição

Deployment do Keycloak 26.4 em cluster Kubernetes com Ingress NGINX e certificados SSL via cert-manager.

## Arquitetura

- **Keycloak**: 26.4 (modo desenvolvimento)
- **Namespace**: keycloak
- **Ingress**: NGINX com SSL/TLS
- **Domínio**: keycloak.archse.eng.br
- **Certificado**: Let's Encrypt (cert-manager)

## Estrutura de Arquivos

```
k8s/
├── namespace.yaml          # Namespace keycloak
├── cluster-issuer.yaml     # ClusterIssuer do cert-manager
├── certificate.yaml        # Certificado SSL
├── secrets.yaml           # Secrets (credenciais)
├── configmap.yaml         # ConfigMaps
├── deployment.yaml        # Deployment do Keycloak
├── services.yaml          # Service ClusterIP
└── ingress.yaml           # Ingress NGINX
```

## Variáveis de Ambiente

### Deployment
- `KC_BOOTSTRAP_ADMIN_USERNAME`: Usuário admin inicial
- `KC_BOOTSTRAP_ADMIN_PASSWORD`: Senha do admin inicial
- `KC_PROXY_HEADERS`: xforwarded (para proxy reverso)
- `KC_HOSTNAME`: keycloak.archse.eng.br
- `KC_HOSTNAME_STRICT`: false
- `KC_HTTP_ENABLED`: true
- `KC_HEALTH_ENABLED`: true

## Recursos

### Requests
- Memory: 512Mi
- CPU: 500m

### Limits
- Memory: 1Gi
- CPU: 1000m

## Health Checks

### Readiness Probe
- Path: `/`
- Port: 8080
- Initial Delay: 90s
- Period: 10s
- Timeout: 10s
- Failure Threshold: 5

### Liveness Probe
- Path: `/`
- Port: 8080
- Initial Delay: 120s
- Period: 30s
- Timeout: 10s
- Failure Threshold: 5

## Deploy via Azure Pipelines

O pipeline está configurado em `azure-pipelines.yml` e executa:

1. Replace de tokens nos arquivos YAML
2. Deploy do namespace
3. Deploy dos recursos (cluster-issuer, secrets, configmap, deployment, service, certificate, ingress)

### Variáveis do Pipeline
- `imagePullSecret`: regcred
- Tokens substituídos: `__KC_BOOTSTRAP_ADMIN_USERNAME__`, `__KC_BOOTSTRAP_ADMIN_PASSWORD__`

## Acesso

- **URL**: https://keycloak.archse.eng.br
- **Console Admin**: https://keycloak.archse.eng.br/admin
- **Usuário padrão**: admin (configurado via variáveis)

## Comandos Úteis

### Ver logs
```bash
kubectl logs -n keycloak deployment/keycloak-deployment -f
```

### Ver status do pod
```bash
kubectl get pods -n keycloak
```

### Descrever pod
```bash
kubectl describe pod -n keycloak -l app=keycloak
```

### Ver eventos
```bash
kubectl get events -n keycloak --sort-by='.lastTimestamp'
```

### Ver ingress
```bash
kubectl get ingress -n keycloak
```

### Ver certificado
```bash
kubectl get certificate -n keycloak
```

### Restart do deployment
```bash
kubectl rollout restart deployment/keycloak-deployment -n keycloak
```

## Troubleshooting

### 503 Service Temporarily Unavailable
1. Verificar se o pod está rodando: `kubectl get pods -n keycloak`
2. Verificar logs do pod: `kubectl logs -n keycloak -l app=keycloak`
3. Verificar readiness probe: `kubectl describe pod -n keycloak -l app=keycloak`
4. Verificar service endpoints: `kubectl get endpoints -n keycloak`

### Pod não inicia
1. Verificar recursos disponíveis no cluster
2. Verificar logs: `kubectl logs -n keycloak -l app=keycloak`
3. Aumentar recursos se necessário (memory/cpu)

### Certificado SSL não funciona
1. Verificar cert-manager: `kubectl get pods -n cert-manager`
2. Verificar certificate: `kubectl describe certificate -n keycloak keycloak-tls`
3. Verificar cluster-issuer: `kubectl describe clusterissuer letsencrypt-keycloak-prod`

## Segurança

⚠️ **IMPORTANTE**: Em produção:
- Alterar credenciais de admin via secrets
- Usar `start` ao invés de `start-dev`
- Configurar banco de dados externo (PostgreSQL/MySQL)
- Habilitar `KC_HOSTNAME_STRICT=true`
- Configurar HTTPS only
- Implementar resource limits apropriados
- Configurar backup do banco de dados

## Referências

- [Documentação oficial Keycloak](https://www.keycloak.org/documentation)
- [Keycloak on Kubernetes](https://www.keycloak.org/operator/installation)
- [Guia de configuração](https://www.keycloak.org/server/configuration)