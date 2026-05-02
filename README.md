# Capstone Project

## Project Goal:
We are building a secure internal web application (student/admin portal) using Docker, Nginx, HTTPS, CI/CD, and Kubernetes. It will run locally but be designed like a real production system.

1. AWS Architecture (Design Only)

2. Application (Frontend + Backend + DB)

- Frontend: HTML/CSS (or React if we want)
- Backend: Flask
- Database: MySQL
- Features:admin dashboard or student Portal.
- Add /health endpoint

3. Nginx Reverse Proxy + TLS

- Configure Nginx as reverse proxy
- Route traffic to app container
- Enable HTTPS using self-signed certificate
-  SSL/TLS 

4. Docker Setup

- Write Dockerfile for app
- Create Docker Compose to connect:
  - nginx
  - app
  - mysql
- Ensure containers communicate properly

5. Logging & Monitoring

- Capture:
  - Nginx access logs
  - App logs
- Show evidence (screenshots/log output)
- Add basic metrics or health checks.

6. CI/CD Pipeline

- Use GitHub Actions
- Build Docker image
- Push image to Docker Hub (or GHCR)

7. Kubernetes (Minikube)

- Deploy app using:
  - Pods / Deployments
  - Services
  - Ingress (acts like Nginx in K8s)





## Project Overview

This project builds a **secure internal web application** (Student Records Portal) running behind an Nginx reverse proxy with HTTPS, containerised with Docker, monitored with Prometheus metrics, and deployed to Kubernetes. It is designed to mirror a real production system architecture.

