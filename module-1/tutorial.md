# Module 1: เริ่มต้นกับ Helm

> ติดตั้ง, ใช้งานพื้นฐาน, deploy application แรก

---

## สิ่งที่จะทำใน Module นี้

1. ติดตั้ง Helm
2. เช็คว่า Helm ทำงานได้
3. เพิ่ม Helm Repository
4. ค้นหา Charts
5. ติดตั้ง Application แรก (NGINX)
6. ทดสอบ Application
7. ลบ Application

---

## Section 1: Helm คืออะไร?

**Helm = Package Manager สำหรับ Kubernetes**

เหมือนกับ:
- macOS → **Homebrew** (`brew install ...`)
- Linux → **apt/yum** (`apt install ...`)
- Windows → **Chocolatey** (`choco install ...`)
- Kubernetes → **Helm** (`helm install ...`)

### ทำไมต้องใช้ Helm?

**ปัญหาเดิม**: Deploy app บน Kubernetes ต้องจัดการหลายไฟล์
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
# ...และอีกมากมาย
```

**ใช้ Helm**: รวมทุกอย่างเป็น package เดียว
```bash
helm install my-app chart-name
```

### คำศัพท์ 3 ตัวที่ต้องรู้:

- **Chart** = Package (ไฟล์ที่รวม Kubernetes resources)
- **Release** = Chart ที่ติดตั้งแล้ว (มีชื่อ)
- **Values** = Configuration ที่ปรับแต่งได้

---

## Section 2: ติดตั้ง Helm

### เช็คว่ามี Kubernetes หรือยัง

เปิด Terminal/PowerShell แล้วรันคำสั่ง:

```bash
kubectl cluster-info
kubectl get nodes
```

**ผลลัพธ์ควรเห็น:** Cluster กำลังทำงาน

ถ้ายังไม่มี → [Prerequisites](../README.md#-ความต้องการระบบ)

---

### ติดตั้ง Helm

เลือกตาม OS ที่คุณใช้:

#### macOS

```bash
# ใช้ Homebrew
brew install helm

# ตรวจสอบ
helm version
```

#### Linux

```bash
# ใช้ script ติดตั้ง
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# ตรวจสอบ
helm version
```

#### Windows

**Option 1: Chocolatey (แนะนำ)**
```powershell
# เปิด PowerShell as Administrator
choco install kubernetes-helm

# ตรวจสอบ
helm version
```

**Option 2: Scoop**
```powershell
scoop install helm

# ตรวจสอบ
helm version
```

**Option 3: Manual Download**
1. ดาวน์โหลดจาก: https://github.com/helm/helm/releases
2. แตกไฟล์ `helm.exe`
3. เพิ่มเข้า PATH
4. ตรวจสอบ: `helm version`

---

### Checkpoint: ตรวจสอบการติดตั้ง

รันคำสั่ง:

```bash
helm version
```

**ผลลัพธ์ควรเห็น:**
```
version.BuildInfo{Version:"v3.14.x", ...}
```

**ต้องเป็น Helm 3.x** (ไม่ใช่ 2.x)


---

## Section 3: เพิ่ม Helm Repository

**Repository** = แหล่งเก็บ Charts (เหมือน App Store สำหรับ Helm)

### เพิ่ม Bitnami Repository

Bitnami = Repository ยอดนิยม มี Charts มากมาย

```bash
# เพิ่ม repository
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**ผลลัพธ์:**
```
"bitnami" has been added to your repositories
```

### ดู Repositories ที่มี

```bash
helm repo list
```

**ผลลัพธ์:**
```
NAME    URL
bitnami https://charts.bitnami.com/bitnami
```

### Update Repository

```bash
helm repo update
```

**ผลลัพธ์:**
```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```

---

### Checkpoint: ตรวจสอบ Repository

```bash
helm repo list
```

**ต้องเห็น:** `bitnami` repository

---

## Section 4: ค้นหา Charts

ลองค้นหา Charts ที่มี:

### ค้นหา NGINX

```bash
helm search repo nginx
```

**ผลลัพธ์:** (จะเห็นหลาย charts)
```
NAME                   CHART VERSION   APP VERSION   DESCRIPTION
bitnami/nginx          15.4.4          1.25.3        NGINX Open Source...
bitnami/nginx-ingress  ...             ...           ...
```

### ค้นหา MySQL

```bash
helm search repo mysql
```

### ค้นหา WordPress

```bash
helm search repo wordpress
```

### ดูข้อมูล Chart

ลองดูข้อมูลของ nginx chart:

```bash
# ดูข้อมูลทั่วไป
helm show chart bitnami/nginx

# ดู values ที่ปรับแต่งได้
helm show values bitnami/nginx | head -30
```

---

## Section 5: ติดตั้ง Application แรก

มาติดตั้ง NGINX กันเลย!

### ติดตั้ง NGINX

```bash
helm install my-nginx bitnami/nginx
```

