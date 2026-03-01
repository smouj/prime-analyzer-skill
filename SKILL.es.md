---
name: Prime Analyzer
version: 1.2.0
description: Analizador de DevOps impulsado por IA para infraestructura, CI/CD, seguridad y rendimiento
author: SMOUJBOT
tags: [devops, ai, automation]
dependencies:
  - kubectl >= 1.28
  - terraform >= 1.5
  - helm >= 3.12
  - docker >= 24.0
  - jq >= 1.6
  - yq >= 4.30
  - promtool >= 2.45
  - trivy >= 0.48
  - opa >= 0.57
required_env:
  - PRIME_OPENAI_API_KEY (for AI analysis engine)
  - K8S_CONTEXT (optional, defaults to current context)
  - PRIME_LOG_LEVEL (debug, info, warn, error - default: info)
workdir: /tmp/prime-analyzer
cache_dir: ~/.prime/cache
timeout: 300s
---

# Prime Analyzer

Analizador impulsado por IA para tareas de DevOps que incluyen validación de infraestructura, optimización de CI/CD, escaneo de seguridad y análisis de rendimiento.

## Propósito

Prime Analyzer automatiza tareas críticas de validación de DevOps:

- **Análisis de Infraestructura como Código**: Validar planes de Terraform, detectar drift, identificar configuraciones de seguridad incorrectas
- **Evaluación de salud de Kubernetes**: Analizar estado del clúster, detectar problemas de recursos, validar políticas de red
- **Optimización de pipelines de CI/CD**: Identificar etapas ineficientes, oportunidades de paralelización, fugas de secretos
- **Escaneo de postura de seguridad**: Análisis de vulnerabilidades de contenedores, cumplimiento de políticas OPA, escaneo de seguridad de IaC
- **Detección de cuellos de botella de rendimiento**: Analizar métricas, logs y traces para identificar problemas de rendimiento
- **Optimización de costos**: Detectar recursos sobreaprovisionados, volúmenes no utilizados, tipos de instancia ineficientes

## Alcance

### Comandos

#### `prime-analyzer analyze terraform`
Analiza configuraciones y planes de Terraform.
- `--plan FILE` - Ruta al archivo JSON del plan de terraform (terraform show -json)
- `--config DIR` - Directorio de configuración de Terraform (por defecto: .)
- `--check drift` - Detectar drift de infraestructura
- `--check security` - Ejecutar comprobaciones de seguridad (reglas tfsec/snyk)
- `--check cost` - Estimar costos e identificar optimizaciones
- `--output FORMAT` - json, yaml, sarif, markdown (por defecto: json)

#### `prime-analyzer analyze k8s`
Analiza la salud y configuración del clúster de Kubernetes.
- `--context STRING` - Contexto de Kubernetes (por defecto: $K8S_CONTEXT o actual)
- `--namespace STRING` - Namespace objetivo (por defecto: todos)
- `--check resources` - Analizar uso de recursos y límites
- `--check network` - Validar políticas de red y exposiciones
- `--check security` - Escanear configuraciones de seguridad incorrectas
- `--check hpa` - Validar configuraciones de HorizontalPodAutoscaler

#### `prime-analyzer analyze cicd`
Analiza configuraciones de pipelines de CI/CD.
- `--type platform` - gitlab, github, jenkins, argo, tekton (requerido)
- `--config FILE` - Ruta del archivo de configuración del pipeline
- `--repo DIR` - Ruta del repositorio para escanear archivos de pipeline
- `--check secrets` - Detectar secretos y tokens hardcodeados
- `--check efficiency` - Identificar etapas lentas, brechas en paralelización
- `--check compliance` - Validar contra marcos de cumplimiento

#### `prime-analyzer analyze security`
Análisis de seguridad integral en múltiples capas.
- `--target TYPE` - container, iac, k8s, cloud (aws/gcp/azure)
- `--image STRING` - Imagen de contenedor a escanear (para objetivo container)
- `--scan-all` - Ejecutar todas las comprobaciones de seguridad (completo)
- `--severity THRESHOLD` - MINIMAL, LOW, MEDIUM, HIGH, CRITICAL (por defecto: MEDIUM)
- `--output-format sarif` - Formato SARIF para integración con GitHub

