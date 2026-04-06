# Module 3: สร้าง Helm Chart

> สร้าง Chart ของตัวเอง สำหรับ Web Application

---

## สิ่งที่จะทำใน Module นี้

1. สร้าง Chart structure
2. เข้าใจโครงสร้างไฟล์
3. แก้ไข values.yaml
4. สร้าง/แก้ไข templates
5. ทดสอบ Chart
6. ติดตั้ง Chart
7. Package Chart

---

## Section 1: Helm Chart คืออะไร?

**Helm Chart** = Package ที่รวม:
- Kubernetes YAML files (Deployment, Service, etc.)
- Configuration (values)
- Metadata

### โครงสร้างพื้นฐาน:

```
mychart/
├── Chart.yaml          # ข้อมูล chart
├── values.yaml         # ค่า default
└── templates/          # Kubernetes YAML templates
    ├── deployment.yaml
    ├── service.yaml
    └── NOTES.txt       # แสดงหลังติดตั้ง
```

### Template Syntax:

```yaml
# แทนที่ด้วยค่าจาก values
replicas: {{ .Values.replicaCount }}

# ใช้ชื่อ release
name: {{ .Release.Name }}-app
```

---

## Section 2: สร้าง Chart Structure

### สร้าง Chart ด้วย helm create

```bash
helm create webapp
```

**ผลลัพธ์:** สร้าง folder `webapp/` พร้อมโครงสร้างพื้นฐาน

---

### ดูโครงสร้าง

**macOS / Linux:**
```bash
# ถ้ามี tree
tree webapp/

# ถ้าไม่มี tree
ls -R webapp/
```

**Windows (PowerShell):**
```powershell
tree /F webapp

# หรือ
Get-ChildItem -Recurse webapp
```

**Windows (CMD):**
```cmd
tree /F webapp
```

---

### ไฟล์สำคัญ:

```
webapp/
├── Chart.yaml              # ข้อมูล chart
├── values.yaml             # ค่า config
├── charts/                 # dependencies (ว่าง)
└── templates/
    ├── deployment.yaml     # Deployment template
    ├── service.yaml        # Service template
    ├── _helpers.tpl        # Helper functions
    └── NOTES.txt          # ข้อความหลังติดตั้ง
```

---

### ลบไฟล์ที่ไม่ใช้

**macOS / Linux:**
```bash
rm webapp/templates/serviceaccount.yaml
rm webapp/templates/hpa.yaml
rm webapp/templates/ingress.yaml
rm -rf webapp/templates/tests/
```

**Windows (PowerShell):**
```powershell
Remove-Item webapp/templates/serviceaccount.yaml
Remove-Item webapp/templates/hpa.yaml
Remove-Item webapp/templates/ingress.yaml
Remove-Item -Recurse webapp/templates/tests/
```

**Windows (CMD):**
```cmd
del webapp\templates\serviceaccount.yaml
del webapp\templates\hpa.yaml
del webapp\templates\ingress.yaml
rd /s /q webapp\templates\tests
```

---

## Section 3: แก้ไข Chart.yaml

### เปิดไฟล์ Chart.yaml

ใช้ text editor ที่ชอบ (VS Code, Notepad++, vim, nano, etc.)

```bash
# macOS
open -a "Visual Studio Code" webapp/Chart.yaml

# Linux
nano webapp/Chart.yaml

# Windows
notepad webapp\Chart.yaml
```

---

### แก้ไขเนื้อหา

**แก้เป็น:**

```yaml
apiVersion: v2
name: webapp
description: Simple web application chart
type: application
version: 1.0.0
appVersion: "1.0.0"
```

**บันทึกไฟล์**

---

## Section 4: แก้ไข values.yaml

### เปิดไฟล์ values.yaml

```bash
# เลือก editor ที่ชอบ
code webapp/values.yaml    # VS Code
nano webapp/values.yaml    # Nano
notepad webapp\values.yaml # Windows
```

---

### แก้ไขเนื้อหา

**แทนที่ทั้งหมดด้วย:**

```yaml
# Application settings
replicaCount: 2

image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi

# Custom config
config:
  message: "Hello from My Helm Chart!"
```

**บันทึกไฟล์**

---

## Section 5: แก้ไข Templates

