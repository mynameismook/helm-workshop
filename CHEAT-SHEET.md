# Helm Cheat Sheet

## Repository Management

```bash
# เพิ่ม repository
helm repo add <name> <url>
helm repo add bitnami https://charts.bitnami.com/bitnami

# ดู repositories
helm repo list

# อัพเดท repositories
helm repo update

# ลบ repository
helm repo remove <name>

# ค้นหา charts
helm search repo <keyword>
helm search hub <keyword>
```

---

## Chart Installation

```bash
# Install chart
helm install <release-name> <chart>
helm install my-app bitnami/nginx

# Install พร้อม namespace
helm install <release-name> <chart> --namespace <ns> --create-namespace

# Install พร้อม values file
helm install <release-name> <chart> -f values.yaml

# Install พร้อม set values
helm install <release-name> <chart> --set key=value

# Install พร้อม generate name
helm install <chart> --generate-name

# Dry-run
helm install <release-name> <chart> --dry-run --debug
```

---

## Release Management

```bash
# ดู releases
helm list
helm list -A                    # ทุก namespaces
helm list -n <namespace>        # เฉพาะ namespace
helm list --all-namespaces      # ทั้งหมด
helm list --uninstalled         # ที่ถูก uninstall

# ดูสถานะ release
helm status <release-name>

# ดูข้อมูล release
helm get manifest <release-name>    # manifests
helm get values <release-name>      # values ที่ override
helm get values <release-name> --all # all values
helm get notes <release-name>       # notes
helm get all <release-name>         # ทั้งหมด

# ดู history
helm history <release-name>
```

---

## Upgrade & Rollback

```bash
# Upgrade release
helm upgrade <release-name> <chart>
helm upgrade <release-name> <chart> -f values.yaml
helm upgrade <release-name> <chart> --set key=value

# Upgrade หรือ install (idempotent)
helm upgrade --install <release-name> <chart>

# Upgrade พร้อม wait
helm upgrade <release-name> <chart> --wait --timeout 5m

# Upgrade แบบ atomic (rollback ถ้าล้มเหลว)
helm upgrade <release-name> <chart> --atomic

# Rollback
helm rollback <release-name>           # rollback ไป revision ก่อนหน้า
helm rollback <release-name> <revision> # rollback ไป revision ที่ระบุ
```

---

## Uninstall

```bash
# Uninstall release
helm uninstall <release-name>

# Uninstall แต่เก็บ history
helm uninstall <release-name> --keep-history

# Uninstall จาก namespace
helm uninstall <release-name> -n <namespace>
```

---

## Chart Inspection

```bash
# ดูข้อมูล chart
helm show chart <chart>

# ดู values ที่ configure ได้
helm show values <chart>

# ดู README
helm show readme <chart>

# ดูทั้งหมด
helm show all <chart>

# Pull chart (download)
helm pull <chart>
helm pull <chart> --untar          # extract ทันที
helm pull <chart> --version <ver>  # version ที่ต้องการ
```

---

## Chart Development

```bash
# สร้าง chart ใหม่
helm create <chart-name>

# Lint chart
helm lint <chart-path>
helm lint <chart-path> --strict
helm lint <chart-path> -f values.yaml

# Render templates
helm template <release-name> <chart-path>
helm template <release-name> <chart-path> -f values.yaml
helm template <release-name> <chart-path> --debug
helm template <release-name> <chart-path> -s templates/deployment.yaml

# Package chart
helm package <chart-path>
helm package <chart-path> --version <version>
helm package <chart-path> --app-version <app-version>

# Verify package
helm verify <package.tgz>
```

---

## Dependencies

```bash
# อัพเดท dependencies
helm dependency update <chart-path>

# Build dependencies
helm dependency build <chart-path>

# ดู dependencies
helm dependency list <chart-path>
```

---

## Testing

```bash
# Run tests
helm test <release-name>

# Run tests พร้อม logs
helm test <release-name> --logs

# Cleanup test pods
kubectl delete pod -l helm.sh/hook=test
```

---

## Useful Flags

```bash
--namespace, -n         # ระบุ namespace
--create-namespace      # สร้าง namespace ถ้ายังไม่มี
--values, -f           # ระบุ values file
--set                  # set ค่าผ่าน command line
--set-string           # set ค่าเป็น string
--dry-run              # ไม่ติดตั้งจริง
--debug                # แสดงข้อมูล debug
--wait                 # รอจน resources พร้อม
--timeout              # timeout (default: 5m)
--atomic               # rollback ถ้าล้มเหลว
--force                # บังคับ upgrade
--install              # install ถ้ายังไม่มี (ใช้กับ upgrade)
--version              # ระบุ chart version
-o, --output           # output format (yaml, json, table)
```

---

## Template Functions

