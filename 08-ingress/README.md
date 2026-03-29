# Lab 08 — Ingress

## Objective
เรียนรู้ **Ingress** ที่ทำหน้าที่เป็น HTTP router รับ traffic จากภายนอก cluster แล้วส่งไปยัง Service ที่ถูกต้อง

## Files
- `deployment.yaml` — Deployment ชื่อ `workshop-web`
- `service.yaml` — ClusterIP Service ชื่อ `workshop-web-svc`
- `ingress.yaml` — Ingress rule สำหรับ host `my.de451.cloud`

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

`my.de451.cloud` ถูกตั้งค่าใช้ resolve ไปที่ `127.0.0.1` อยู่แล้ว เราจะใช้ประโยชน์จาก domain นี้

### เปิดผ่าน Browser

เปิด **http://my.de451.cloud**

### ทดสอบด้วย curl

```bash
curl http://my.de451.cloud
```

### ทดสอบ Load Balancing ผ่าน Ingress

```bash
for i in $(seq 1 6); do
  curl -s http://my.de451.cloud | grep "Pod:"
done
```

## Cleanup

```bash
kubectl delete -f 08-ingress/
```

## Key Takeaways
- Ingress ต้องมี **Ingress Controller** (nginx, traefik, etc.) ถึงจะทำงานได้
- Ingress ทำ L7 routing ตาม **host** และ **path**
- `*.localhost` ก็ resolve เป็น `127.0.0.1` อัตโนมัติ เหมาะสำหรับ local development ใช้ได้กับ browser แต่บาง client อาจใช้ไม่ได้ เช่น curl
- หลาย Ingress rule ใช้ Ingress Controller เดียวกัน (ประหยัด LoadBalancer)
- `ingressClassName` ระบุว่าใช้ controller ไหน ถ้ามีหลายตัว
