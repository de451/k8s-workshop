# Lab 00 — Namespace

## Objective
สร้าง namespace `workshop` ที่จะใช้เป็น boundary ของทุก lab ใน workshop นี้

## Files
- `namespace.yaml` — สร้าง namespace ชื่อ `workshop`

## Apply

> **ต้อง apply lab นี้ก่อนทุก lab เสมอ**

```bash
kubectl apply -f 00-namespace/
```

## Verify

```bash
kubectl get namespace workshop
kubectl describe namespace workshop
```

## Cleanup

```bash
kubectl delete -f 00-namespace/
# หมายเหตุ: การลบ namespace จะลบ resource ทุกตัวใน namespace ด้วย
```

## Key Takeaways
- Namespace ใช้แบ่งแยก workload ออกจากกัน
- ทุก resource ใน workshop นี้จะอยู่ใน namespace `workshop`
- การลบ namespace จะลบ resource ทั้งหมดที่อยู่ใน namespace นั้นด้วย
