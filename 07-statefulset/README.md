# Lab 07 — StatefulSet

## Objective
เรียนรู้ **StatefulSet** ที่จัดการ Pod ที่ต้องการ **stable identity** และ **persistent storage** แยกต่างหากต่อ Pod

## Files
- `headless-service.yaml` — Headless Service สำหรับ stable DNS ของแต่ละ Pod
- `statefulset.yaml` — StatefulSet 3 replicas พร้อม volumeClaimTemplates

## Deployment vs StatefulSet

| | Deployment | StatefulSet |
|---|---|---|
| Pod name | random (workshop-web-**abc12**) | stable (workshop-web-**0**, **1**, **2**) |
| Scale order | ไม่แน่นอน | สร้างตามลำดับ 0→1→2, ลบย้อน 2→1→0 |
| Storage | shared หรือไม่มี | แยก PVC ต่อ Pod (volumeClaimTemplates) |
| DNS | Service → load balance | Pod แต่ละตัวมี DNS ของตัวเอง |
| Use case | stateless apps | databases, queues, stateful clusters |

## Apply

```bash
kubectl apply -f 07-statefulset/
```

## Verify

```bash
# ดู pods (สังเกต name: workshop-web-0, -1, -2)
kubectl get pods -l app=workshop-stateful

# ดู PVC ที่ถูกสร้างอัตโนมัติต่อ Pod
kubectl get pvc

# ดู StatefulSet
kubectl get statefulset workshop-web
```

## Test

### Stable Pod Identity

```bash
# Pod แต่ละตัวรู้จัก hostname ตัวเอง
kubectl exec workshop-web-0 -- hostname   # workshop-web-0
kubectl exec workshop-web-1 -- hostname   # workshop-web-1
kubectl exec workshop-web-2 -- hostname   # workshop-web-2

# ดู HTML ที่แต่ละ Pod แสดง (จะต่างกัน)
kubectl exec workshop-web-0 -- cat /usr/share/nginx/html/index.html
kubectl exec workshop-web-1 -- cat /usr/share/nginx/html/index.html
```

### DNS Resolution ต่อ Pod

```bash
# แต่ละ Pod มี DNS ของตัวเอง: <pod-name>.<service-name>.<namespace>.svc.cluster.local
kubectl run -it --rm debug --image=busybox:1.36 --restart=Never -- nslookup workshop-web-0.workshop-web-headless.workshop.svc.cluster.local
```

### แยก PVC ต่อ Pod

```bash
# เขียนไฟล์ต่างกันใน Pod 0 และ Pod 1
kubectl exec workshop-web-0 -- sh -c "echo 'pod-0 data' > /data/test.txt"
kubectl exec workshop-web-1 -- sh -c "echo 'pod-1 data' > /data/test.txt"

# ข้อมูลแยกกัน
kubectl exec workshop-web-0 -- cat /data/test.txt   # pod-0 data
kubectl exec workshop-web-1 -- cat /data/test.txt   # pod-1 data
```

## Cleanup

```bash
kubectl delete -f 07-statefulset/
# PVC จาก volumeClaimTemplates ต้องลบแยก
kubectl delete pvc -l app=workshop-stateful
```

## Key Takeaways
- StatefulSet ให้ Pod มีชื่อ stable เช่น `workshop-web-0` แทนที่จะสุ่ม
- **Headless Service** (`clusterIP: None`) ช่วยให้ resolve DNS ไปยัง Pod โดยตรง
- **volumeClaimTemplates** สร้าง PVC แยกต่อ Pod อัตโนมัติ — Pod 0 ได้ `data-workshop-web-0`
- Scale down ลบ Pod จากหมายเลขสูงก่อน (2→1→0), scale up สร้างตามลำดับ (0→1→2)
- เหมาะกับ: databases, Kafka, Zookeeper, Elasticsearch
