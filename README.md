# Helm Workshop - เรียนแบบปฏิบัติ 100%

## ภาพรวม Workshop

หลักสูตรนี้เป็น **hands-on workshop** ที่:
- เรียนจาก tutorial โดยตรง
- เน้นปฏิบัติ 100%
- อธิบายสั้นๆ ตรงประเด็น
- รองรับทุก OS - Windows, macOS, Linux

---

## Workshop Modules

### Module 1: เริ่มต้นกับ Helm
> ติดตั้ง Helm, ใช้งานพื้นฐาน, ค้นหาและติดตั้ง application แรก

---

### Module 2: จัดการ Applications
> Install, upgrade, rollback, customize applications ด้วย values

---

### Module 3: สร้าง Helm Chart
> สร้าง Chart ของตัวเอง สำหรับ web application

---

## ความต้องการระบบ

### ต้องมี:
- **Kubernetes Cluster**:
  - minikube, k3d, k3s (แนะนำสำหรับ local)
  - Docker Desktop (macOS/Windows)
  - Cloud cluster (GKE, EKS, AKS)

- **kubectl**: ติดตั้งและตั้งค่าเรียบร้อย

- **Terminal/Command Prompt**:
  - Windows: PowerShell หรือ CMD
  - macOS: Terminal
  - Linux: Terminal

### จะติดตั้งใน Workshop:
- Helm 3.x

---

## เริ่มต้น

### ก่อนเริ่ม Workshop:

#### 1. เช็ค Kubernetes Cluster
```bash
kubectl cluster-info
kubectl get nodes
```

#### 2. ถ้ายังไม่มี Cluster:

**macOS/Linux (minikube):**
```bash
brew install minikube
minikube start
```

**Windows (minikube):**
```powershell
choco install minikube
minikube start
```

หรือใช้ **Docker Desktop** (เปิด Kubernetes ใน Settings)

---

## วิธีใช้ Workshop

### เรียนแต่ละ Module:

1. **เปิดไฟล์ tutorial.md** ของ module นั้น
2. **อ่านทีละ section** - มี theory สั้นๆ + ทำทันที
3. **ทำตามคำสั่ง** - copy-paste และรันได้เลย
4. **ดูผลลัพธ์** - ตรวจสอบว่าถูกต้อง

### รูปแบบคำสั่ง:

เราจะให้คำสั่งสำหรับทุก OS:

```bash
# macOS / Linux
helm version

# Windows (PowerShell)
helm version

# Windows (CMD)
helm version
```

---

## Tips

- เตรียม text editor สำหรับแก้ไขไฟล์
- เปิด 2 terminals (1 สำหรับคำสั่ง, 1 สำหรับ monitor)
- ลองผิดลองถูก - ไม่ต้องกลัวทำพัง

---

## สิ่งที่จะได้เรียนรู้

หลัง workshop จบ:

- ติดตั้งและใช้งาน Helm CLI
- ค้นหาและติดตั้ง applications จาก Helm Hub
- Customize applications ด้วย values
- Upgrade และ rollback applications
- สร้าง Helm Chart ของตัวเอง
- Deploy application ด้วย Chart ที่สร้าง

---

## Help

- [Helm Cheat Sheet](CHEAT-SHEET.md)
- [Helm Official Docs](https://helm.sh/docs/)

---

## Prerequisites Check

ก่อนเริ่ม ตรวจสอบว่าพร้อมหรือยัง:

- [ ] มี Kubernetes cluster ที่ทำงานได้
- [ ] kubectl ใช้งานได้
- [ ] เปิด terminal/command prompt ได้
- [ ] มี internet connection
- [ ] พร้อมที่จะเรียนรู้

---

**[Module 1: เริ่มต้นกับ Helm](./module-1/tutorial.md)**
