# Module 2: จัดการ Applications

> Upgrade, Rollback, Customize ด้วย Values

---

## สิ่งที่จะทำใน Module นี้

1. Install application
2. Upgrade application
3. Rollback เมื่อมีปัญหา
4. Customize application ด้วย values
5. สร้าง values files สำหรับ dev/prod
6. เข้าใจ values priority

---

## Section 1: Install → Upgrade → Rollback

### ติดตั้ง NGINX

```bash
helm install my-web bitnami/nginx
```

รอจน pod พร้อม:

```bash
kubectl get pods -w
# กด Ctrl+C เมื่อ STATUS = Running
```

---

### ดู Release ปัจจุบัน

```bash
# ดู releases
helm list

# ดูสถานะ
helm status my-web
```

**จำ REVISION ไว้** → ตอนนี้ควรเป็น **1** (install ครั้งแรก)

---

### Upgrade: เพิ่มจำนวน Replicas

ตอนนี้มี pod เดียว เราจะเพิ่มเป็น 3 pods:

```bash
helm upgrade my-web bitnami/nginx --set replicaCount=3
```

**ผลลัพธ์:**
```
Release "my-web" has been upgraded. Happy Helming!
```

---

### ตรวจสอบ Upgrade

```bash
# ดู pods (ควรมี 3 ตัว)
kubectl get pods

# ดู history
helm history my-web
```

**ผลลัพธ์ history:**
```
REVISION  STATUS      CHART        DESCRIPTION
1         superseded  nginx-...    Install complete
2         deployed    nginx-...    Upgrade complete
```

**REVISION เพิ่มเป็น 2**

---

### Upgrade อีกครั้ง: เปลี่ยน Service Type

```bash
helm upgrade my-web bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort
```

**สำคัญ:** ต้องใส่ `replicaCount=3` ด้วย ไม่งั้นจะกลับเป็น 1!

---

### ตรวจสอบ Service

```bash
kubectl get svc
```

**ต้องเห็น:** TYPE = `NodePort`

---

### ดู History อีกครั้ง

```bash
helm history my-web
```

**ควรมี 3 revisions:**
- Revision 1: Install
- Revision 2: Upgrade (3 replicas)
- Revision 3: Upgrade (3 replicas + NodePort)

---

### Checkpoint: Upgrade สำเร็จ

```bash
# เช็ค pods
kubectl get pods | grep my-web

# เช็ค service
kubectl get svc my-web-nginx
```

**ต้องเห็น:**
- 3 pods
- Service type = NodePort

---

### Rollback ไปยัง Revision ก่อนหน้า

สมมติว่า upgrade ครั้งล่าสุดมีปัญหา เราจะ rollback:

```bash
helm rollback my-web 2
```

**อธิบาย:** rollback ไป revision 2

---

### ตรวจสอบ Rollback

```bash
# ดู service (ควรกลับเป็น LoadBalancer)
kubectl get svc my-web-nginx

# ดู history
helm history my-web
```

**ผลลัพธ์:**
```
REVISION  STATUS        CHART
1         superseded    ...
2         superseded    ...
3         superseded    ...
4         deployed      ... (Rollback to 2)
```

**REVISION 4 = rollback ไป revision 2**

---

### Checkpoint: Rollback สำเร็จ

```bash
kubectl get svc my-web-nginx
```

**TYPE ต้องกลับเป็น:** `LoadBalancer`

---

## Section 2: Customize ด้วย Values

### ดู Values ที่มี

```bash
# ดู values ทั้งหมด
helm show values bitnami/nginx | head -100
```

**จะเห็น:**
- `replicaCount`
- `image.repository`, `image.tag`
- `service.type`, `service.port`
- `resources.limits`, `resources.requests`
- และอื่นๆ อีกมากมาย

---

### Save Values เป็นไฟล์

```bash
# macOS / Linux
helm show values bitnami/nginx > nginx-values.yaml

# Windows (PowerShell)
helm show values bitnami/nginx | Out-File -Encoding UTF8 nginx-values.yaml
```

---

### สร้าง Custom Values File

สร้างไฟล์ `nginx-custom.yaml`:

**macOS / Linux:**
```bash
cat > nginx-custom.yaml <<EOF
global:
  security:
    allowInsecureImages: true

replicaCount: 2

image:
  registry: docker.io
  repository: nginx
  tag: 1.29.7

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
EOF
```

**Windows (PowerShell):**
```powershell
@"
global:
  security:
    allowInsecureImages: true

replicaCount: 2

image:
  registry: docker.io
  repository: nginx
  tag: 1.29.7

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
"@ | Out-File -Encoding UTF8 nginx-custom.yaml
```

**Windows (Notepad):**
- เปิด Notepad
- Copy YAML ข้างบนไปวาง
- Save as `nginx-custom.yaml` (UTF-8)

---

### Upgrade ด้วย Custom Values

```bash
helm upgrade my-web bitnami/nginx -f nginx-custom.yaml
```

---

### ตรวจสอบ