#### `prime-analyzer analyze performance`
Analiza rendimiento usando métricas, logs y traces.
- `--metrics URL` - URL de Prometheus remote read o archivo de métricas local
- `--logs FILE` - Ruta del archivo de logs (logs JSON estructurados)
- `--traces FILE` - Archivo de trace Jaeger/Zipkin
- `--slo-file FILE` - Archivo de definición SLO (YAML)
- `--window DURATION` - Ventana de tiempo de análisis (ej: 1h, 24h, 7d)
- `--report` - Generar reporte de rendimiento con recomendaciones

#### `prime-analyzer analyze cost`
Análisis y optimización de costos en la nube.
- `--platform aws|gcp|azure` - Proveedor de nube (requerido)
- `--period RANGE` - Período del reporte de costos (ej: 30d, 90d)
- `--resource-group STRING` - Filtrar por grupo de recursos/etiqueta
- `--check idle` - Identificar recursos inactivos
- `--check overprovision` - Detectar instancias sobreaprovisionadas
- `--recommendations` - Generar recomendaciones de optimización con ahorros

#### `prime-analyzer plugins list`
Listar plugins de análisis disponibles.

#### `prime-analyzer plugins enable PLUGIN`
Habilitar un plugin.

#### `prime-analyzer plugins disable PLUGIN`
Deshabilitar un plugin.

#### `prime-analyzer version`
Mostrar versión y estado de dependencias.

## Proceso de Trabajo

### Flujo de Análisis Estándar

1. **Inicialización**: Verificar dependencias, cargar modelo de IA, verificar API keys
2. **Recopilación de Entrada**: Leer archivos de configuración, recopilar datos en vivo de APIs
3. **Análisis Estático**: Parsear configuraciones, aplicar comprobaciones basadas en reglas
4. **Mejora con IA**: Enviar contexto a OpenAI API para reconocimiento de patrones y detección de anomalías
5. **Referencia Cruzada**: Correlacionar hallazgos entre diferentes tipos de análisis
6. **Detección de Brechas**: Identificar configuraciones faltantes, dependencias no documentadas
7. **Puntuación de Riesgo**: Calcular puntuaciones de riesgo basadas en severidad, exposición y radio de explosión
8. **Generación de Recomendaciones**: Priorizar correcciones con contexto de impacto empresarial
9. **Generación de Reporte**: Salida en formato solicitado con pasos accionables

### Pasos Detallados

```
# 1. Comprobaciones pre-vuelo
prime-analyzer check-env              # Verificar todas las dependencias
prime-analyzer validate-config FILE   # Validar esquema de configuración

# 2. Ejecutar análisis con alcance específico
prime-analyzer analyze terraform --plan terraform/plan.json --check=drift,security

# 3. Ver resultados con filtrado
jq '.[] | select(.severity == "HIGH" or .severity == "CRITICAL")' report.json

# 4. Aplicar correcciones con flujo de trabajo guiado
prime-analyzer fix apply --plan      # Generar plan de corrección de terraform
prime-analyzer fix apply --k8s       # Generar manifiestos de corrección de k8s
prime-analyzer fix apply --approve   # Aplicar correcciones (después de revisión)
```

## Reglas de Oro

1. **Nunca ejecutar en clústeres de producción sin --dry-run primero** - siempre previsualizar cambios
2. **Las sugerencias de IA requieren revisión humana** - la IA puede alucinar, validar todas las recomendaciones
3. **Cachear resultados para análisis repetibles** - usar flag `--cache` para evitar límites de tasa
4. **Usar umbrales de severidad** - filtrar a HIGH/CRITICAL en CI/CD para evitar ruido
5. **Ejecutar por etapas** - comenzar con modo solo checklist, luego habilitar análisis con IA
6. **Almacenar reportes como artefactos** - mantener JSON/SARIF para auditoría
7. **Actualizar plugins regularmente** - `prime-analyzer plugins update --all`
8. **Nunca exponer PRIME_OPENAI_API_KEY en logs** - usar solo variables de entorno
9. **Limitar alcance de análisis** - usar `--namespace` y `--resource-group` para evitar creep de alcance
10. **Siempre verificar correcciones** - re-ejecutar análisis con mismos flags después de aplicar cambios

## Ejemplos

