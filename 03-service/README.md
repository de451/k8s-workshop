# Lab 03 — Service

## Objective
เรียนรู้ **Service** ที่ทำหน้าที่เป็น stable endpoint สำหรับเข้าถึง Pod — แม้ Pod IP จะเปลี่ยนก็ตาม

## Files
- `deployment.yaml` — Deployment ชื่อ `workshop-web` (2 replicas)
- `service.yaml` — ClusterIP Service ชื่อ `workshop-web-svc`

## ทำไม Service จึงจำเป็น?

Pod มี IP เป็นของตัวเอง แต่ IP จะเปลี่ยนทุกครั้งที่ Pod restart หรือถูก reschedule
Service แก้ปัญหานี้ด้วย:
- **Stable IP / DNS name** ที่ไม่เปลี่ยน
- **Load balancing** กระจาย traffic ไปยัง Pod ทุกตัวที่ match selector
- **Service discovery** ภายใน cluster ผ่าน DNS (`workshop-web-svc.workshop.svc.cluster.local`)

## Apply

```bash
kubectl apply -f 03-service/
```

## Verify

```bash
# ดู service
kubectl get service workshop-web-svc

# ดู endpoints (IP ของ Pods ที่ Service กำลัง load balance ถึง)
kubectl get endpoints workshop-web-svc

# ดูรายละเอียด
kubectl describe service workshop-web-svc
```

## Test

```bash
# เข้าถึง Service ผ่าน port-forward
kubectl port-forward service/workshop-web-svc 8080:80
# เปิดอีก terminal: curl http://localhost:8080

# ทดสอบ DNS resolution จากภายใน cluster
kubectl run -it --rm debug --image=busybox:1.36 --restart=Never -- \
  wget -qO- http://workshop-web-svc.workshop.svc.cluster.local
```

## Cleanup

```bash
kubectl delete -f 03-service/
```

## Key Takeaways
- Service ใช้ `selector` จับ Pod ที่มี label ตรงกัน
- `port` คือ port ของ Service, `targetPort` คือ port ของ Pod
- ClusterIP เข้าถึงได้เฉพาะภายใน cluster
- DNS name รูปแบบ: `<service-name>.<namespace>.svc.cluster.local`