### String Functions
```yaml
{{ .Value | upper }}              # ตัวพิมพ์ใหญ่
{{ .Value | lower }}              # ตัวพิมพ์เล็ก
{{ .Value | quote }}              # ใส่ double quotes
{{ .Value | squote }}             # ใส่ single quotes
{{ .Value | trim }}               # trim whitespace
{{ .Value | trimSuffix "-" }}     # trim suffix
{{ .Value | trimPrefix "pre-" }}  # trim prefix
{{ .Value | replace "-" "_" }}    # replace
{{ printf "Hello %s" .Name }}     # format string
```

### Default & Required
```yaml
{{ .Value | default "default" }}                    # default value
{{ required "value required" .Value }}              # required (fail if empty)
```

### Type Conversion
```yaml
{{ .Value | toString }}           # to string
{{ .Value | toJson }}             # to JSON
{{ .Value | toYaml }}             # to YAML
{{ .Value | int }}                # to integer
{{ .Value | float64 }}            # to float
```

### Indentation
```yaml
{{ .Value | indent 2 }}           # indent
{{ .Value | nindent 2 }}          # newline + indent
```

### Lists & Dicts
```yaml
{{ list "a" "b" "c" }}                      # สร้าง list
{{ dict "key1" "val1" "key2" "val2" }}      # สร้าง dict
{{ .Values.list | join "," }}               # join list
{{ .Values.string | split "," }}            # split string
```

---

## Built-in Objects

```yaml
# Release
{{ .Release.Name }}               # release name
{{ .Release.Namespace }}          # namespace
{{ .Release.Service }}            # "Helm"
{{ .Release.IsUpgrade }}          # true ถ้าเป็น upgrade
{{ .Release.IsInstall }}          # true ถ้าเป็น install
{{ .Release.Revision }}           # revision number

# Chart
{{ .Chart.Name }}                 # chart name
{{ .Chart.Version }}              # chart version
{{ .Chart.AppVersion }}           # app version
{{ .Chart.Description }}          # description

# Values
{{ .Values.key }}                 # ค่าจาก values.yaml
{{ .Values.nested.key }}          # nested values

# Files
{{ .Files.Get "config.txt" }}     # อ่านไฟล์
{{ .Files.Glob "*.yaml" }}        # glob files

# Capabilities
{{ .Capabilities.KubeVersion }}   # Kubernetes version
{{ .Capabilities.APIVersions }}   # API versions
```

---

## Control Structures

### If/Else
```yaml
{{- if .Values.enabled }}
enabled: true
{{- else }}
enabled: false
{{- end }}

{{- if eq .Values.env "prod" }}
replicas: 3
{{- else if eq .Values.env "staging" }}
replicas: 2
{{- else }}
replicas: 1
{{- end }}
```

### With (Scope)
```yaml
{{- with .Values.service }}
type: {{ .type }}
port: {{ .port }}
{{- end }}
```

### Range (Loop)
```yaml
{{- range .Values.items }}
- {{ . }}
{{- end }}

{{- range $key, $value := .Values.labels }}
{{ $key }}: {{ $value }}
{{- end }}

{{- range $index, $item := .Values.list }}
- id: {{ $index }}
  value: {{ $item }}
{{- end }}
```

---

## Named Templates

### Define
```yaml
{{- define "mychart.labels" -}}
app: {{ .Chart.Name }}
version: {{ .Chart.Version }}
{{- end -}}
```

### Use
```yaml
labels:
  {{- include "mychart.labels" . | nindent 2 }}
```

---

## Debugging

```bash
# Debug rendering
helm template <name> <chart> --debug

# Show computed values
helm install <name> <chart> --dry-run --debug 2>&1 | grep "COMPUTED VALUES:"

# Validate with kubectl
helm template <name> <chart> | kubectl apply --dry-run=client -f -

# Check differences
helm diff upgrade <release> <chart>   # ต้องติดตั้ง helm-diff plugin
```

---

## Plugins

```bash
# ติดตั้ง plugin
helm plugin install <url>

# ดู plugins
helm plugin list

# ลบ plugin
helm plugin uninstall <plugin>

# Useful plugins:
helm plugin install https://github.com/databus23/helm-diff
helm plugin install https://github.com/helm-unittest/helm-unittest
```

---

## Quick Tips

1. **Always use `--dry-run` ก่อน install/upgrade**
   ```bash
   helm install myapp ./chart --dry-run --debug
   ```

2. **ใช้ `--atomic` เพื่อ auto-rollback**
   ```bash
   helm upgrade myapp ./chart --atomic
   ```

3. **ใช้ `--wait` รอให้ resources พร้อม**
   ```bash
   helm install myapp ./chart --wait --timeout 5m
   ```

4. **Export values เป็นไฟล์**
   ```bash
   helm show values bitnami/nginx > nginx-values.yaml
   ```

5. **ใช้ `upgrade --install` (idempotent)**
   ```bash
   helm upgrade --install myapp ./chart
   ```

6. **Backup values ก่อน upgrade**
   ```bash
   helm get values myapp > myapp-values-backup.yaml
   ```

---

## Resources

- Official Docs: https://helm.sh/docs/
- Artifact Hub: https://artifacthub.io/
- Helm GitHub: https://github.com/helm/helm
- Best Practices: https://helm.sh/docs/chart_best_practices/
