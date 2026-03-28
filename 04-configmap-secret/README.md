# Lab 04 — ConfigMap & Secret

## Objective
เรียนรู้การแยก **configuration** ออกจาก container image ด้วย ConfigMap และการจัดการ **sensitive data** ด้วย Secret

## Files
- `configmap.yaml` — ConfigMap ที่มีทั้ง env vars และ HTML file
- `secret.yaml` — Secret ที่มี password และ API key
- `deployment.yaml` — Deployment ที่ consume ทั้ง ConfigMap และ Secret
- `service.yaml` — ClusterIP Service

## Apply

```bash
kubectl apply -f 04-configmap-secret/
```

## Verify

```bash
kubectl get configmap workshop-web-config -n workshop
kubectl get secret workshop-web-secret -n workshop
kubectl get pods -n workshop -l app=workshop-web
```

## Test

### ตรวจสอบ Environment Variables

```bash
# ดู env ทั้งหมดใน container
kubectl exec -it deploy/workshop-web -n workshop -- env | grep -E "APP_|DB_|API_"

# ผลที่ควรเห็น:
# APP_ENV=production
# APP_COLOR=blue
# DB_PASSWORD=s3cur3-p@ssw0rd
# API_KEY=workshop-api-key-12345
```

### ตรวจสอบ Mounted File

```bash
# ดู HTML file ที่ mount มาจาก ConfigMap
kubectl exec -it deploy/workshop-web -n workshop -- cat /usr/share/nginx/html/index.html

# ทดสอบ HTTP response
kubectl port-forward service/workshop-web-svc 8080:80 -n workshop
# เปิดอีก terminal: curl http://localhost:8080
```

### ดูค่าใน Secret (base64 encoded)

```bash
# ดู Secret (ค่าจะถูก encode เป็น base64)
kubectl get secret workshop-web-secret -n workshop -o yaml

# decode ค่า
kubectl get secret workshop-web-secret -n workshop \
  -o jsonpath='{.data.DB_PASSWORD}' | base64 -d
```

## Cleanup

```bash
kubectl delete -f 04-configmap-secret/
```

## Key Takeaways
- **ConfigMap** เก็บ config ที่ไม่ sensitive เช่น ENV, feature flags, config files
- **Secret** เก็บ sensitive data เช่น passwords, API keys (เก็บแบบ base64 ไม่ใช่ encrypted โดย default)
- มี 3 วิธีใช้: env var เดี่ยว, `envFrom` (ทั้ง ConfigMap), mount เป็น file
- `stringData` ใน Secret ช่วยให้เขียน plain text ได้โดยไม่ต้อง encode เอง
- ควรใช้ tool เช่น Sealed Secrets หรือ Vault สำหรับ production
