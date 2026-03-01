---
name: Prime Analyzer
version: 1.2.0
description: AI-powered DevOps analyzer for infrastructure, CI/CD, security, and performance
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
  -opa >= 0.57
required_env:
  - PRIME_OPENAI_API_KEY (for AI analysis engine)
  - K8S_CONTEXT (optional, defaults to current context)
  - PRIME_LOG_LEVEL (debug, info, warn, error - default: info)
workdir: /tmp/prime-analyzer
cache_dir: ~/.prime/cache
timeout: 300s
---

# Prime Analyzer

AI-powered analyzer for DevOps tasks including infrastructure validation, CI/CD optimization, security scanning, and performance analysis.

## Purpose

Prime Analyzer automates critical DevOps validation tasks:

- **Infrastructure-as-Code analysis**: Validate Terraform plans, detect drift, identify security misconfigurations
- **Kubernetes health assessment**: Analyze cluster state, detect resource issues, validate network policies
- **CI/CD pipeline optimization**: Identify inefficient stages, parallelization opportunities, secret leaks
- **Security posture scanning**: Container vulnerability analysis, OPA policy compliance, IaC security scanning
- **Performance bottleneck detection**: Analyze metrics, logs, and traces to identify performance issues
- **Cost optimization**: Detect overprovisioned resources, unused volumes, inefficient instance types

## Scope

### Commands

#### `prime-analyzer analyze terraform`
Analyze Terraform configurations and plans.
- `--plan FILE` - Path to terraform plan JSON (terraform show -json)
- `--config DIR` - Terraform configuration directory (default: .)
- `--check drift` - Detect infrastructure drift
- `--check security` - Run security checks (tfsec/snyk rules)
- `--check cost` - Estimate costs and identify optimizations
- `--output FORMAT` - json, yaml, sarif, markdown (default: json)

#### `prime-analyzer analyze k8s`
Analyze Kubernetes cluster health and configuration.
- `--context STRING` - Kubernetes context (default: $K8S_CONTEXT or current)
- `--namespace STRING` - Target namespace (default: all)
- `--check resources` - Analyze resource usage and limits
- `--check network` - Validate network policies and exposures
- `--check security` - Scan for security misconfigurations
- `--check hpa` - Validate HorizontalPodAutoscaler configurations

#### `prime-analyzer analyze cicd`
Analyze CI/CD pipeline configurations.
- `--type platform` - gitlab, github, jenkins, argo, tekton (required)
- `--config FILE` - Pipeline configuration file path
- `--repo DIR` - Repository path to scan for pipeline files
- `--check secrets` - Detect hardcoded secrets and tokens
- `--check efficiency` - Identify slow stages, gaps in parallelization
- `--check compliance` - Validate against compliance frameworks

#### `prime-analyzer analyze security`
Comprehensive security analysis across multiple layers.
- `--target TYPE` - container, iac, k8s, cloud (aws/gcp/azure)
- `--image STRING` - Container image to scan (for container target)
- `--scan-all` - Run all security checks (comprehensive)
- `--severity THRESHOLD` - MINIMAL, LOW, MEDIUM, HIGH, CRITICAL (default: MEDIUM)
- `--output-format sarif` - SARIF format for GitHub integration

#### `prime-analyzer analyze performance`
Analyze performance using metrics, logs, and traces.
- `--metrics URL` - Prometheus remote read URL or local metrics file
- `--logs FILE` - Log file path (structured JSON logs)
- `--traces FILE` - Jaeger/Zipkin trace file
- `--slo-file FILE` - SLO definition file (YAML)
- `--window DURATION` - Analysis time window (e.g., 1h, 24h, 7d)
- `--report` - Generate performance report with recommendations

#### `prime-analyzer analyze cost`
Cloud cost analysis and optimization.
- `--platform aws|gcp|azure` - Cloud provider (required)
- `--period RANGE` - Cost report period (e.g., 30d, 90d)
- `--resource-group STRING` - Filter by resource group/tag
- `--check idle` - Identify idle resources
- `--check overprovision` - Detect overprovisioned instances
- `--recommendations` - Generate optimization recommendations with savings

#### `prime-analyzer plugins list`
List available analysis plugins.

#### `prime-analyzer plugins enable PLUGIN`
Enable a plugin.

#### `prime-analyzer plugins disable PLUGIN`
Disable a plugin.

#### `prime-analyzer version`
Show version and dependency status.

## Work Process

### Standard Analysis Flow

