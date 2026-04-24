* Master/Minion ingress setup (F5)
* Authentication using `auth_request.`
* Generic example headers (safe for public sharing)
* Clear explanation of flow

You can use this directly in your GitHub repo.

---

# F5 NGINX Ingress Authentication with Master/Minion Architecture

## 1. Overview

This document explains how authentication is implemented using the F5 NGINX Ingress Controller with the **mergeable (Master/Minion) ingress pattern**.

The solution uses native NGINX directives (`auth_request`) to enable flexible and scalable authentication for Kubernetes workloads.

---

## 2. Architecture

### Components

* **Master Ingress**

  * Defines host and TLS
* **Minion Ingress**

  * Defines paths and routing
* **Auth Service**

  * Validates requests and returns user identity headers

---

## 3. Authentication Flow

1. Client sends request to application endpoint
2. NGINX triggers internal subrequest using:

   ```
   auth_request /auth;
   ```
3. Request is forwarded to the authentication service
4. Auth service responds:

   * `200 OK` → request allowed
   * `401 Unauthorized` → redirected to login
5. Response headers from auth service are extracted
6. Headers are forwarded to backend service

---

## 4. Master Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-master-ingress
  annotations:
    nginx.org/mergeable-ingress-type: "master"
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
```

---

## 5. Minion Ingress with Authentication

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-minion-ingress
  annotations:
    nginx.org/mergeable-ingress-type: "minion"

    nginx.org/location-snippets: |
      auth_request /auth;

      error_page 401 = @error401;

      # Extract headers from auth response
      auth_request_set $test-user $upstream_http_x_test-user;
      auth_request_set $test_role $upstream_http_x_test_role;
      auth_request_set $test_email $upstream_http_x_test_email;

      # Pass headers to backend service
      proxy_set_header X-Test-Id $test-user;
      proxy_set_header X-Test-Role $test_role;
      proxy_set_header X-Test-Email $test_email;

      proxy_set_header Authorization $http_authorization;
      proxy_set_header Cookie $http_cookie;

spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
```

---

## 6. Internal Authentication Endpoint

You must define an internal location for auth:

```yaml
nginx.org/server-snippets: |
  location = /auth {
        internal;

        if ($request_method = OPTIONS) {
          return 200;
        }

        proxy_pass example.com/auth;
        proxy_method GET;

        proxy_pass_request_headers on;
        proxy_pass_request_body off;

        proxy_set_header Content-Length "";
        proxy_set_header Host example.com;
        proxy_set_header Authorization $http_authorization;
        proxy_set_header Cookie $http_cookie;

        proxy_set_header X-Original-URI $request_uri;
        proxy_set_header X-Original-Method $request_method;

        proxy_ssl_server_name on;
        proxy_ssl_name example.com;
      }
```

---

## 7. Example Headers from Auth Service

The authentication service should return headers like:

```
X-Test-Id: 12345
X-Test-Role: admin
X-Test-Email: user@example.com
```

These are then:

* Captured using `auth_request_set.`
* Forwarded to backend services

---

## 8. How Header Mapping Works

### Step 1: Extract from Auth Response

```
auth_request_set $test-user $upstream_http_x_test-user;
```

NGINX reads response header:

```
X-Test-Id
```

---

### Step 2: Store in Variable

```
$test-user
```

---

### Step 3: Pass to Backend

```
proxy_set_header X-Test-Id $test-user;
```

---

## 9. Swagger / UI Authentication (Optional Pattern)

For UI endpoints, use a separate auth path:

```nginx
auth_request /auth-ui;
```

👉 Recommended:

* Separate API and UI authentication
* Use different domains if needed

---

## 10. Key Advantages of This Approach

* Full control over authentication logic
* Custom header propagation
* Supports multiple authentication flows
* Avoids annotation limitations
* Scales well for enterprise use cases

---

## 11. Common Issues & Fixes

| Issue                | Cause                              | Fix                      |
| -------------------- | ---------------------------------- | ------------------------ |
| Auth loop            | Same upstream host                 | Add ssl sni block        |
| Missing headers      | Auth service not returning headers | Validate response        |
| 401 not redirecting  | error_page not configured          | Add redirect block       |
| NGINX reload failure | Invalid host/DNS                   | Fix service name         |

---

## 12. Debugging Tips

* Check NGINX logs:

  ```
  kubectl logs <nginx-pod>
  ```

* Test auth endpoint:

  ```
  curl http://auth-service/validate
  ```

* Validate headers:

  ```
  curl -I <endpoint>
  ```

* Exec into pod:

  ```
  kubectl exec -it <pod> -- sh
  ```

---

## 13. Best Practices

* Use **internal locations** for auth (`internal;`)
* Avoid sending request body to auth service
* Always forward original request context
* Use **separate auth endpoints** for different flows
* Keep header names consistent

---

## 14. Conclusion

Using F5 NGINX Ingress with `auth_request` and Master/Minion architecture provides a flexible and scalable authentication mechanism.

This approach enables:

* Fine-grained control over request handling
* Clean separation of routing and authentication
* Reliable and production-ready ingress design

---

# ✔️ Final Result

This version now clearly demonstrates:

* Real NGINX expertise
* Kubernetes ingress design skills
* Authentication architecture understanding

---

If you want next:

* I can create a **visual architecture diagram (very useful for interviews)**
* Or convert this into a **blog-style story for your portfolio**

