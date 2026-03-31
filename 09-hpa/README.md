# Lab 09 — HorizontalPodAutoscaler (HPA)

## Objective
เรียนรู้ **HPA** ที่ scale จำนวน Pod อัตโนมัติตาม CPU utilization

## Files
- `deployment.yaml` — Deployment ของ Python HTTP server ที่ burn CPU ต่อ request (รองรับ amd64 และ arm64)
- `service.yaml` — ClusterIP Service
- `hpa.yaml` — HPA ที่ scale ระหว่าง 1–5 replicas เมื่อ CPU เกิน 50%
- `load-generator.yaml` — Pod busybox ที่ส่ง request ซ้ำๆ เพื่อสร้าง load

> **Prerequisites:** cluster ต้องมี Metrics Server
> ตรวจสอบ: `kubectl top nodes` ต้องได้ผลออกมา

## ติดตั้ง Metrics Server

### ตรวจสอบก่อนว่ามีแล้วหรือยัง

```bash
kubectl top nodes
# ถ้าได้ข้อมูล CPU/Memory → พร้อมใช้งาน ข้ามขั้นตอนนี้ได้เลย
```

### k3d

k3d bundle Metrics Server มาให้ในตัว ไม่ต้องติดตั้งเพิ่ม

### minikube

```bash
minikube addons enable metrics-server
```

### kind

```bash
# ติดตั้ง Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# kind ใช้ self-signed cert ต้องเพิ่ม --kubelet-insecure-tls
kubectl patch deployment metrics-server -n kube-system --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

### k3s

```bash
# k3s มี Metrics Server เป็น addon ติดตั้งด้วย flag ตอนสร้าง cluster
# ถ้ายังไม่มีให้ติดตั้งเหมือน kind ข้างบน
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl patch deployment metrics-server -n kube-system --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

### ตรวจสอบหลังติดตั้ง

```bash
# รอ pod พร้อม
kubectl rollout status deployment/metrics-server -n kube-system

# ทดสอบ (อาจต้องรอ ~1 นาทีหลัง pod พร้อม)
kubectl top nodes
kubectl top pods
```

## Apply

```bash
# Apply deployment และ HPA ก่อน (ยังไม่ apply load-generator)
kubectl apply -f 09-hpa/deployment.yaml
kubectl apply -f 09-hpa/service.yaml
kubectl apply -f 09-hpa/hpa.yaml
```

## Verify

```bash
# ดู HPA (TARGETS จะแสดง current CPU / target CPU)
kubectl get hpa workshop-hpa

# ดู pods (เริ่มต้น 1 replica)
kubectl get pods -l app=workshop-hpa-app

# ดู resource usage (รอสักครู่ให้ metrics เก็บข้อมูล)
kubectl top pods -l app=workshop-hpa-app
```

## Test — สร้าง Load เพื่อดู Scale Up

```bash
# Step 1: เปิด terminal แยก แล้ว watch HPA
kubectl get hpa workshop-hpa -w

# Step 2: เปิดอีก terminal แล้ว apply load-generator
kubectl apply -f 09-hpa/load-generator.yaml

# Step 3: รอสักครู่ (1-2 นาที) แล้วดู HPA scale up
# REPLICAS จะเพิ่มจาก 1 → 2, 3, ... (สูงสุด 5)
kubectl get pods -l app=workshop-hpa-app -w
```

### ดู Scale Down

```bash
# ลบ load-generator เพื่อหยุด load
kubectl delete pod load-generator

# HPA จะ scale down กลับภายใน ~5 นาที (default cool-down period)
kubectl get hpa workshop-hpa -w
```

### ดูรายละเอียด HPA

```bash
kubectl describe hpa workshop-hpa
# ดู events: scale up/down history
```

## Cleanup

```bash
kubectl delete -f 09-hpa/
```

## Key Takeaways
- HPA ต้องการ **Metrics Server** เพื่ออ่าน CPU/Memory ของ Pod
- **resource.requests** ต้องกำหนดเสมอ ไม่เช่นนั้น HPA คำนวณ % ไม่ได้
- Scale up เร็วกว่า scale down (ป้องกัน thrashing)
- HPA ทำงานร่วมกับ Deployment/StatefulSet/ReplicaSet
- `minReplicas` และ `maxReplicas` เป็น safety boundary สำคัญ
- HPA v2 รองรับ custom metrics นอกจาก CPU/Memory ได้ด้วย
