# Lab 01 — Pod

## Objective
เรียนรู้ building block พื้นฐานที่สุดของ Kubernetes คือ **Pod** ซึ่งเป็น unit เล็กสุดที่ schedule ได้

## Files
- `pod.yaml` — สร้าง Pod เดี่ยวชื่อ `workshop-web` รัน nginx

## Apply

```bash
kubectl apply -f 01-pod/
```

## Verify

```bash
# ดู pod ที่รันอยู่
kubectl get pods

# ดูรายละเอียด pod (events, conditions, IP)
kubectl describe pod workshop-web

# ดู logs จาก container
kubectl logs workshop-web

# ติดตาม logs แบบ real-time
kubectl logs -f workshop-web
```

## Test

```bash
# exec เข้าไปใน container
kubectl exec -it workshop-web -- sh

# ทดสอบ HTTP จาก port-forward (เปิด terminal อีกอัน)
kubectl port-forward pod/workshop-web 8080:80
# จากนั้น: curl http://localhost:8080
```

## Cleanup

```bash
kubectl delete -f 01-pod/
# หรือ
kubectl delete pod workshop-web
```

## Key Takeaways
- Pod คือ wrapper ของ container หนึ่งหรือหลายตัว
- Pod มี IP address ของตัวเอง แต่ IP จะหายไปเมื่อ Pod ถูกลบ
- Pod เดี่ยวไม่มี self-healing — ถ้า crash จะไม่ restart อัตโนมัติ (ไม่มี controller)
- ใช้ `containerPort` เพื่อ document port แต่ไม่ได้ expose ออก cluster อัตโนมัติ