1. **Initialization**: Check dependencies, load AI model, verify API keys
2. **Input Collection**: Read configuration files, gather live data from APIs
3. **Static Analysis**: Parse configurations, apply rule-based checks
4. **AI Enhancement**: Send context to OpenAI API for pattern recognition and anomaly detection
5. **Cross-Reference**: Correlate findings across different analysis types
6. **Gap Detection**: Identify missing configurations, undocumented dependencies
7. **Risk Scoring**: Calculate risk scores based on severity, exposure, and blast radius
8. **Recommendation Generation**: Prioritize fixes with business impact context
9. **Report Generation**: Output in requested format with actionable steps

### Detailed Steps

```
# 1. Pre-flight checks
prime-analyzer check-env              # Verify all dependencies
prime-analyzer validate-config FILE   # Validate configuration schema

# 2. Run analysis with specific scope
prime-analyzer analyze terraform --plan terraform/plan.json --check=drift,security

# 3. View results with filtering
jq '.[] | select(.severity == "HIGH" or .severity == "CRITICAL")' report.json

# 4. Apply fixes with guided workflow
prime-analyzer fix apply --plan      # Generate terraform fix plan
prime-analyzer fix apply --k8s       # Generate k8s fix manifests
prime-analyzer fix apply --approve   # Apply fixes (after review)
```

## Golden Rules

1. **Never run in production clusters without --dry-run first** - always preview changes
2. **AI suggestions require human review** - AI can hallucinate, validate all recommendations
3. **Cache results for repeatable analysis** - use `--cache` flag to avoid rate limits
4. **Use severity thresholds** - filter to HIGH/CRITICAL in CI/CD to avoid noise
5. **Run in stages** - start with checklist-only mode, then enable AI analysis
6. **Store reports as artifacts** - keep JSON/SARIF for audit trail
7. **Regularly update plugins** - `prime-analyzer plugins update --all`
8. **Never expose PRIME_OPENAI_API_KEY in logs** - use environment variables only
9. **Limit analysis scope** - use `--namespace` and `--resource-group` to avoid scope creep
10. **Always verify fixes** - re-run analysis with same flags after applying changes

## Examples

### Example 1: Terraform drift detection
```bash
# Generate terraform plan
terraform plan -out=tfplan
terraform show -json tfplan > plan.json

# Analyze for drift
prime-analyzer analyze terraform \
  --plan plan.json \
  --config ./terraform \
  --check drift \
  --output markdown > drift-report.md

# Expected output: Resources with missing/extra attributes, count mismatches
```

### Example 2: Kubernetes cluster health check
```bash
# Scan all namespaces for resource issues
prime-analyzer analyze k8s \
  --context production-cluster \
  --check resources,network \
  --output json > k8s-health.json

# Expected: CPU/Memory requests/limits mismatch, exposed services, missing network policies
```

### Example 3: CI/CD secret leakage scan
```bash
# Scan GitHub Actions workflows
prime-analyzer analyze cicd \
  --type github \
  --repo ./myapp \
  --check secrets \
  --severity HIGH \
  --output sarif > secrets.sarif

# Expected: Detection of AWS_ACCESS_KEY, GITHUB_TOKEN, Database passwords in logs
```

### Example 4: Performance SLO analysis
```bash
# Analyze Prometheus metrics for API latency
prime-analyzer analyze performance \
  --metrics "http://prometheus:9090/api/v1/query_range" \
  --slo-file ./slo.yaml \
  --window 7d \
  --report > performance-report.html

# SLO file example:
# slos:
#   - name: api_latency
#     expr: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
#     target: 0.2  # 200ms
#     window: 7d
```

### Example 5: AWS cost optimization
```bash
prime-analyzer analyze cost \
  --platform aws \
  --period 30d \
  --check idle,overprovision \
  --resource-group tag:Env=production \
  --recommendations > cost-savings.yaml

# Expected: Stopped EC2 instances, underutilized RDS, outdated instance types
```

## Environment Variables

- `PRIME_OPENAI_API_KEY` (required) - OpenAI API key for AI analysis engine
- `K8S_CONTEXT` (optional) - Default Kubernetes context (overrides current context)
- `PRIME_LOG_LEVEL` (optional) - Logging level: debug, info, warn, error (default: info)
- `PRIME_MAX_TOKENS` (optional) - Max tokens for AI analysis (default: 4000)
- `PRIME_CACHE_TTL` (optional) - Cache time-to-live in seconds (default: 3600)
- `PRIME_PROXY_URL` (optional) - HTTP proxy for API calls
- `PRIME_TIMEOUT` (optional) - Global timeout override (e.g., 600s)

## Dependencies

