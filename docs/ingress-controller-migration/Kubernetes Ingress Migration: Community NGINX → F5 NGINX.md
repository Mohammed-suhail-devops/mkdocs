#   Kubernetes Ingress Migration: Community NGINX → F5 NGINX

## 📌 Overview

Migrated from the community NGINX Ingress Controller to F5 NGINX Ingress Controller to improve scalability, routing control, and maintainability.

---

## 🎯 Key Objectives

* Replace community ingress with an enterprise-grade solution
* Introduce structured ingress design (Master/Minion)
* Improve routing clarity and reduce conflicts

---

## 🏗️ Architecture Change

### Before

* Single ingress resource
* Mixed host + path configuration
* Limited flexibility

### After

* **Master Ingress**

  * Handles host-level configuration

* **Minion Ingress**

  * Handles path-based routing

---

## ⚙️ Installation (OCI Helm)

```bash
helm install nginx-ingress oci://ghcr.io/nginx/charts/nginx-ingress --version 2.5.1
```

Verify:

```bash
kubectl get ingressclasses
```

---

## 📄 Sample Configuration

### Master Ingress

```yaml
annotations:
  nginx.org/mergeable-ingress-type: "master"
```

### Minion Ingress

```yaml
annotations:
  nginx.org/mergeable-ingress-type: "minion"
  nginx.org/proxy-connect-timeout: "360s"
  nginx.org/proxy-read-timeout: "360s"
  nginx.org/proxy-send-timeout: "360s"
```

---

## ⚙️ F5 NGINX Key Annotations

### 🔹 Mergeable Ingress

```yaml
nginx.org/mergeable-ingress-type: "master"
nginx.org/mergeable-ingress-type: "minion"
```

---

### 🔹 Timeout Configuration

```yaml
nginx.org/proxy-connect-timeout: "360s"
nginx.org/proxy-read-timeout: "360s"
nginx.org/proxy-send-timeout: "360s"
```

---

### 🔹 Path Handling

```yaml
nginx.org/path-regex: "case_sensitive"
```

---

### 🔹 Request Size Limit

```yaml
nginx.org/client-max-body-size: "50m"
```

---

### 🔹 Buffer Configuration

```yaml
nginx.org/proxy-buffer-size: "16k"
nginx.org/proxy-buffers: "4 16k"
```

---

### 🔹 Rate Limiting

```yaml
nginx.org/limit-req-rate: "10r/s"
```

---

### 🔹 Load Balancing Method

```yaml
nginx.org/lb-method: "round_robin"
```

---

### 🔹 SSL Redirect Control

```yaml
nginx.org/ssl-redirect: "false"
```
---

## 🔄 Key Improvements

* Structured ingress design
* Better separation of concerns
* Improved routing control
* Easier debugging and maintenance

---

## ⚖️ Comparison

| Feature         | Community NGINX | F5 NGINX      |
| --------------- | --------------- | ------------- |
| Architecture    | Single          | Master/Minion |
| Flexibility     | Limited         | High          |
| Routing Control | Basic           | Advanced      |
| Maintainability | Medium          | High          |

---

## 🐞 Challenges Solved

* Fixed domain routing issues
* Resolved NGINX reload failures
* Eliminated wildcard path conflicts
* Standardized ingress configuration

---

## 🧪 Validation

```bash
kubectl get ingress
kubectl logs <nginx-pod>
curl http://app.example.com/api
```

---

## 📈 Outcome

* Production-ready ingress setup
* Reduced configuration complexity
* Scalable and maintainable architecture

---

## 🧠 Key Learnings

* Separate host and path responsibilities
* Avoid wildcard routing when possible
* Use correct annotation prefixes (`nginx.org`)
* Keep configurations clean and minimal

---

## ✅ Summary

This migration improved reliability and scalability by adopting a structured ingress model using F5 NGINX, making it suitable for enterprise Kubernetes environments.