**ผลลัพธ์:** จะแสดงข้อมูลและคำแนะนำ

**อธิบาย:**
- `my-nginx` = ชื่อ release (ตั้งเองได้)
- `bitnami/nginx` = chart ที่จะติดตั้ง

### ดู Releases ที่ติดตั้ง

```bash
helm list
```

**ผลลัพธ์:**
```
NAME      NAMESPACE   REVISION   STATUS     CHART          APP VERSION
my-nginx  default     1          deployed   nginx-15.4.4   1.25.3
```

### ตรวจสอบ Kubernetes Resources

```bash
# ดู Pods
kubectl get pods

# ดู Services
kubectl get svc

# ดู Deployments
kubectl get deployments
```

**จะเห็น:** resources ที่ Helm สร้างให้

---

### Checkpoint: Application ทำงานหรือยัง?

```bash
kubectl get pods
```

**ต้องเห็น:** Pod STATUS = `Running`

ถ้ายัง `ContainerCreating` → รอสักครู่

---

## Section 6: ทดสอบ Application

### Port Forward

เราจะเชื่อมต่อกับ NGINX ผ่าน port forward:

```bash
kubectl port-forward svc/my-nginx 8080:80
```

**ผลลัพธ์:**
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

**⚠️ อย่าปิด terminal นี้!** ให้รันอยู่

---

### ทดสอบด้วย Browser

**เปิด browser ใหม่** แล้วไปที่:
```
http://localhost:8080
```

**ควรเห็น:** หน้า NGINX welcome page!

---

### ทดสอบด้วย curl (ถ้ามี)

**เปิด terminal ใหม่** (อันเดิมยัง port-forward อยู่):

```bash
curl http://localhost:8080
```

**ผลลัพธ์:** HTML ของ NGINX

---

### หยุด Port Forward

กลับไปที่ terminal ที่รัน port-forward

**กด:** `Ctrl + C` เพื่อหยุด

---

## Section 7: ดูข้อมูล Release

### ดูสถานะ

```bash
helm status my-nginx
```

**ผลลัพธ์:** แสดงสถานะและคำแนะนำ

### ดู Manifest

ดู YAML ที่ Helm ติดตั้ง:

```bash
helm get manifest my-nginx
```

**ผลลัพธ์:** YAML ทั้งหมด (Deployment, Service, etc.)

### ดู Values ที่ใช้

```bash
# ดูเฉพาะที่เรา override (ตอนนี้ยังไม่มี)
helm get values my-nginx

# ดูทั้งหมด (รวม default)
helm get values my-nginx --all | head -50
```

---

## Section 8: ลบ Application

### Uninstall

```bash
helm uninstall my-nginx
```

**ผลลัพธ์:**
```
release "my-nginx" uninstalled
```

### ตรวจสอบ

```bash
# ดู releases (ไม่มีแล้ว)
helm list

# ดู pods (ไม่มีแล้ว)
kubectl get pods
```

---

### Checkpoint: ลบสำเร็จ

```bash
helm list
```

**ต้องเห็น:** ไม่มี releases

---

## สรุป

### คำสั่งสำคัญที่เรียนรู้:

| คำสั่ง | ทำอะไร |
|--------|--------|
| `helm version` | ตรวจสอบ Helm version |
| `helm repo add` | เพิ่ม repository |
| `helm repo list` | ดู repositories |
| `helm search repo` | ค้นหา charts |
| `helm install` | ติดตั้ง chart |
| `helm list` | ดู releases |
| `helm status` | ดูสถานะ release |
| `helm uninstall` | ลบ release |

---

## สิ่งที่ควรจำ

1. **Chart** = Package ที่รวม Kubernetes resources
2. **Release** = Chart ที่ติดตั้งแล้ว (มีชื่อ)
3. **Repository** = แหล่งเก็บ charts
4. Helm ช่วยให้จัดการ applications ง่ายขึ้น (ติดตั้ง/ลบ ด้วยคำสั่งเดียว)

---

## Quiz เช็คความเข้าใจ

1. Helm คืออะไร?
2. Chart กับ Release ต่างกันอย่างไร?
3. คำสั่งติดตั้ง chart คืออะไร?
4. คำสั่งดู releases คืออะไร?

---

## Next Module

**[Module 2: จัดการ Applications](../module-2/tutorial.md)**

---

## Troubleshooting

### ปัญหาที่พบบ่อย:

**1. `helm: command not found`**
- ติดตั้ง Helm ใหม่
- เช็คว่าอยู่ใน PATH หรือยัง

**2. `Error: Kubernetes cluster unreachable`**
- เช็ค: `kubectl cluster-info`
- Start cluster: `minikube start`

**3. Pod ไม่ทำงาน (CrashLoopBackOff)**
- ดู logs: `kubectl logs <pod-name>`
- ดู events: `kubectl get events`

**4. Port forward ไม่ทำงาน**
- เช็คว่า pod running แล้วหรือยัง
- ลองเปลี่ยน port: `8081:80`