The project is based on the [`docker/awesome-compose` nginx-flask-mysql sample](https://github.com/docker/awesome-compose) and rebranded into a fully functional Student Records Portal.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML/CSS (served via Flask templates) |
| Backend | Python · Flask |
| Database | MySQL |
| Reverse Proxy | Nginx |
| TLS | Self-signed certificate (HTTPS local) |
| Containerisation | Docker · Docker Compose |
| Monitoring | Prometheus client (`/metrics`) · `/health` endpoint |
| CI/CD | GitHub Actions |
| Orchestration | Kubernetes (Minikube) |
| Cloud Design | AWS (EC2 · ALB · S3 · IAM · Security Groups) |

---
## Task One: # Architecture

![AWS Design Architecture: Nginx Reverse Proxy Diagram](./AWS-Architecture/aws-architecture.jpg)

## Architecture Overview

This project uses a secure AWS architecture with a reverse proxy and layered TLS encryption.

### Key Components

- Application Load Balancer (ALB)
- NGINX Reverse Proxy
- Kubernetes (Minikube)
- Amazon RDS (MySQL)
- Private VPC with public and private subnets

### Traffic Flow

User → ALB (TLS) → NGINX (TLS) → Frontend/Backend → RDS

🔎 **Detailed Architecture Explanation:**  
See [ARCHITECTURE.md](./AWS-Architecture/ARCHITECTURE.md)


## Task 2: What Was Modified

### Before — Original Sample App

The starting point was the `docker/awesome-compose` nginx-flask-mysql sample. It was a plain Flask hello-world app with no database interaction, no styling, and no monitoring.

![Original sample app before modification](screenshots/app-before.png)

---

### After — Student Records Portal

The app was rebranded and extended into a **Student Records Portal** with the following changes:

- Connected Flask to MySQL and seeds a `students` table on startup
- Homepage (`/`) displays student records (name + course) in styled cards
- Password read securely from a Docker secret at `/run/secrets/db-password`
- Added `/health` endpoint returning `{"status": "ok"}`
- Added `/metrics` endpoint with Prometheus request counter

![Student Records Portal after modification](screenshots/app-home.png)

---

**Key changes made to the app:**

- Connected to MySQL and creates a `students` table on startup with seeded records (Ada, John, Mary, David)
- Homepage (`/`) displays student name and course in styled cards
- Reads the database password securely from a Docker secret file at `/run/secrets/db-password`
- Added a **`/health`** endpoint returning `{"status": "ok"}` with HTTP 200
- Added a **`/metrics`** endpoint exposing Prometheus-compatible metrics (total HTTP request count)
- Integrated `prometheus_client` with a `REQUEST_COUNT` counter that increments on every request

```python
# Key endpoints added
@app.route('/health')
def health():
    return jsonify({"status": "ok"}), 200

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': 'text/plain'}
```
## Health & Metrics Endpoints

### /health

Returns a simple JSON health check — confirms the app is running:

![/health endpoint response](screenshots/health-endpoint.png)

### /metrics

Exposes Prometheus-compatible metrics including the `request_count` counter:

![/metrics endpoint Prometheus output](screenshots/metrics-endpoint.png)


## Project Structure

```
.
├── backend/
│   ├── hello.py              # Flask app — Student Records Portal
│   ├── requirements.txt      # Python dependencies
│   └── Dockerfile            # App dockerfile
├── proxy/
│   ├── conf            # Nginx reverse proxy config
│   └── Dockerfile/                # dockerfile for Nginx
├── docker-compose.yml   #docker compose
├── db        #database folder
|    ├── password.txt        
├── screenshots/              # Evidence screenshots
│   ├── app-home.png
│   ├── health-endpoint.png
│   ├── metrics-endpoint.png
│   ├── app-before.png
│   
└── README.md
```



---

## How to Run Locally

### Prerequisites
- Docker Desktop installed and running
- `docker compose` available

### Steps

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd <project-folder>

# 2. Build and start all containers
docker compose up --build

# 3. Access the app
# HTTP  → redirected to HTTPS
# HTTPS → https://localhost (accept the self-signed cert warning)

# 4. Test endpoints
curl -k https://localhost/health
curl -k https://localhost/metrics

# 5. View proxy logs
docker compose logs proxy

# 6. Stop everything
docker compose down
```

---

# TASK 3
## Reverse Proxy & TLS Implementation

This project implements a secure internal portal using *Nginx as a reverse proxy with TLS termination*, ensuring that all client traffic is encrypted and routed through a single controlled entry point.

###  Architecture Overview

Client → HTTPS → Nginx (Reverse Proxy) → Backend (Flask App) → Database (MariaDB)

* *Nginx* handles incoming requests and terminates TLS (HTTPS)
* *Flask backend* serves application logic internally
* *MariaDB* stores application data
* Backend and database are isolated within Docker networks

###  Reverse Proxy Configuration

Nginx is configured to act as a reverse proxy, forwarding all incoming requests to the backend service:

* All external traffic is routed through Nginx
* Backend service is not directly exposed to the host
* Internal communication occurs via Docker service networking (backend:8000)

###  TLS (HTTPS) Setup

HTTPS is enabled using a *self-signed SSL certificate* for local development and testing.

#### Key Features:

* TLS termination at the Nginx layer
* Support for modern protocols: *TLSv1.2 and TLSv1.3*
* Secure request handling between client and proxy

#### Certificate Configuration:


ssl_certificate:     /etc/nginx/certs/nginx.crt;
ssl_certificate_key: /etc/nginx/certs/nginx.key;

###  New Project Structure

```
.
├── backend/
│   ├── hello.py              # Flask app — Student Records Portal
│   ├── requirements.txt      # Python dependencies
│   └── Dockerfile            # App dockerfile
├── proxy/
│   ├── conf            # Nginx reverse proxy config
│   ├── cert/
          ├── nginx.cert          # Certificate
          ├── nginx.key            # Key
│   └── Dockerfile/                # dockerfile for Nginx
├── docker-compose.yml   #docker compose
├── db        #database folder
|    ├── password.txt        
├── screenshots/              # Evidence screenshots
│   ├── app-home.png
│   ├── health-endpoint.png
│   ├── metrics-endpoint.png
│   ├── app-before.png
│   
└── README.md
```



---
###  HTTP to HTTPS Redirection

All HTTP traffic is automatically redirected to HTTPS to enforce secure communication:

HTTP (port 80) → 301 Redirect → HTTPS (port 443)

This ensures that users cannot access the application over an insecure connection.


###  Security Enhancements

Additional security headers are configured in Nginx:

* X-Frame-Options → Prevents clickjacking
* X-Content-Type-Options → Prevents MIME sniffing
* X-XSS-Protection → Enables browser XSS filtering



###  Verification & Testing

The HTTPS setup was validated using the following methods:

#### ✔ Browser Verification

* Application accessible via: https://localhost
* Browser displays secure connection (self-signed warning expected)

![](screenshots/sca_capstone_TSL.png)

#### ✔ HTTP Redirect Test

bash
curl -I http://localhost


Expected:

HTTP/1.1 301 Moved Permanently
Location: https://localhost/
![](screenshots/reverse_terminal1.png)

#### ✔ HTTPS Response Test

bash
curl -k -I https://localhost

Expected:

HTTP/1.1 200 OK
![](screenshots/reverse_terminal2.png)

#### ✔ Nginx Logs

Requests are successfully routed through Nginx, confirmed via container logs:

bash
docker-compose logs proxy

![](screenshots/log.png)


#### ✔ Certificate Presence

bash
docker-compose exec proxy ls /etc/nginx/certs


###  Security Design Decisions

* Backend service is *not exposed publicly* (no direct port mapping)
* All access is controlled through Nginx
* TLS is terminated at the proxy layer, mimicking real-world production architecture
* Internal services communicate over isolated Docker networks


###  Outcome

This implementation ensures:

* Secure HTTPS communication
* Centralized traffic routing via reverse proxy
* Proper service isolation
* Production-aligned architecture design


# Task 4 — Logging & Monitoring

## Overview

This task adds logging and monitoring to the Student Records Portal, a Flask application served behind an Nginx reverse proxy with HTTPS. The monitoring stack captures Nginx access logs, Flask application logs, health check endpoints, and Prometheus metrics visualised in Grafana.

---

## Monitoring Stack

| Tool | Role |
|------|------|
| **Prometheus** | Scrapes and stores metrics from Flask and Nginx |
| **Grafana** | Visualises Prometheus metrics in dashboards |
| **nginx-prometheus-exporter** | Exposes Nginx connection metrics to Prometheus |
| **prometheus-flask-exporter** | Exposes per-route Flask metrics at `/metrics` |

---

## What Was Added

### 1. Flask App (`backend/hello.py`)
- Added Python `logging` module with timestamped `[INFO]` log output
- Integrated `prometheus_flask_exporter` to auto-track request counts and latency per route
- Added `/health` endpoint returning `{"status": "ok", "service": "student-portal"}`
- Added `/metrics` endpoint exposing Prometheus metrics

### 2. Nginx Config (`proxy/conf`)
- Added JSON-structured access logging format (`json_combined`) capturing:
  - Timestamp, remote IP, HTTP method, URI, status code, bytes sent, response time
- Added internal `stub_status` server on port `8080` for the nginx-prometheus-exporter
- Added `/metrics` proxy location to expose Flask metrics through Nginx

### 3. Docker Compose (`compose.yaml`)
Added three new services:

- **`prometheus`** — scrapes Flask and Nginx metrics every 15 seconds
- **`grafana`** — visualises metrics on port `3000`
- **`nginx-exporter`** — scrapes Nginx `stub_status` and exposes to Prometheus

### 4. Monitoring Config (`monitoring/`)
- `monitoring/prometheus.yml` — defines scrape targets for Flask and Nginx
- `monitoring/grafana/provisioning/datasources/prometheus.yml` — auto-connects Grafana to Prometheus on startup

---

## Project Structure

```
.
├── backend/
│   ├── Dockerfile
│   ├── hello.py               # Flask app with logging + metrics
│   └── requirements.txt
├── proxy/
│   ├── Dockerfile
│   ├── conf                   # Nginx config with JSON logs + stub_status
│   └── certs/                 # Self-signed TLS certificate
├── monitoring/
│   ├── prometheus.yml         # Prometheus scrape config
│   └── grafana/
│       └── provisioning/
│           └── datasources/
│               └── prometheus.yml  # Grafana datasource config
├── db/
│   └── password.txt
└── compose.yaml
```

---

## How to Run

```bash
docker compose down
docker compose up --build
```

---

## Endpoints

| Endpoint | Description |
|----------|-------------|
| `https://localhost` | Student Records Portal |
| `https://localhost/health` | Health check |
| `https://localhost/metrics` | Prometheus metrics |
| `http://localhost:9090` | Prometheus UI |
| `http://localhost:9090/targets` | Prometheus scrape targets |
| `http://localhost:3000` | Grafana (admin / admin) |

---

## Evidence

### Nginx Access Logs
Nginx logs every request in JSON format, captured via `docker compose logs proxy`:

```json
{"time":"2026-04-25T16:27:42+00:00","remote_addr":"192.168.32.1","method":"GET","uri":"/","status":200,"bytes_sent":1125,"request_time":0.101}
{"time":"2026-04-25T16:27:46+00:00","remote_addr":"192.168.32.1","method":"GET","uri":"/health","status":200,"bytes_sent":53,"request_time":0.003}
{"time":"2026-04-25T16:28:07+00:00","remote_addr":"192.168.32.1","method":"GET","uri":"/metrics","status":200,"bytes_sent":8165,"request_time":0.011}
```

### Flask App Logs
Flask logs every request and DB operation, captured via `docker compose logs backend`:

```
2026-04-25 16:27:42,279 [INFO] GET / - serving student records
2026-04-25 16:27:42,377 [INFO] Fetched 4 students from DB
2026-04-25 16:27:46,154 [INFO] GET /health - OK
```

### Health Check
```bash
curl -k https://localhost/health
```
```json
{"status": "ok", "service": "student-portal"}
```

### Metrics Endpoint
```bash
curl -k https://localhost/metrics
```
Key metrics exposed:
```
flask_http_request_total{method="GET",status="200"} 5.0
flask_http_request_duration_seconds{path="/",status="200"}
flask_http_request_duration_seconds{path="/health",status="200"}
app_info{version="1.0.0"} 1.0
```

### Prometheus Targets
Both scrape targets show **UP** at `http://localhost:9090/targets`:
- `flask-app` — scraping Flask `/metrics` every 15s
- `nginx` — scraping Nginx via nginx-exporter every 15s

### Useful Prometheus Queries
Run these in the Prometheus UI at `http://localhost:9090`:

```
flask_http_request_total
flask_http_request_duration_seconds_sum
nginx_connections_active
```

---

## Key Decisions

- **JSON access logs** were chosen over default Nginx logs because they are structured, easy to parse, and can be fed into log aggregation tools like Loki or CloudWatch in a production environment.
- **prometheus-flask-exporter** was used instead of manually tracking metrics with `prometheus_client` counters because it automatically instruments all routes with request counts and latency histograms.
- **stub_status** is exposed on an internal port (`8080`) only — not published to the host — so it is only accessible within the Docker network.


### Screenshots

#### Health Endpoint
![Health](screenshots/health-endpoint.png)

#### Metrics Endpoint
![Metrics](screenshots/metrics-endpoint.png)

#### Nginx Access Logs
![Nginx Logs](screenshots/nginx-access-logs.png)

#### Flask App Logs
![Flask Logs](screenshots/flask-app-logs.png)

#### Prometheus Targets
![Prometheus Targets](screenshots/prometheus-targets.png)

#### Prometheus Query
![Prometheus Query](screenshots/prometheus-query.png)

#### Grafana Dashboard
![Grafana](screenshots/grafana-metrics.png)





## Task 5 — CI/CD Pipeline

### Overview
This task implements a CI/CD pipeline using GitHub Actions that automatically builds the Docker image and pushes it to Docker Hub on every push to `main-branch`.

### What Was Added
- `.github/workflows/docker-build-push.yml` — GitHub Actions workflow file

### How It Works
1. Code is pushed to `dev-branch`
2. GitHub Actions triggers automatically
3. Builds the Docker image from `backend/Dockerfile`
4. Pushes the image to Docker Hub

### Docker Hub Image
🔗 **Image Link:** https://hub.docker.com/r/kehindemasuud/sca-capstone-app

![Docker Hub Image](screenshots/dockerhub-image.png)

### Evidence
![GitHub Actions Workflow](screenshots/cicd-workflow.png)

# Task 6 — Kubernetes Deployment (Minikube)

## Overview

The Student Records Portal is deployed to a local Kubernetes cluster (Minikube) using:

- **Secret** — database password, base64-encoded
- **Deployments** — MariaDB (1 replica) and Flask app (2 replicas)
- **Services** — ClusterIP for MariaDB, NodePort for the app
- **Ingress** — nginx-ingress controller with TLS termination using the project's existing self-signed certificate

The `proxy/` Nginx container from the Compose stack is **not** deployed in Kubernetes. Its role (TLS termination + reverse proxy) is performed by the nginx-ingress controller, which uses the same self-signed certificate from `proxy/certs/`.

---

## Architecture

```
                         ┌─────────────────────┐
                         │   Browser / curl    │
                         │ https://secure-...  │
                         └──────────┬──────────┘
                                    │ TLS
                         ┌──────────▼──────────┐
                         │  nginx-ingress      │
                         │  (TLS termination)  │
                         └──────────┬──────────┘
                                    │ HTTP
                         ┌──────────▼──────────┐
                         │ secure-reverse-proxy│
                         │   Service (NodePort)│
                         └──────────┬──────────┘
                                    │
                       ┌────────────┴────────────┐
                       │                         │
              ┌────────▼────────┐       ┌────────▼────────┐
              │  Flask Pod 1    │       │  Flask Pod 2    │
              └────────┬────────┘       └────────┬────────┘
                       │                         │
                       └────────────┬────────────┘
                                    │ TCP 3306
                         ┌──────────▼──────────┐
                         │   db Service        │
                         │   (ClusterIP)       │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   MariaDB Pod       │
                         └─────────────────────┘
```

---

## Prerequisites

- Docker
- Minikube
- kubectl
- Existing TLS cert and key in `proxy/certs/` (`nginx.crt`, `nginx.key`)
- Existing DB password in `db/password.txt`

---

## Manifest Files

All manifests are in the `k8s/` directory:

| File | Purpose |
|---|---|
| `db-secret.yaml` | Base64-encoded database password |
| `db-deployment.yaml` | MariaDB deployment, mounts the secret at `/run/secrets/db-password` |
| `db-service.yaml` | ClusterIP service exposing port 3306 |
| `deployment.yaml` | Flask app deployment, 2 replicas, mounts the same secret |
| `service.yaml` | NodePort service exposing port 80 → containerPort 8000 |
| `ingress.yaml` | Ingress with TLS, host `secure-proxy.local`, HTTP→HTTPS redirect |

---

## Deployment Steps

### 1. Start Minikube

```bash
minikube start --driver=docker
minikube status
```

![minikube start](./screenshots/minikube-start.png)
![minikube status](./screenshots/minikube-status.png)

### 2. Enable the Ingress Addon

```bash
minikube addons enable ingress
kubectl get pods -n ingress-nginx
```

Wait for the `ingress-nginx-controller-*` pod to show `1/1 Running`.

![addons list](./screenshots/minikube-list.png)

### 3. Generate the DB Secret

The base64-encoded password in `db-secret.yaml` is generated from `db/password.txt`:

```bash
echo -n "$(cat db/password.txt)" | base64
```

The result is pasted into the `data.db-password` field of `k8s/db-secret.yaml`.

### 4. Create the TLS Secret

The same self-signed certificate used by the Compose `proxy/` container is reused as a Kubernetes TLS secret:

```bash
kubectl create secret tls secure-proxy-tls \
  --cert=proxy/certs/nginx.crt \
  --key=proxy/certs/nginx.key
```
![create proxy](./screenshots/k8s-secure-proxy.png)

### 5. Apply Manifests

```bash
kubectl apply -f k8s/db-secret.yaml
kubectl apply -f k8s/db-deployment.yaml
kubectl apply -f k8s/db-service.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

![get apply](./screenshots/k8s-apply.png)
### 6. Verify Pods, Services, and Ingress

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

All three pods (1 db + 2 app replicas) should show `1/1 Running`. The ingress should show the Minikube IP in the ADDRESS column and `80, 443` in the PORTS column.

![get pods](./screenshots/k8s-get-pod.png)

![get svc](./screenshots/k8s-get-svc.png)

![get ingress](./screenshots/k8s-get-ingress.png)

### 7. Configure DNS

Add an entry to your hosts file so `secure-proxy.local` resolves locally.

**Linux / macOS / WSL:**
```bash
echo "$(minikube ip) secure-proxy.local" | sudo tee -a /etc/hosts
```

![get host](./screenshots/k8-host.png)

**Windows** (for browser access from Windows when running Minikube in WSL2):
Edit `C:\Windows\System32\drivers\etc\hosts` (as administrator) and add:
```
127.0.0.1 secure-proxy.local
```

### 8. (WSL2 only) Start Minikube Tunnel

When using the docker driver on WSL2, Minikube IP routing requires a tunnel. In a separate terminal:

```bash
minikube tunnel
```

Leave this terminal running.

---

## Verification

### HTTPS Health Check via Ingress

```bash
curl -k https://secure-proxy.local/health
```
```json
{"status": "ok"}
```

![health check](./screenshots/k8s-health.png)

### HTTP → HTTPS Redirect

```bash
curl -I http://secure-proxy.local/
```
Returns `308 Permanent Redirect` to HTTPS.

### Application via Browser

```bash
minikube service secure-reverse-proxy
```

Or open `https://secure-proxy.local` directly in the browser (click through the self-signed cert warning).

![app in browser](./screenshots/k8s-app-browser.png)

### Application Logs

```bash
kubectl logs -l app=secure-reverse-proxy --tail=20
```



![app logs](./screenshots/k8s-app-logs.png)

### Cluster Events

```bash
kubectl get events --sort-by='.lastTimestamp' | tail -20
```

![events](./screenshots/k8s-events.png)

---

## Monitoring

The full Prometheus + Grafana monitoring stack was implemented in **Task 4 — Logging & Monitoring** as part of the Docker Compose deployment. See Task4 for the complete setup, scrape configs, and dashboards.

In the Kubernetes deployment, the application's `/metrics` endpoint remains exposed via the ingress and is ready to be scraped by an in-cluster Prometheus:

```bash
curl -k https://secure-proxy.local/metrics | head -20
```


![metrics endpoint](./screenshots/k8s-metrics.png)

---

## Key Decisions

- **No `proxy/` container in Kubernetes.** The nginx-ingress controller takes over TLS termination and reverse proxying. The same self-signed cert from `proxy/certs/` is reused as a Kubernetes TLS secret, so the certificate identity is consistent across both deployment modes.
- **Secret mount with `subPath`.** The DB password is mounted at `/run/secrets/db-password` using `subPath` rather than as a directory mount. This avoids the conflict with Kubernetes' service account token, which is also injected under `/var/run/secrets`.
- **NodePort service for the app.** ClusterIP would not be reachable via `minikube service` for browser access. NodePort allows both ingress-routed traffic (the production path) and direct service access for screenshots and debugging.
- **Same DB password file as Compose.** The base64 in the secret is generated from `db/password.txt`, so credentials stay consistent between Compose and Kubernetes deployments.

---

## Cleanup

```bash
kubectl delete -f k8s/
kubectl delete secret secure-proxy-tls
minikube stop
minikube delete
```
![clean up](./screenshots/k8s-clean-up.png)

To remove the hosts entry:
```bash
sudo sed -i '/secure-proxy.local/d' /etc/hosts
```

---

## Screenshots

All screenshots are in the `screenshots/` directory.



# References

- [docker/awesome-compose nginx-flask-mysql](https://github.com/docker/awesome-compose/tree/master/nginx-flask-mysql)
- [gh640/docker-compose-depends_on-nginx-certs-sample](https://github.com/gh640/docker-compose-depends_on-nginx-certs-sample)
- [Prometheus Python Client](https://github.com/prometheus/client_python)
- [Flask Documentation](https://flask.palletsprojects.com/)