```bash
# ดู pods (ต้องมี 2 ตัว)
kubectl get pods

# ดู values ที่ใช้
helm get values my-web
```

**ผลลัพธ์ `helm get values`:**
```yaml
global:
  security:
    allowInsecureImages: true
replicaCount: 2
image:
  registry: docker.io
  repository: nginx
  tag: 1.29.7
service:
  port: 80
  type: ClusterIP
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

---

## Section 3: Dev vs Production Values

### สร้าง Values สำหรับ Development

สร้างไฟล์ `values-dev.yaml`:

```yaml
replicaCount: 1

service:
  type: ClusterIP

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

---

### สร้าง Values สำหรับ Production

สร้างไฟล์ `values-prod.yaml`:

```yaml
replicaCount: 3

service:
  type: LoadBalancer

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

---

### Deploy Development Environment

```bash
# Upgrade ด้วย dev values
helm upgrade my-web bitnami/nginx -f values-dev.yaml

# ตรวจสอบ
kubectl get pods
kubectl get svc my-web-nginx
```

**ต้องเห็น:**
- 1 pod
- Service = ClusterIP

---

### Deploy "Production" Environment

ติดตั้งอีก release (ชื่อต่างกัน):

```bash
helm install my-web-prod bitnami/nginx -f values-prod.yaml
```

---

### ตรวจสอบทั้ง 2 Environments

```bash
# ดู releases ทั้งหมด
helm list

# ดู pods ทั้งหมด
kubectl get pods

# ดู services
kubectl get svc
```

**ต้องเห็น:**
- `my-web`: 1 pod, ClusterIP (dev)
- `my-web-prod`: 3 pods, LoadBalancer (prod)

---

### เปรียบเทียบ Resources

```bash
# Dev
kubectl describe deployment my-web-nginx | grep -A 5 "Limits:"

# Prod
kubectl describe deployment my-web-prod-nginx | grep -A 5 "Limits:"
```

**ต้องเห็น:** Resource limits ต่างกัน

---

## Section 4: Values Priority

### ทดสอบ Priority

Values มีลำดับความสำคัญ:

```
--set  (สูงสุด)
  ↓
-f <file>
  ↓
values.yaml (ต่ำสุด)
```

---

### ทดลอง 1: file + --set

```bash
helm upgrade my-web bitnami/nginx \
  -f values-dev.yaml \
  --set replicaCount=5
```

**คำถาม:** จะมีกี่ pods?

```bash
kubectl get pods
```

---

### ทดลอง 2: หลาย files

```bash
helm upgrade my-web bitnami/nginx \
  -f values-dev.yaml \
  -f values-prod.yaml
```

---

### Dry-run: ทดสอบก่อนติดตั้งจริง

```bash
helm upgrade my-web bitnami/nginx \
  -f values-dev.yaml \
  --dry-run --debug
```

**ผลลัพธ์:** แสดง YAML ที่จะติดตั้ง (ไม่ติดตั้งจริง)

ใช้เพื่อ:
- ดูว่าจะได้อะไร
- เช็ค syntax
- ทดสอบ values

---

## Section 5: Cleanup

### ลบ Releases

```bash
helm uninstall my-web
helm uninstall my-web-prod
```

---

### ลบไฟล์

**macOS / Linux:**
```bash
rm nginx-values.yaml nginx-custom.yaml values-dev.yaml values-prod.yaml
```

**Windows (PowerShell):**
```powershell
Remove-Item nginx-values.yaml, nginx-custom.yaml, values-dev.yaml, values-prod.yaml
```

**Windows (CMD):**
```cmd
del nginx-values.yaml nginx-custom.yaml values-dev.yaml values-prod.yaml
```

---

### ตรวจสอบ

```bash
helm list
kubectl get pods
```

**ต้องเห็น:** ไม่มี releases และ pods

---

## สรุป

### คำสั่งสำคัญ:

| คำสั่ง | ทำอะไร |
|--------|--------|
| `helm upgrade` | อัพเกรด release |
| `helm rollback` | ย้อนกลับ revision |
| `helm history` | ดู history |
| `-f values.yaml` | ใช้ values file |
| `--set key=value` | Override values |
| `--dry-run` | ทดสอบก่อนติดตั้ง |

### Values Priority:

```
--set > -f (ไฟล์หลัง) > -f (ไฟล์แรก) > values.yaml
```

---

## Quiz

1. คำสั่ง upgrade คืออะไร?
2. คำสั่ง rollback คืออะไร?
3. `--set` กับ `-f` อันไหนมี priority สูงกว่า?
4. `--dry-run` ทำอะไร?

---

## Next Module

**[Module 3: สร้าง Helm Chart](../module-3/tutorial.md)**

---

## Tips

- แยก values ตาม environment เสมอ
- ใช้ `--dry-run` ก่อนติดตั้งจริง
- เก็บ values files ใน Git
- ตั้งชื่อ release ให้มีความหมาย
- ใช้ `helm get values` ดูค่าที่ใช้จริง