Verify installation:

```bash
# Check all dependencies
prime-analyzer doctor

# Install missing dependencies (Ubuntu/Debian)
sudo apt-get install -y jq yq kubectl terraform helm docker.io trivy

# Install OPA
curl -L -o opa https://openpolicyagent.org/downloads/v0.57.0/opa_linux_amd64_static
chmod +x opa && sudo mv opa /usr/local/bin/

# Install promtool (from prometheus release)
# Download from: https://github.com/prometheus/prometheus/releases
```

## Verification Steps

After analysis, verify results:

```bash
# 1. Check exit code
echo $?
# 0 = success with findings or no findings
# 1 = analysis error
# 2 = dependency missing
# 3 = invalid arguments

# 2. Validate JSON output (if using --output json)
jq empty report.json && echo "Valid JSON"

# 3. Count findings by severity
jq '[.[] | select(.severity | in(["CRITICAL","HIGH"]))] | length' report.json

# 4. Check for hallucinations (AI-specific)
jq '[.[] | select(.recommendation | test("^[A-Z]"))] | length' report.json
# Should be 0 or very low - recommendations should start lowercase

# 5. Cross-reference with manual check
# Pick 1-2 findings and validate manually
```

## Rollback Commands

### For Terraform changes:
```bash
# List generated fix plans (stored in cache)
prime-analyzer cache list | grep terraform

# Apply rollback (if fix was applied)
terraform apply -var="rollback=true" rollback.tfplan

# Or revert git commit if changes were committed
git revert <commit-hash>
```

### For Kubernetes fixes:
```bash
# View generated fixes before applying
prime-analyzer cache get k8s-fix-<timestamp>.yaml

# Delete applied resources
kubectl delete -f ./prime-fixes/manifests.yaml --ignore-not-found

# Restore from backup (if --backup flag was used)
kubectl apply -f ./prime-backup/pre-fix-backup.yaml
```

### For security policy changes:
```bash
# OPA policies are versioned in ~/.prime/policies/
# Rollback to previous version
prime-analyzer policy rollback --policy network-policy.rego --to v1.2

# Or disable policy entirely
prime-analyzer policy disable --policy strict-pod-security
```

### Full cleanup (nuclear option):
```bash
# Remove all temporary analysis files
prime-analyzer cache clean --all

# Reset configuration to default
prime-analyzer config reset --confirm

# Unset environment variables
unset PRIME_OPENAI_API_KEY
```

## Troubleshooting

### Issue: "OpenAI API quota exceeded"
**Solution**: Set `PRIME_MAX_TOKENS=2000` to reduce token usage, or add caching: `--cache-ttl 7200`

### Issue: "kubectl: command not found"
**Solution**: Install kubectl or set `K8S_USE_API=true` to use direct API calls (requires kubeconfig)

### Issue: Slow analysis (> timeout)
**Solution**: Increase timeout: `PRIME_TIMEOUT=600s prime-analyzer ...` or narrow scope with `--namespace`

### Issue: "Invalid terraform plan format"
**Solution**: Ensure plan is JSON: `terraform show -json <plan-file> > plan.json`

### Issue: AI hallucinations in results
**Solution**: Lower AI contribution: `--ai-weight=0.3` (range 0-1, default 0.7) or use `--no-ai` for rule-only

### Issue: Permission denied on cache directory
**Solution**: Fix permissions: `chmod 700 ~/.prime/cache && chown $USER ~/.prime/cache`

### Issue: High memory usage
**Solution**: Limit parallel analysis: `--max-parallel=2` (default: CPU count)

### Issue: Missing plugin
**Solution**: List available: `prime-analyzer plugins list`. Enable: `prime-analyzer plugins enable <name>`

### Issue: False positive security findings
**Solution**: Create exception rules in `~/.prime/config/exceptions.yaml`:
```yaml
rules:
  - id: K8S_001
    exempt:
      - namespace: monitoring
        reason: "Monitoring needs hostPath"
```

## Configuration

Create `~/.prime/config.yaml`:

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

## Exit Codes

- `0` - Success (with or without findings)
- `1` - Analysis error (invalid input, API failure)
- `2` - Missing dependency
- `3` - Invalid arguments
- `4` - Permission denied
- `5` - Timeout
- `6` - AI service unavailable
- `7` - Insufficient resources (memory/disk)

## Support

- Issues: https://github.com/smouj/prime-analyzer/issues
- Docs: https://prime-analyzer.smouj.dev/docs
- Logs: `~/.prime/logs/prime-analyzer-$(date +%Y-%m-%d).log`
```