### 5.1 แก้ไข deployment.yaml

เปิด `webapp/templates/deployment.yaml`

**แทนที่ทั้งหมดด้วย:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-webapp
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: webapp
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: 80
        env:
        - name: MESSAGE
          value: {{ .Values.config.message | quote }}
        resources:
          limits:
            cpu: {{ .Values.resources.limits.cpu }}
            memory: {{ .Values.resources.limits.memory }}
          requests:
            cpu: {{ .Values.resources.requests.cpu }}
            memory: {{ .Values.resources.requests.memory }}
```

**บันทึกไฟล์**

---

### 5.2 แก้ไข service.yaml

เปิด `webapp/templates/service.yaml`

**แทนที่ทั้งหมดด้วย:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-webapp
  labels:
    app: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
      protocol: TCP
  selector:
    app: {{ .Release.Name }}
```

**บันทึกไฟล์**

---

### 5.3 แก้ไข NOTES.txt

เปิด `webapp/templates/NOTES.txt`

**แทนที่ทั้งหมดด้วย:**

```
{{ .Chart.Name }} has been installed!

Release: {{ .Release.Name }}
Namespace: {{ .Release.Namespace }}
Replicas: {{ .Values.replicaCount }}

To access your application:

  kubectl port-forward svc/{{ .Release.Name }}-webapp 8080:{{ .Values.service.port }}

Then visit: http://localhost:8080

Message: {{ .Values.config.message }}
```

**บันทึกไฟล์**

---

## Section 6: ทดสอบ Chart

### 6.1 Lint Chart

ตรวจสอบ syntax:

```bash
helm lint webapp/
```

**ผลลัพธ์ต้องเป็น:**
```
==> Linting webapp/
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
```

**0 failed = ผ่าน!**

---

### 6.2 Render Templates

ดู YAML ที่จะได้:

```bash
helm template test-release webapp/
```

**ผลลัพธ์:** แสดง Deployment และ Service YAML

---

### 6.3 ดูเฉพาะ Deployment

```bash
helm template test-release webapp/ -s templates/deployment.yaml
```

---

### 6.4 Dry-run

ทดสอบกับ Kubernetes API:

```bash
helm install test webapp/ --dry-run --debug
```

**ผลลัพธ์:** แสดง YAML พร้อม values ที่ใช้

---

## Section 7: ติดตั้ง Chart

### ติดตั้ง

```bash
helm install my-app webapp/
```

**ผลลัพธ์:** แสดง NOTES ที่เราเขียน

---

### ตรวจสอบ

```bash
# ดู release
helm list

# ดู pods
kubectl get pods

# ดู service
kubectl get svc
```

**ต้องเห็น:**
- Release: my-app
- 2 pods (replicaCount=2)
- Service: my-app-webapp

---

### ทดสอบ Application

```bash
# Port forward
kubectl port-forward svc/my-app-webapp 8080:80
```

**เปิด browser:** http://localhost:8080

**ควรเห็น:** NGINX welcome page

**กด Ctrl+C** หยุด port-forward

---

### ดู Values ที่ใช้

```bash
helm get values my-app
```

**ผลลัพธ์:** ไม่มี (เพราะใช้ default values)

---

## Section 8: Customize & Upgrade

### สร้าง Custom Values

สร้างไฟล์ `custom-values.yaml`:

```yaml
replicaCount: 3

image:
  tag: "1.24"

service:
  type: LoadBalancer

config:
  message: "Hello from Custom Values!"

resources:
  limits:
    cpu: 200m
    memory: 256Mi
```

**บันทึกไฟล์**

---

### Upgrade ด้วย Custom Values

```bash
helm upgrade my-app webapp/ -f custom-values.yaml
```

---

### ตรวจสอบ

```bash
# ดู pods (ต้องมี 3)
kubectl get pods

# ดู service
kubectl get svc my-app-webapp

# ดู values ที่ใช้
helm get values my-app
```

**ต้องเห็น:**
- 3 pods
- Service type = LoadBalancer
- Custom values ที่เราตั้ง

---

## Section 9: Package Chart

### Package

```bash
helm package webapp/
```

**ผลลัพธ์:**
```
Successfully packaged chart and saved it to: webapp-1.0.0.tgz
```

