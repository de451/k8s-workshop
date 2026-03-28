# Kubernetes Workshop (3 Hours)

Workshop นี้ครอบคลุมพื้นฐาน Kubernetes ผ่าน 9 labs ที่เรียงต่อเนื่องกัน ทุก lab apply แยกเดี่ยวได้และทำงานบน local cluster

---

## Learning Path

| # | Lab | หัวข้อ | เวลา |
|---|-----|--------|------|
| 00 | namespace | สร้าง namespace สำหรับ workshop | 5 นาที |
| 01 | pod | Pod เดี่ยว, lifecycle basics | 15 นาที |
| 02 | deployment | ReplicaSet, rollout, scaling | 15 นาที |
| 03 | service | ClusterIP, service discovery | 15 นาที |
| 04 | configmap-secret | Config และ sensitive data | 20 นาที |
| 05 | probes | Liveness & Readiness probe | 20 นาที |
| 06 | storage-pv-pvc | PersistentVolumeClaim, storage | 20 นาที |
| 07 | statefulset | Stable identity, StatefulSet | 20 นาที |
| 08 | ingress | HTTP routing, Ingress | 15 นาที |
| 09 | hpa | HorizontalPodAutoscaler | 15 นาที |

---

## Prerequisites

- `kubectl` CLI ติดตั้งแล้ว
- Local Kubernetes cluster (k3d, kind, minikube, หรือ k3s)
- Ingress Controller (ใช้ใน lab 08)
- Metrics Server (ใช้ใน lab 09)

---

## Cluster Setup

### k3d (แนะนำ)
```bash
# ติดตั้ง k3d
brew install k3d

# สร้าง cluster (มี Ingress Controller และ Metrics Server ในตัว)
k3d cluster create workshop --port "8080:80@loadbalancer" --port "8443:443@loadbalancer"

# ตรวจสอบ
kubectl get nodes
```

### kind
```bash
# สร้าง cluster
kind create cluster --name workshop

# ติดตั้ง Ingress Controller (nginx)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# ติดตั้ง Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
# kind ต้องแก้ไข args เพิ่ม --kubelet-insecure-tls
kubectl patch deployment metrics-server -n kube-system \
  --type=json \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

### minikube
```bash
# สร้าง cluster พร้อม addons
minikube start
minikube addons enable ingress
minikube addons enable metrics-server
```

---

## ตรวจสอบ Dependencies

### Ingress Controller
```bash
kubectl get pods -n ingress-nginx
# หรือ
kubectl get pods -n kube-system | grep ingress
```

### Metrics Server
```bash
kubectl top nodes
# ถ้าได้ข้อมูล แสดงว่า metrics-server พร้อมใช้
```

---

## ลำดับการ Apply Labs

```bash
# เริ่มต้นที่ namespace ก่อนเสมอ
kubectl apply -f 00-namespace/

# จากนั้น apply ทีละ lab
kubectl apply -f 01-pod/
kubectl apply -f 02-deployment/
# ...
```

---

## Cleanup ทั้ง Workshop

```bash
# ลบทุก resource ใน namespace workshop
kubectl delete namespace workshop

# หรือลบทีละ lab (ย้อนหลังจาก lab หลังสุด)
kubectl delete -f 09-hpa/
kubectl delete -f 08-ingress/
# ...
```

---

## Namespace

ทุก lab ใช้ namespace `workshop` เพื่อให้ cleanup ง่าย และแยกออกจาก workload อื่น
