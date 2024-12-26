# E-commerce Multi-cluster Kubernetes Configuration

## Overview
Repositori ini berisi konfigurasi Kubernetes lengkap untuk platform e-commerce global dengan arsitektur multi-kluster. Sistem ini dirancang untuk menangani skenario lalu lintas tinggi dengan fokus pada skalabilitas, keamanan, dan konsistensi data di berbagai wilayah geografis (AS, Eropa, Asia).

## Features
- Penerapan multi-kluster di tiga wilayah
- Arsitektur mikroservis dengan layanan terkontainerisasi
- Penskalaan otomatis menggunakan metrik khusus
- Caching terdistribusi dengan Redis Enterprise
- Enkripsi end-to-end menggunakan sidecar container
- Pemantauan dan observabilitas komprehensif
- Implementasi service mesh dengan Istio
- Kebijakan jaringan untuk meningkatkan keamanan

## Prerequisites
- Kubernetes 1.24+
- kubectl
- Kustomize
- Helm v3
- Istio 1.18+
- Redis Enterprise Operator
- Prometheus Operator
- Grafana
- Jaeger

### Core Services
1. **Layanan Katalog Produk**  
   - Mengelola informasi produk  
   - Mengimplementasikan caching untuk kinerja tinggi  
   - Menangani pencarian dan penyaringan produk  

2. **Layanan Pembayaran**  
   - Memproses pembayaran secara aman  
   - Mengimplementasikan mekanisme pengulangan (retry)  
   - Menyediakan riwayat transaksi  

3. **Layanan Rekomendasi**  
   - Menghasilkan rekomendasi yang dipersonalisasi  
   - Menggunakan caching terdistribusi  
   - Mengimplementasikan model ML untuk pelayanan  

4. **Layanan Autentikasi**  
   - Menangani autentikasi pengguna  
   - Mengelola token sesi  
   - Mengimplementasikan OAuth2/OIDC

### Infrastructure Components
1. **Service Mesh (Istio)**  
   - Manajemen lalu lintas  
   - Keamanan (mTLS)  
   - Observabilitas  

2. **Stack Pemantauan**  
   - Prometheus untuk pengumpulan metrik  
   - Grafana untuk visualisasi  
   - Jaeger untuk pelacakan terdistribusi  
   - Dasbor khusus untuk metrik bisnis  

3. **Lapisan Caching**  
   - Kluster Redis Enterprise  
   - Replikasi multi-master  
   - Sinkronisasi lintas wilayah  

## Directory Structure
```
kubernetes-ecommerce/
├── README.md
├── clusters/                 # Region-specific configurations
│   ├── asia/
│   ├── eu/
│   └── us/
├── base/                    # Base configurations
│   ├── deployments/
│   ├── config/
│   ├── networking/
│   ├── scaling/
│   └── services/
└── overlays/               # Environment-specific configs
    ├── development/
    ├── staging/
    └── production/
```

## Installation Guide

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/kubernetes-ecommerce.git
cd kubernetes-ecommerce
```

### 2. Install Prerequisites
```bash
# Install Istio
istioctl install --set profile=demo

# Install Redis Operator
helm repo add redis-enterprise https://redis-enterprise.github.io/redis-enterprise-k8s-docs
helm install redis-operator redis-enterprise/redis-enterprise-operator

# Install Prometheus Operator
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

### 3. Deploy Base Infrastructure
```bash
# Create namespaces
kubectl apply -f base/namespaces/

# Deploy Redis Enterprise Cluster
kubectl apply -f base/config/redis-config.yaml

# Deploy Monitoring Stack
kubectl apply -f base/config/prometheus-config.yaml
kubectl apply -f base/config/grafana-dashboard.yaml
```

### 4. Deploy Services
```bash
# Deploy to production environment
kubectl apply -k overlays/production/

# Deploy to specific region
kubectl apply -k clusters/us/
```

## Configuration Guide

### Scaling Configuration
Edit `base/scaling/hpa-config.yaml` to modify scaling parameters:
```yaml
spec:
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Security Configuration
1. Network Policies in `base/networking/network-policies.yaml`
2. mTLS Configuration in `base/networking/service-mesh/peer-authentication.yaml`
3. Encryption settings in deployment sidecars

## Monitoring and Maintenance

### Accessing Dashboards
```bash
# Grafana
kubectl port-forward svc/grafana 3000:3000 -n monitoring

# Prometheus
kubectl port-forward svc/prometheus 9090:9090 -n monitoring

# Jaeger
kubectl port-forward svc/jaeger 16686:16686 -n monitoring
```

### Common Operations
1. Scale deployments:
```bash
kubectl scale deployment product-catalog --replicas=5 -n ecommerce
```

2. Check HPA status:
```bash
kubectl get hpa -n ecommerce
```

3. View logs:
```bash
kubectl logs -f deployment/product-catalog -n ecommerce
```

## Troubleshooting

### Common Issues and Solutions

1. Pod Scaling Issues
```bash
# Check HPA events
kubectl describe hpa [hpa-name] -n ecommerce

# Check metrics-server
kubectl get apiservice v1beta1.metrics.k8s.io
```

2. Service Mesh Issues
```bash
# Analyze Istio configuration
istioctl analyze

# Check proxy status
istioctl proxy-status
```

3. Monitoring Issues
```bash
# Check Prometheus status
kubectl logs -n monitoring deployment/prometheus

# Verify ServiceMonitor
kubectl get servicemonitor -n monitoring
```