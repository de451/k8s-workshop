# k3d — Local Kubernetes Cluster

## k3d คืออะไร?

**k3d** เป็น wrapper ที่รัน **k3s** (Kubernetes แบบ lightweight ของ Rancher) อยู่ใน Docker container
ทำให้สร้าง Kubernetes cluster บนเครื่อง local ได้ในไม่กี่วินาที โดยไม่ต้องติดตั้ง VM

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

### ตรวจสอบ

```bash
kubectl version --client
```

---

## สร้าง Cluster สำหรับ Workshop

```bash
k3d cluster create workshop \
  --port "80:80@loadbalancer" \
  --port "443:443@loadbalancer"
```

- `--port "80:80@loadbalancer"` — map port 80 ของเครื่องเข้า LoadBalancer ของ cluster (ใช้ใน lab 08)

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
kubectl apply -f 00-namespace/
kubectl config set-context --current --namespace=workshop
```

### ดู Resource

```bash
kubectl get pods
kubectl get pods -w                        # watch real-time
kubectl get all                            # ดูทุก resource
kubectl describe pod <name>
kubectl logs <pod-name>
kubectl logs -f <pod-name>                 # follow logs
```

### แก้ไข / Scale

```bash
kubectl apply -f <file-or-dir>
kubectl delete -f <file-or-dir>
kubectl scale deployment <name> --replicas=3
kubectl rollout restart deployment <name>
```

### Debug

```bash
kubectl exec -it <pod-name> -- sh          # เข้า shell ใน container
kubectl port-forward pod/<name> 8080:80    # forward port มาที่เครื่อง
kubectl port-forward service/<name> 8080:80
kubectl run -it --rm debug --image=busybox:1.36 --restart=Never -- sh
```

### Events และ Resource Usage

```bash
kubectl get events --sort-by='.lastTimestamp'
kubectl top nodes
kubectl top pods
```

---

## Cleanup หลังจบ Workshop

```bash
# ลบทุก resource ใน namespace workshop
kubectl delete namespace workshop

# ลบ cluster ทิ้งถ้าไม่ใช้แล้ว
k3d cluster delete workshop
```
