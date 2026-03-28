# Lab 02 — Deployment

## Objective
เรียนรู้ **Deployment** ที่จัดการ Pod ให้มี replicas, self-healing, และ rolling update

## Files
- `deployment.yaml` — Deployment ชื่อ `workshop-web` รัน 2 replicas ของ nginx

## Apply

```bash
kubectl apply -f 02-deployment/
```

## Verify

```bash
# ดู deployment
kubectl get deployment workshop-web

# ดู pods ที่สร้างมาจาก deployment
kubectl get pods -l app=workshop-web

# ดูสถานะ rollout
kubectl rollout status deployment/workshop-web

# ดู ReplicaSet ที่ deployment สร้างขึ้น
kubectl get replicaset
```

## Test

```bash
# ทดสอบ self-healing: ลบ pod แล้วดูว่า deployment สร้างใหม่
kubectl delete pod -l app=workshop-web --wait=false
kubectl get pods -w   # watch ดูว่า pod ใหม่ขึ้นมา

# Scale replicas
kubectl scale deployment workshop-web --replicas=4
kubectl get pods

# Scale กลับ
kubectl scale deployment workshop-web --replicas=2
```

## Rolling Update

```bash
# เปลี่ยน image version (simulate update)
kubectl set image deployment/workshop-web web=nginx:1.26-alpine

# ดูสถานะ rolling update
kubectl rollout status deployment/workshop-web

# ดู history
kubectl rollout history deployment/workshop-web

# Rollback
kubectl rollout undo deployment/workshop-web
```

## Cleanup

```bash
kubectl delete -f 02-deployment/
```

## Key Takeaways
- Deployment สร้าง ReplicaSet ที่ควบคุม Pod ให้ครบ replicas เสมอ
- ถ้า Pod crash หรือถูกลบ Deployment จะสร้าง Pod ใหม่แทน (self-healing)
- Rolling update ช่วยให้ update โดยไม่ downtime
- `selector.matchLabels` ต้องตรงกับ `template.labels` ไม่เช่นนั้น Deployment ไม่รู้ว่า Pod ไหนเป็นของตัวเอง