### Ejemplo 1: Detección de drift de Terraform
```bash
# Generar plan de terraform
terraform plan -out=tfplan
terraform show -json tfplan > plan.json

# Analizar para drift
prime-analyzer analyze terraform \
  --plan plan.json \
  --config ./terraform \
  --check drift \
  --output markdown > drift-report.md

# Salida esperada: Recursos con atributos faltantes/extra, discrepancias de count
```

### Ejemplo 2: Verificación de salud de clúster Kubernetes
```bash
# Escanear todos los namespaces para problemas de recursos
prime-analyzer analyze k8s \
  --context production-cluster \
  --check resources,network \
  --output json > k8s-health.json

# Esperado: Discrepancias de CPU/Memory requests/limits, servicios expuestos, políticas de red faltantes
```

### Ejemplo 3: Escaneo de fuga de secretos en CI/CD
```bash
# Escanear workflows de GitHub Actions
prime-analyzer analyze cicd \
  --type github \
  --repo ./myapp \
  --check secrets \
  --severity HIGH \
  --output sarif > secrets.sarif

# Esperado: Detección de AWS_ACCESS_KEY, GITHUB_TOKEN, contraseñas de bases de datos en logs
```

### Ejemplo 4: Análisis de SLO de rendimiento
```bash
# Analizar métricas de Prometheus para latencia de API
prime-analyzer analyze performance \
  --metrics "http://prometheus:9090/api/v1/query_range" \
  --slo-file ./slo.yaml \
  --window 7d \
  --report > performance-report.html

# Ejemplo de archivo SLO:
# slos:
#   - name: api_latency
#     expr: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
#     target: 0.2  # 200ms
#     window: 7d
```

### Ejemplo 5: Optimización de costos AWS
```bash
prime-analyzer analyze cost \
  --platform aws \
  --period 30d \
  --check idle,overprovision \
  --resource-group tag:Env=production \
  --recommendations > cost-savings.yaml

# Esperado: Instancias EC2 detenidas, RDS subutilizadas, tipos de instancia desactualizados
```

## Variables de Entorno

- `PRIME_OPENAI_API_KEY` (requerido) - API key de OpenAI para motor de análisis con IA
- `K8S_CONTEXT` (opcional) - Contexto de Kubernetes por defecto (sobrescribe contexto actual)
- `PRIME_LOG_LEVEL` (opcional) - Nivel de logging: debug, info, warn, error (por defecto: info)
- `PRIME_MAX_TOKENS` (opcional) - Tokens máximos para análisis con IA (por defecto: 4000)
- `PRIME_CACHE_TTL` (opcional) - Tiempo de vida de caché en segundos (por defecto: 3600)
- `PRIME_PROXY_URL` (opcional) - Proxy HTTP para llamadas a API
- `PRIME_TIMEOUT` (opcional) - Anulación de timeout global (ej: 600s)

## Dependencias

Verificar instalación:

```bash
# Verificar todas las dependencias
prime-analyzer doctor

# Instalar dependencias faltantes (Ubuntu/Debian)
sudo apt-get install -y jq yq kubectl terraform helm docker.io trivy

# Instalar OPA
curl -L -o opa https://openpolicyagent.org/downloads/v0.57.0/opa_linux_amd64_static
chmod +x opa && sudo mv opa /usr/local/bin/

# Instalar promtool (desde release de prometheus)
# Descargar desde: https://github.com/prometheus/prometheus/releases
```

## Pasos de Verificación

Después del análisis, verificar resultados:

```bash
# 1. Verificar código de salida
echo $?
# 0 = éxito con hallazgos o sin hallazgos
# 1 = error de análisis
# 2 = dependencia faltante
# 3 = argumentos inválidos

# 2. Validar salida JSON (si se usa --output json)
jq empty report.json && echo "JSON Válido"

# 3. Contar hallazgos por severidad
jq '[.[] | select(.severity | in(["CRITICAL","HIGH"]))] | length' report.json

# 4. Verificar alucinaciones (específico de IA)
jq '[.[] | select(.recommendation | test("^[A-Z]"))] | length' report.json
# Debería ser 0 o muy bajo - las recomendaciones deberían empezar con minúscula

# 5. Referencia cruzada con verificación manual
# Elegir 1-2 hallazgos y validar manualmente
```

## Comandos de Rollback

