# Testing and Verification Guide

This document provides comprehensive instructions for testing and verifying the Chart Publishing Improvements implementation.

## Prerequisites

Before running any tests, ensure you have the following tools installed:

```bash
# Helm 3.x
helm version

# kubectl for Kubernetes interaction
kubectl version --client

# Docker or alternative container runtime for image pulls
docker version
# OR
podman version

# ORAS for OCI artifact operations (optional, for advanced testing)
oras version
```

## Local Testing

### 1. Validate Chart Structure

Verify that all chart files are in the correct locations:

```bash
# Check root-level files
ls -la Chart.yaml values.yaml .helmignore templates/NOTES.txt README.md.gotmpl values.schema.json artifacthub-repo.yml

# Expected output:
# -rw-r--r--  Chart.yaml
# -rw-r--r--  values.yaml
# -rw-r--r--  .helmignore
# -rw-r--r--  README.md.gotmpl
# -rw-r--r--  values.schema.json
# -rw-r--r--  artifacthub-repo.yml
# drwxr-xr-x  templates/
# drwxr-xr-x  crds/

# Check templates directory
ls -la templates/ | head -20
```

**Success Criteria**: All files exist at root level, 36+ template files present, no errors in file listings.

### 2. Validate Chart Configuration

```bash
# Lint the Helm chart
helm lint .

# Expected output:
# ==> Linting .
# [OK] Chart is consistent and all required fields are present.
```

**Success Criteria**: Helm lint completes without errors.

### 3. Validate Chart Syntax

```bash
# Validate Helm templates
helm template test-release . --values values.yaml

# Check for any validation errors
helm template test-release . --values values.yaml > /tmp/rendered.yaml && echo "✓ Templates render successfully" || echo "✗ Template rendering failed"
```

**Success Criteria**: All templates render without errors, no undefined variables.

### 4. Validate values.schema.json

Validate the JSON schema is well-formed:

```bash
# Check JSON syntax
python3 -m json.tool values.schema.json > /dev/null && echo "✓ Schema is valid JSON" || echo "✗ Schema has errors"

# View schema structure
jq 'keys' values.schema.json

# Verify schema covers all top-level values
jq '.properties | keys' values.schema.json
```

**Success Criteria**: JSON is valid, schema includes all properties from values.yaml.

### 5. Test values.schema.json in IDE

IDE validation is most effective with VS Code:

```bash
# Install VS Code Kubernetes extension if needed
# Open values.yaml in VS Code
code values.yaml

# Test autocomplete by:
# 1. Typing "service:" and checking for autocomplete suggestions
# 2. Typing "image:" and verifying nested properties appear
# 3. Checking validation errors appear for invalid values
```

**Success Criteria**: Autocomplete works, validation provides helpful error messages.

### 6. Verify README Template

```bash
# Check template file exists and is readable
cat README.md.gotmpl | head -20

# Verify template syntax (basic check)
grep -q "{{ template" README.md.gotmpl && echo "✓ Template syntax looks correct" || echo "⚠ Check template syntax"

# Count sections
echo "Custom sections:"
grep "^##" README.md.gotmpl | wc -l
```

**Success Criteria**: Template file exists, contains template directives, has multiple sections.

### 7. Verify NOTES.txt Template

```bash
# Check NOTES.txt exists
cat templates/NOTES.txt | head -20

# Verify template conditionals
grep -c "{{ if" templates/NOTES.txt
echo "Conditional sections found in NOTES.txt"

# Render NOTES.txt with sample values
helm template test-release . --values values.yaml --show-notes

# Check for service type conditionals
grep -E "(ClusterIP|NodePort|LoadBalancer|ExternalName)" templates/NOTES.txt
```

**Success Criteria**: NOTES.txt exists with conditionals for different service types.

### 8. Package and Inspect Chart

```bash
# Package the chart
helm package . --destination /tmp/

# List package
ls -la /tmp/common-*.tgz

# Extract and inspect
mkdir -p /tmp/chart-inspection
cd /tmp/chart-inspection
tar xzf /tmp/common-*.tgz

# Verify package contents
ls -la common/
```

**Success Criteria**: Chart packages successfully, contains all required files.

## Pre-commit Hook Testing

### 1. Verify Pre-commit Configuration

