# Lab 05 — Liveness & Readiness Probes

## Objective
เรียนรู้ความแตกต่างระหว่าง **Liveness Probe** และ **Readiness Probe** และวิธีที่ Kubernetes ใช้เพื่อดูแล health ของ Pod

## Files
- `configmap.yaml` — nginx config ที่มี `/healthz` และ `/ready` endpoints
- `deployment.yaml` — Deployment พร้อม liveness + readiness probe
- `service.yaml` — ClusterIP Service

## ความแตกต่าง Liveness vs Readiness

| | Liveness Probe | Readiness Probe |
|---|---|---|
| คำถาม | "ยังมีชีวิตอยู่ไหม?" | "พร้อมรับ traffic ไหม?" |
| ถ้า fail | **Restart container** | **เอาออกจาก Service** (ไม่ restart) |
| ใช้กับ | deadlock, frozen app | startup ช้า, dependencies ยังไม่พร้อม |

## Apply

```bash
kubectl apply -f 05-probes/
```

## Verify

```bash
kubectl get pods -l app=workshop-probe
kubectl describe pod -l app=workshop-probe | grep -A 15 "Liveness\|Readiness"
```

## Test

### ทดสอบ Probe Endpoints

```bash
kubectl port-forward service/workshop-probe-svc 8080:80
# เปิดอีก terminal:
curl http://localhost:8080/healthz   # ควรได้ OK
curl http://localhost:8080/ready     # ควรได้ READY
```

### สังเกต Probe Events

```bash
# ดู events ของ pod
kubectl describe pod -l app=workshop-probe | tail -20
```

### จำลอง Liveness Failure (optional demo)

```bash
# exec เข้า container แล้วลบ nginx binary เพื่อจำลอง crash
# (ทำใน lab environment เท่านั้น)
POD=$(kubectl get pod -l app=workshop-probe -o jsonpath='{.items[0].metadata.name}')

# ดู restart count ก่อน
kubectl get pod $POD

# ลบ nginx process (liveness จะ fail → container restart)
kubectl exec $POD -- nginx -s stop

# ดู pod restart (RESTARTS จะเพิ่มขึ้น)
kubectl get pod $POD -w
```

## Cleanup

```bash
kubectl delete -f 05-probes/
```

## Key Takeaways
- **Liveness** ป้องกัน zombie process — ถ้า fail จะ restart container
- **Readiness** ป้องกัน traffic ไปยัง Pod ที่ยังไม่พร้อม — ถ้า fail จะเอาออกจาก endpoints
- `initialDelaySeconds` สำคัญมาก — ตั้งให้นานพอให้ app start ก่อน
- probe ทั้งคู่ทำงานแยกกัน สามารถมีค่า threshold ต่างกันได้
- probe type มี 3 แบบ: `httpGet`, `exec`, `tcpSocket`
