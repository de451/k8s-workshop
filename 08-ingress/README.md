# Lab 08 — Ingress

## Objective
เรียนรู้ **Ingress** ที่ทำหน้าที่เป็น HTTP router รับ traffic จากภายนอก cluster แล้วส่งไปยัง Service ที่ถูกต้อง

## Files
- `deployment.yaml` — Deployment ชื่อ `workshop-web`
- `service.yaml` — ClusterIP Service ชื่อ `workshop-web-svc`
- `ingress.yaml` — Ingress rule สำหรับ host `web.localhost`

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
kubectl get ingress workshop-ingress
kubectl describe ingress workshop-ingress

# ดู address ของ Ingress (อาจใช้เวลาสักครู่)
kubectl get ingress -w
```

## Test

`web.localhost` resolve ไปที่ `127.0.0.1` โดยอัตโนมัติบนทุก OS ไม่ต้องแก้ `/etc/hosts`

### เปิดผ่าน Browser

เปิด **http://web.localhost** (k3d) หรือ **http://web.localhost:8080** ถ้าใช้ port อื่น

### ทดสอบด้วย curl

```bash
curl http://web.localhost

# ถ้า curl ไม่ได้ (บาง OS ไม่ resolve .localhost อัตโนมัติ) ให้ใช้ --resolve แทน
curl --resolve web.localhost:80:127.0.0.1 http://web.localhost
```

### ทดสอบ Load Balancing ผ่าน Ingress

```bash
# รัน curl หลายครั้ง สังเกต Pod hostname ที่ต่างกัน
for i in $(seq 1 6); do
  curl -s --resolve web.localhost:80:127.0.0.1 http://web.localhost | grep "Pod:"
done
```

## Cleanup

```bash
kubectl delete -f 08-ingress/
```

## Key Takeaways
- Ingress ต้องมี **Ingress Controller** (nginx, traefik, etc.) ถึงจะทำงานได้
- Ingress ทำ L7 routing ตาม **host** และ **path**
- `*.localhost` resolve เป็น `127.0.0.1` อัตโนมัติ เหมาะสำหรับ local development
- หลาย Ingress rule ใช้ Ingress Controller เดียวกัน (ประหยัด LoadBalancer)
- `ingressClassName` ระบุว่าใช้ controller ไหน ถ้ามีหลายตัว