```bash
# Check configuration file exists
cat .pre-commit-config.yaml

# Check repositories are accessible
grep "repo:" .pre-commit-config.yaml
```

**Success Criteria**: Configuration file valid, all repo URLs are accessible.

### 2. Test Pre-commit Hooks Locally (Optional)

If you have pre-commit installed:

```bash
# Install pre-commit framework
pip install pre-commit

# Install hooks
pre-commit install

# Run all hooks
pre-commit run --all-files

# Expected output:
# helm-docs............................................................Passed
# Check YAML syntax......................................................Passed
# Check JSON syntax......................................................Passed
# Trim trailing whitespace...............................................Passed
```

**Success Criteria**: All hooks pass without errors.

## GitHub Actions Workflow Testing

### 1. Verify Workflow File

```bash
# Check workflow file exists and is valid YAML
cat .github/workflows/helm.yml | head -30

# Validate YAML syntax
python3 -m yaml < .github/workflows/helm.yml > /dev/null && echo "✓ Workflow YAML is valid" || echo "✗ YAML syntax error"

# Check trigger configuration
grep -A5 "on:" .github/workflows/helm.yml
```

**Success Criteria**: Workflow file exists, valid YAML, triggers on version tags (v*.*.*).

### 2. Verify Workflow Steps

```bash
# List workflow steps
grep "- name:" .github/workflows/helm.yml

# Expected steps:
# - Checkout code
# - Set up Helm
# - Set up ORAS
# - Log in to Container Registry
# - Extract version from tag
# - Prepare chart for packaging
# - Package Helm chart
# - Push chart to OCI registry
# - Push ArtifactHub metadata
```

**Success Criteria**: All required steps present and in logical order.

### 3. Dry-run Workflow Commands

Simulate workflow commands locally:

```bash
# Extract version from git tag format
TAG="v1.2.6"
VERSION=${TAG#v}
echo "Chart version: $VERSION"

# Simulate sed commands for Chart.yaml
echo "Chart.yaml update command:"
echo "sed -i \"s/^version:.*/version: $VERSION/\" Chart.yaml"

# Check package command
helm package --help | grep destination
```

**Success Criteria**: Commands are syntactically correct, sed patterns match Chart.yaml format.

## ArtifactHub Metadata Testing

### 1. Verify Metadata File

```bash
# Check artifacthub-repo.yml exists
cat artifacthub-repo.yml

# Verify required fields
grep -E "^repositoryID:|^owners:|^scannerDisabled:" artifacthub-repo.yml

# Check links
grep "url:" artifacthub-repo.yml
```

**Success Criteria**: All required fields present, links are valid URLs.

### 2. Validate YAML Syntax

```bash
# Validate YAML
python3 -m yaml < artifacthub-repo.yml > /dev/null && echo "✓ YAML is valid" || echo "✗ YAML error"

# Alternative with yq (if installed)
yq artifacthub-repo.yml > /dev/null && echo "✓ Valid YAML" || echo "✗ Invalid"
```

**Success Criteria**: YAML is valid and well-formed.

## Chart Installation Testing

### 1. Dry-run Installation

```bash
# Perform a dry-run helm install
helm install test-app . --dry-run --debug

# Check rendered manifests
helm install test-app . --dry-run --debug 2>&1 | head -50
```

**Success Criteria**: No errors, manifests are syntactically valid Kubernetes objects.

### 2. Verify NOTES.txt Renders

```bash
# Install with --show-notes to verify NOTES.txt rendering
helm install test-app . --dry-run --show-notes

# Check specific service type rendering
helm install test-app . --set service.type=ClusterIP --dry-run --show-notes
helm install test-app . --set service.type=LoadBalancer --dry-run --show-notes
helm install test-app . --set service.type=NodePort --dry-run --show-notes
```

**Success Criteria**: NOTES.txt renders correctly for each service type without errors.

### 3. Test with Custom Values

```bash
# Create test values
cat > /tmp/test-values.yaml <<EOF
image:
  repository: nginx
  tag: "latest"

deployment:
  replicaCount: 2

service:
  type: LoadBalancer

resources:
  limits:
    cpu: 500m
    memory: 512Mi
EOF

# Install with custom values
helm install test-app . -f /tmp/test-values.yaml --dry-run --debug

# Verify image is applied
helm template test-app . -f /tmp/test-values.yaml | grep "image: nginx"
```

