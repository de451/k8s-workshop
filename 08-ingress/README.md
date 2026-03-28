# Lab 08 — Ingress

## Objective
เรียนรู้ **Ingress** ที่ทำหน้าที่เป็น HTTP router รับ traffic จากภายนอก cluster แล้วส่งไปยัง Service ที่ถูกต้อง

## Files
- `deployment.yaml` — Deployment ชื่อ `workshop-web`
- `service.yaml` — ClusterIP Service ชื่อ `workshop-web-svc`
- `ingress.yaml` — Ingress rule สำหรับ host `workshop.local`

## ความต่างจาก Service NodePort/LoadBalancer

| | NodePort | LoadBalancer | Ingress |
|---|---|---|---|
| Layer | L4 (TCP) | L4 (TCP) | L7 (HTTP) |
| Routing | port เท่านั้น | port เท่านั้น | host, path |
| Cost | — | cloud LB (เสียเงิน) | shared LB |
| TLS | manual | manual | ง่าย (cert-manager) |

> **Prerequisites:** cluster ต้องมี Ingress Controller
> - k3d: มีในตัว (Traefik หรือ nginx ขึ้นกับ version)
> - kind: ติดตั้งเพิ่ม
> - minikube: `minikube addons enable ingress`

## Apply

```bash
kubectl apply -f 08-ingress/
```

## Verify

```bash
kubectl get ingress workshop-ingress -n workshop
kubectl describe ingress workshop-ingress -n workshop

# ดู address ของ Ingress (อาจใช้เวลาสักครู่)
kubectl get ingress -n workshop -w
```

## Test

### วิธีที่ 1: แก้ /etc/hosts (แนะนำ)

```bash
# หา IP ของ Ingress Controller
# k3d: ใช้ 127.0.0.1 (ถ้าสร้าง cluster ด้วย --port flag)
# minikube: ใช้ $(minikube ip)
# kind: ใช้ localhost หรือ node IP

# เพิ่มบรรทัดนี้ใน /etc/hosts
echo "127.0.0.1 workshop.local" | sudo tee -a /etc/hosts

# ทดสอบ
curl http://workshop.local
# ถ้า k3d ใช้ port 8080: curl http://workshop.local:8080
```

### วิธีที่ 2: ใช้ curl ด้วย Host header (ไม่ต้องแก้ hosts)

```bash
# หา CLUSTER_IP ของ Ingress Controller
INGRESS_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}' 2>/dev/null || echo "127.0.0.1")

curl -H "Host: workshop.local" http://$INGRESS_IP

# k3d ทดสอบด้วย
curl -H "Host: workshop.local" http://localhost:8080
```

### ทดสอบ Load Balancing ผ่าน Ingress

```bash
# รัน curl หลายครั้ง สังเกต Pod hostname ที่ต่างกัน
for i in $(seq 1 6); do
  curl -s -H "Host: workshop.local" http://localhost:8080 | grep "Pod:"
done
```

## Cleanup

```bash
kubectl delete -f 08-ingress/
# ลบบรรทัด workshop.local ออกจาก /etc/hosts ถ้าเพิ่มไว้
```

## Key Takeaways
- Ingress ต้องมี **Ingress Controller** (nginx, traefik, etc.) ถึงจะทำงานได้
- Ingress ทำ L7 routing ตาม **host** และ **path**
- หลาย Ingress rule ใช้ Ingress Controller เดียวกัน (ประหยัด LoadBalancer)
- `ingressClassName` ระบุว่าใช้ controller ไหน ถ้ามีหลายตัว
