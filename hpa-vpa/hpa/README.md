# Metrics Server, VPA, and HPA Setup on Kind Cluster

This README documents the steps to install and configure **Metrics Server**, **Vertical Pod Autoscaler (VPA)**, and **Horizontal Pod Autoscaler (HPA)** on a Kubernetes (Kind) cluster, along with a load generator for testing.

---

## ✅ 1. Install Metrics Server (For Kind Cluster)

Apply the official Metrics Server manifest:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## ✏️ 2. Edit Metrics Server Deployment

Edit the Metrics Server deployment:

```bash
kubectl -n kube-system edit deployment metrics-server
```

Add the following flags under `containers.args`:

```yaml
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,ExternalIP
```

---

## 🔄 3. Restart Metrics Server

```bash
kubectl -n kube-system rollout restart deployment metrics-server
```

---

## ✔️ 4. Verify Metrics Server

Check if the Metrics Server pod is running:

```bash
kubectl get pods -n kube-system
```

Verify metrics collection:

```bash
kubectl top nodes
```

---

## 🚀 5. Install Vertical Pod Autoscaler (VPA)

Clone the Kubernetes Autoscaler repository:

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
```

Deploy VPA components:

```bash
./hack/vpa-up.sh
```

Verify VPA pods:

```bash
kubectl get pods -n kube-system
```

---

## ⚙️ 6. Deploy Application & HPA

Create and apply Kubernetes resources:

```bash
vim namespace.yaml
kubectl apply -f namespace.yaml

vim deployment.yaml
kubectl apply -f deployment.yaml

vim service.yaml
kubectl apply -f service.yaml

vim hpa.yaml
kubectl apply -f hpa.yaml
```

Check HPA status:

```bash
kubectl get hpa -n apache
```

---

## 🔌 7. Port Forward Service (Optional)

To access the Apache service locally:

```bash
sudo -E kubectl port-forward service/apache-service -n apache 82:80 --address 0.0.0.0
```

Access in browser:

```
http://localhost:82
```

---

## 🔥 8. Generate Load for Testing HPA

Run a temporary load generator pod:

```bash
kubectl run -i --tty load-generator --image=busybox -n apache /bin/sh
```

Inside the container, generate continuous traffic:

```bash
while true; do wget -q -O- http://apache-service.apache.svc.cluster.local; done
```

Monitor HPA behavior:

```bash
kubectl get hpa -n apache --watch
```

---

## 📌 Notes

* Metrics Server must be working before enabling HPA or VPA.
* Kind clusters often require `--kubelet-insecure-tls` for Metrics Server.
* VPA should be used carefully in production environments.

---

## Author

Prepared for Kubernetes learning and autoscaling experiments.