**Success Criteria**: Custom values are applied correctly, no rendering errors.

## Docker/Container Registry Testing

### 1. Verify Chart Packaging for OCI

When running the GitHub Actions workflow:

```bash
# Expected OCI registry path:
# oci://ghcr.io/wasilak/common-chart:v1.2.6

# Manual test (requires Docker credentials):
helm push common-1.2.6.tgz oci://ghcr.io/wasilak/common-chart/

# Pull from registry:
helm pull oci://ghcr.io/wasilak/common-chart --version 1.2.6

# View manifests from registry:
helm show values oci://ghcr.io/wasilak/common-chart --version 1.2.6
```

## Troubleshooting

### Issue: Chart lint fails

**Cause**: Invalid Chart.yaml or values.yaml syntax

**Solution**:
```bash
# Check Chart.yaml syntax
cat Chart.yaml

# Validate YAML
python3 -c "import yaml; yaml.safe_load(open('Chart.yaml'))" && echo "✓ Valid"

# Check values.yaml
python3 -c "import yaml; yaml.safe_load(open('values.yaml'))" && echo "✓ Valid"
```

### Issue: Template rendering fails

**Cause**: Missing variables or invalid template syntax

**Solution**:
```bash
# Get detailed error
helm template test . 2>&1 | tail -20

# Check specific template
helm template test . --show-only templates/NOTES.txt

# Validate template syntax
grep -n "{{" templates/NOTES.txt | head -10
```

### Issue: NOTES.txt doesn't render service-specific instructions

**Cause**: Conditional logic not evaluating correctly

**Solution**:
```bash
# Test each condition
helm template test . --set service.type=ClusterIP --show-notes | grep -i "clusterip"
helm template test . --set service.type=LoadBalancer --show-notes | grep -i "loadbalancer"

# Verify conditional syntax
grep -A3 "if eq" templates/NOTES.txt
```

### Issue: values.schema.json validation not working in IDE

**Cause**: IDE not configured to use schema

**Solution**:
```bash
# In VS Code settings.json, add:
{
  "yaml.schemas": {
    "./values.schema.json": "values.yaml"
  }
}

# Or use schemaVersion directive in values.yaml:
# Requires VS Code YAML extension
```

### Issue: Pre-commit hook fails

**Cause**: helm-docs not installed or misconfigured

**Solution**:
```bash
# Install helm-docs manually
brew install helm-docs  # macOS
# or
docker run --rm -v $(pwd):/workspace ghcr.io/norwoodj/helm-docs:latest

# Generate README.md manually
helm-docs --chart-search-root=. --sort-values-order=file
```

## Verification Checklist

Use this checklist to verify all components are working:

- [ ] All chart files at root level (Chart.yaml, values.yaml, etc.)
- [ ] Helm lint passes without errors
- [ ] Chart templates render without errors
- [ ] values.schema.json is valid JSON and covers all properties
- [ ] README.md.gotmpl contains template directives and custom sections
- [ ] templates/NOTES.txt has conditional sections for service types
- [ ] artifacthub-repo.yml is valid YAML with all required fields
- [ ] .github/workflows/helm.yml is valid and triggers on v*.*.* tags
- [ ] .pre-commit-config.yaml includes helm-docs and standard checks
- [ ] .gitignore excludes generated files (README.md, artifacthub-repo.yml)
- [ ] crds/ directory exists (even if empty)
- [ ] Chart installs successfully in dry-run mode
- [ ] Custom values apply correctly
- [ ] All service types (ClusterIP, NodePort, LoadBalancer) are documented

## Next Steps

1. Tag a release: `git tag -a v1.0.0 -m "Release v1.0.0"`
2. Push the tag: `git push origin v1.0.0`
3. Watch GitHub Actions workflow execute
4. Verify chart appears in GHCR: `helm pull oci://ghcr.io/wasilak/common-chart`
5. Check ArtifactHub for chart metadata
6. Test installation from OCI registry in test cluster

## Additional Resources

- [Helm Chart Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Helm Official Documentation](https://helm.sh/docs/)
- [OCI Registry Specification](https://github.com/opencontainers/image-spec)
- [ArtifactHub Documentation](https://artifacthub.io/docs/)
- [Pre-commit Framework](https://pre-commit.com/)