### Para cambios de Terraform:
```bash
# Listar planes de corrección generados (almacenados en caché)
prime-analyzer cache list | grep terraform

# Aplicar rollback (si se aplicó la corrección)
terraform apply -var="rollback=true" rollback.tfplan

# O revertir commit de git si los cambios fueron commiteados
git revert <commit-hash>
```

### Para correcciones de Kubernetes:
```bash
# Ver correcciones generadas antes de aplicar
prime-analyzer cache get k8s-fix-<timestamp>.yaml

# Eliminar recursos aplicados
kubectl delete -f ./prime-fixes/manifests.yaml --ignore-not-found

# Restaurar desde backup (si se usó flag --backup)
kubectl apply -f ./prime-backup/pre-fix-backup.yaml
```

### Para cambios de políticas de seguridad:
```bash
# Las políticas OPA están versionadas en ~/.prime/policies/
# Rollback a versión anterior
prime-analyzer policy rollback --policy network-policy.rego --to v1.2

# O deshabilitar política completamente
prime-analyzer policy disable --policy strict-pod-security
```

### Limpieza completa (opción nuclear):
```bash
# Eliminar todos los archivos de análisis temporales
prime-analyzer cache clean --all

# Resetear configuración a valores por defecto
prime-analyzer config reset --confirm

# Unset variables de entorno
unset PRIME_OPENAI_API_KEY
```

## Solución de Problemas

### Problema: "OpenAI API quota exceeded"
**Solución**: Set `PRIME_MAX_TOKENS=2000` para reducir uso de tokens, o agregar caché: `--cache-ttl 7200`

### Problema: "kubectl: command not found"
**Solución**: Instalar kubectl o set `K8S_USE_API=true` para usar llamadas directas a API (requiere kubeconfig)

### Problema: Análisis lento (> timeout)
**Solución**: Aumentar timeout: `PRIME_TIMEOUT=600s prime-analyzer ...` o reducir alcance con `--namespace`

### Problema: "Invalid terraform plan format"
**Solución**: Asegurar que el plan sea JSON: `terraform show -json <plan-file> > plan.json`

### Problema: Alucinaciones de IA en resultados
**Solución**: Reducir contribución de IA: `--ai-weight=0.3` (rango 0-1, por defecto 0.7) o usar `--no-ai` para solo reglas

### Problema: Permission denied en directorio de caché
**Solución**: Arreglar permisos: `chmod 700 ~/.prime/cache && chown $USER ~/.prime/cache`

### Problema: Alto uso de memoria
**Solución**: Limitar análisis paralelo: `--max-parallel=2` (por defecto: cuenta de CPU)

### Problema: Plugin faltante
**Solución**: Listar disponibles: `prime-analyzer plugins list`. Habilitar: `prime-analyzer plugins enable <name>`

### Problema: Falsos positivos en hallazgos de seguridad
**Solución**: Crear reglas de excepción en `~/.prime/config/exceptions.yaml`:
```yaml
rules:
  - id: K8S_001
    exempt:
      - namespace: monitoring
        reason: "Monitoring necesita hostPath"
```

## Configuración

Crear `~/.prime/config.yaml`:

```yaml
ai:
  model: gpt-4-turbo-preview
  temperature: 0.1
  max_tokens: 4000
  cache_ttl: 3600

plugins:
  enabled:
    - terraform
    - kubernetes
    - cicd
    - security
    - cost
  disabled:
    - experimental

checks:
  severity_threshold: MEDIUM
  ignore_patterns:
    - "*.test.yaml"
    - "examples/*"

reporting:
  format: json
  include_ai_reasoning: true
  include_raw_findings: false

cloud:
  aws:
    region: us-east-1
    profile: default
  gcp:
    project: my-project
    credentials_file: ~/.gcp/key.json
```

## Códigos de Salida

- `0` - Éxito (con o sin hallazgos)
- `1` - Error de análisis (entrada inválida, fallo de API)
- `2` - Dependencia faltante
- `3` - Argumentos inválidos
- `4` - Permiso denegado
- `5` - Timeout
- `6` - Servicio de IA no disponible
- `7` - Recursos insuficientes (memoria/disco)

## Soporte

- Issues: https://github.com/smouj/prime-analyzer/issues
- Docs: https://prime-analyzer.smouj.dev/docs
- Logs: `~/.prime/logs/prime-analyzer-$(date +%Y-%m-%d).log`
```
```