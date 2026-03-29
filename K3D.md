# k3d — Local Kubernetes Cluster

## k3d คืออะไร?

**k3d** เป็น wrapper ที่รัน **k3s** (Kubernetes แบบ lightweight ของ Rancher) อยู่ใน Docker container
ทำให้สร้าง Kubernetes cluster บนเครื่อง local ได้ในไม่กี่วินาที โดยไม่ต้องติดตั้ง VM

> เว็บไซต์หลัก: https://k3d.io

```
นักพัฒนา  →  k3d CLI  →  Docker containers  →  k3s cluster
```

**จุดเด่น:**
- สร้าง/ลบ cluster ได้ในไม่กี่วินาที
- รัน multi-node cluster บนเครื่องเดียว
- มี Ingress Controller (Traefik) และ LoadBalancer ในตัว
- เหมาะสำหรับ local development และ workshop

---

## Prerequisites

| สิ่งที่ต้องมี | ตรวจสอบ |
|---|---|
| Docker Desktop หรือ Docker Engine | `docker version` |
| macOS, Linux, หรือ Windows (WSL2) | — |
| RAM ว่างอย่างน้อย 2GB | — |

> **หมายเหตุ:** Docker ต้องรันอยู่ก่อนใช้ k3d ทุกครั้ง

---

## ติดตั้ง k3d

### macOS

```bash
brew install k3d
```

### Linux

```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

### Windows (PowerShell)

```powershell
choco install k3d
```

> Windows แนะนำใช้ผ่าน **WSL2** เพื่อประสิทธิภาพที่ดีกว่า

### ตรวจสอบการติดตั้ง

```bash
k3d version
```

---

## ติดตั้ง kubectl

### macOS

```bash
brew install kubectl
```

### Linux

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

### Windows (PowerShell)

```powershell
choco install kubernetes-cli
```

### ตรวจสอบ

```bash
kubectl version --client
```

---

## สร้าง Cluster สำหรับ Workshop

```bash
k3d cluster create workshop \
  --servers 1 \
  --agents 2 \
  --port "80:80@loadbalancer" \
  --port "443:443@loadbalancer"
```

| Flag | ความหมาย |
|---|---|
| `--servers 1` | จำนวน **Server node** (control plane) |
| `--agents 2` | จำนวน **Agent node** (worker) |
| `--port "80:80@loadbalancer"` | map port 80 ของเครื่องเข้า LoadBalancer ของ cluster (ใช้ใน lab 08) |

### Server vs Agent คืออะไร?

Kubernetes แบ่ง node ออกเป็น 2 บทบาท:

**Server (Control Plane)**
- สมองของ cluster — ตัดสินใจว่า Pod จะรันที่ node ไหน
- รัน component หลัก ได้แก่ `kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`
- ใน production มักมีหลาย server เพื่อ high availability
- ใน workshop นี้ใช้ 1 server ก็เพียงพอ

**Agent (Worker Node)**
- แรงงานของ cluster — รับ Pod มาจาก Server แล้วรันจริง
- รัน `kubelet` และ `container runtime` (containerd)
- Pod ของ workload ทั้งหมดจะถูก schedule มาที่ agent node
- ยิ่งมีหลาย agent ยิ่งรัน Pod ได้มากขึ้นและกระจาย workload ได้

```
┌─────────────────────────────────────────────┐
│                  k3d cluster                │
│                                             │
│  ┌──────────────┐    ┌───────┐  ┌───────┐   │
│  │    Server    │    │ Agent │  │ Agent │   │
│  │ (control     │───▶│  01   │  │  02   │   │
│  │   plane)     │    │       │  │       │   │
│  └──────────────┘    └───────┘  └───────┘   │
│     API สั่งงาน              Pod รันที่นี่         │
└─────────────────────────────────────────────┘
```

ตรวจสอบ cluster พร้อมใช้งาน:

```bash
kubectl get nodes
# ควรเห็น node สถานะ Ready
```

---

## Commands ที่ใช้บ่อยใน Workshop

### จัดการ Cluster

```bash
# ดู cluster ทั้งหมด
k3d cluster list

# หยุด cluster (ประหยัด resource ตอนไม่ใช้)
k3d cluster stop workshop

# เริ่ม cluster อีกครั้ง
k3d cluster start workshop

# ลบ cluster
k3d cluster delete workshop
```

### จัดการ kubeconfig

```bash
# merge kubeconfig ของ cluster เข้า ~/.kube/config (k3d ทำให้อัตโนมัติตอนสร้าง)
k3d kubeconfig merge workshop --kubeconfig-switch-context

# ดู context ปัจจุบัน
kubectl config current-context

# สลับ context
kubectl config use-context k3d-workshop
```

---

## Commands kubectl ที่ใช้ใน Workshop

### Namespace

```bash
kubectl apply -f 00-namespace/                              # สร้าง namespace workshop
kubectl config set-context --current --namespace=workshop   # ตั้ง default namespace
```

### ดู Resource

```bash
kubectl get pods                              # ดู pods ใน current namespace
kubectl get pods -w                           # ดู pods แบบ real-time (watch)
kubectl get pods -A                           # ดู pods ทุก namespace
kubectl get all                               # ดูทุก resource ใน namespace
kubectl get nodes                             # ดู nodes ทั้งหมดใน cluster
kubectl describe pod <name>                   # ดูรายละเอียด + events ของ pod
kubectl logs <pod-name>                       # ดู logs ของ container
kubectl logs -f <pod-name>                    # ติดตาม logs แบบ real-time
kubectl logs <pod-name> --previous            # ดู logs ของ container ที่ crash ไปแล้ว
```

### แก้ไข / Scale

```bash
kubectl apply -f <file-or-dir>                # สร้างหรืออัปเดต resource จากไฟล์
kubectl delete -f <file-or-dir>               # ลบ resource ตามไฟล์
kubectl scale deployment <name> --replicas=3  # ปรับจำนวน replicas
kubectl rollout restart deployment <name>     # restart pods ทั้งหมดใน deployment
kubectl rollout status deployment <name>      # ดูสถานะการ deploy
kubectl rollout undo deployment <name>        # rollback ไป version ก่อนหน้า
```

### Debug

```bash
kubectl exec -it <pod-name> -- sh             # เข้า shell ใน container
kubectl exec -it <pod-name> -- env            # ดู environment variables ใน container
kubectl port-forward pod/<name> 8080:80       # forward port จาก pod มาที่เครื่อง
kubectl port-forward service/<name> 8080:80   # forward port จาก service มาที่เครื่อง
kubectl run -it --rm debug \
  --image=busybox:1.36 --restart=Never -- sh  # รัน pod ชั่วคราวสำหรับ debug
kubectl cp <pod-name>:/path/to/file ./file    # copy ไฟล์ออกจาก container
```

### Events และ Resource Usage

```bash
kubectl get events --sort-by='.lastTimestamp' # ดู events เรียงตามเวลา
kubectl top nodes                             # ดู CPU/Memory ของแต่ละ node
kubectl top pods                              # ดู CPU/Memory ของแต่ละ pod
```

---

## Cleanup หลังจบ Workshop

```bash
# ลบทุก resource ใน namespace workshop
kubectl delete namespace workshop

# ลบ cluster ทิ้งถ้าไม่ใช้แล้ว
k3d cluster delete workshop
```