ได้ไฟล์ `webapp-1.0.0.tgz`

---

### ดูเนื้อหาใน Package

**macOS / Linux:**
```bash
tar -tzf webapp-1.0.0.tgz | head -20
```

**Windows (PowerShell):**
```powershell
tar -tzf webapp-1.0.0.tgz
```

---

### ติดตั้งจาก Package

ลองติดตั้ง release ใหม่จาก package:

```bash
# ติดตั้งจาก package
helm install packaged-app webapp-1.0.0.tgz

# ตรวจสอบ
helm list
kubectl get pods
```

---

## Section 10: ทดสอบเพิ่มเติม

### ทดสอบ Template Variables

ดูว่า template ใช้งานได้ถูกต้อง:

```bash
# ดู deployment
kubectl get deployment my-app-webapp -o yaml | grep "replicas:"

# ดู env variable
kubectl get deployment my-app-webapp -o yaml | grep -A 2 "env:"
```

---

### ทดสอบ Upgrade

สร้างไฟล์ `test-values.yaml`:

```yaml
replicaCount: 1
config:
  message: "Testing upgrade!"
```

Upgrade:

```bash
helm upgrade my-app webapp/ -f test-values.yaml

# ตรวจสอบ
kubectl get pods
```

**ต้องเห็น:** pods ลดเหลือ 1

---

## Section 11: Cleanup

### ลบ Releases

```bash
helm uninstall my-app
helm uninstall packaged-app
```

---

### ลบไฟล์

**macOS / Linux:**
```bash
rm -rf webapp/
rm webapp-1.0.0.tgz custom-values.yaml test-values.yaml
```

**Windows (PowerShell):**
```powershell
Remove-Item -Recurse webapp
Remove-Item webapp-1.0.0.tgz, custom-values.yaml, test-values.yaml
```

---

### ตรวจสอบ

```bash
helm list
kubectl get pods
```

**ต้องว่างเปล่า**

---

## สรุป

### โครงสร้าง Chart:

```
webapp/
├── Chart.yaml       # Metadata
├── values.yaml      # Default values
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── _helpers.tpl
    └── NOTES.txt
```

### Template Syntax:

| Syntax | ความหมาย |
|--------|----------|
| `{{ .Release.Name }}` | ชื่อ release |
| `{{ .Chart.Name }}` | ชื่อ chart |
| `{{ .Values.key }}` | ค่าจาก values.yaml |
| `{{ .Values.key \| quote }}` | ใส่ quotes |

---

### สิ่งที่เรียนรู้:

**Module 1:**
- ติดตั้งและใช้งาน Helm
- ค้นหาและติดตั้ง applications

**Module 2:**
- Upgrade และ rollback
- Customize ด้วย values
- จัดการหลาย environments

**Module 3:**
- สร้าง Helm Chart
- เขียน templates
- Package และแชร์

---

## Next Steps

### ต่อไปทำอะไร?

1. **ฝึกฝน:**
   - สร้าง charts สำหรับ projects จริง
   - ลองแก้ไข templates ต่างๆ

2. **เรียนรู้เพิ่มเติม:**
   - Advanced templating (loops, conditions)
   - Chart dependencies
   - Helm hooks
   - Testing charts

3. **แชร์ความรู้:**
   - สอนเพื่อนร่วมงาน
   - Contribute to open source charts

---

## Resources

- **[Helm Cheat Sheet](../CHEAT-SHEET.md)** - คำสั่งที่ใช้บ่อย
- **[Helm Official Docs](https://helm.sh/docs/)** - เอกสารเต็ม
- **[Artifact Hub](https://artifacthub.io/)** - ค้นหา charts

---

## Quiz สุดท้าย

1. โครงสร้างพื้นฐานของ Helm Chart มีอะไรบ้าง?
2. ไฟล์ values.yaml ทำอะไร?
3. Template syntax สำหรับดึงค่าจาก values คืออะไร?
4. คำสั่ง package chart คืออะไร?

---

**Thank you! Happy Helming!**

---

## Final Tips

- เริ่มจาก `helm create` เสมอ
- ใช้ `helm lint` ก่อน commit
- ใช้ `--dry-run` ทดสอบ
- เก็บ charts ใน Git
- Version charts ตาม Semantic Versioning

**Good luck!**
