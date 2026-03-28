# Lab 06 — Storage: PV & PVC

## Objective
เรียนรู้การใช้ **PersistentVolumeClaim (PVC)** เพื่อขอ persistent storage ให้ Pod ที่ข้อมูลไม่หายเมื่อ Pod restart

## Files
- `pvc.yaml` — PersistentVolumeClaim ขอ storage 1Gi
- `pod.yaml` — Pod ที่ mount PVC และเขียนไฟล์ต่อเนื่อง

## ความสัมพันธ์ PV / PVC / StorageClass

```
StorageClass  →  สร้าง PV อัตโนมัติ (dynamic provisioning)
PVC           →  request storage จาก StorageClass
Pod           →  mount PVC เป็น volume
```

> **Note:** lab นี้ใช้ dynamic provisioning ผ่าน default StorageClass
> k3d, kind, minikube, และ k3s ทุกตัวมี default StorageClass พร้อมใช้งาน

## Apply

```bash
kubectl apply -f 06-storage-pv-pvc/
```

## Verify

```bash
# ดู PVC status (ควรเป็น Bound)
kubectl get pvc workshop-data-pvc -n workshop

# ดู PV ที่ถูกสร้างอัตโนมัติ
kubectl get pv

# ดู Pod
kubectl get pod workshop-storage -n workshop
kubectl logs workshop-storage -n workshop
```

## Test

```bash
# exec เข้า container แล้วดูไฟล์ที่เขียนลงใน PVC
kubectl exec -it workshop-storage -n workshop -- sh

# ภายใน container:
ls /data
cat /data/workshop.log
tail -f /data/workshop.log   # ดู real-time
exit
```

### ทดสอบ Persistence

```bash
# ลบ Pod (แต่ไม่ลบ PVC)
kubectl delete pod workshop-storage -n workshop

# สร้าง Pod ใหม่
kubectl apply -f 06-storage-pv-pvc/pod.yaml

# ข้อมูลเดิมยังอยู่ใน PVC
kubectl exec -it workshop-storage -n workshop -- cat /data/workshop.log
```

## Cleanup

```bash
kubectl delete -f 06-storage-pv-pvc/
# PVC ถูกลบ → PV อาจถูกลบตาม reclaimPolicy (default: Delete)
```

## Key Takeaways
- **PVC** คือ request ของ storage, **PV** คือ storage จริง
- Dynamic provisioning: StorageClass สร้าง PV ให้อัตโนมัติเมื่อมี PVC
- `accessModes: ReadWriteOnce` = mount ได้ 1 node พร้อมกัน
- ข้อมูลใน PVC ยังอยู่แม้ Pod ถูกลบ (ต้องลบ PVC เองเพื่อลบข้อมูล)
- StatefulSet ใช้ `volumeClaimTemplates` แทน PVC ที่กำหนดล่วงหน้า (ดู lab 07)
