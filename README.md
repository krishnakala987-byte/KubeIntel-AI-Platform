# KubeIntel AI Platform

An AI-assisted platform for Kubernetes operations. Instead of digging through logs and `kubectl describe` output by hand, you hand it cluster state, logs, an incident description, or a Kubernetes manifest, and it uses a large language model (Groq's LLaMA 3.3 70B) to return root-cause analysis, incident triage, a deployment health score, security findings, and a ready-to-follow runbook. It runs on AWS EKS with a Flask API, a Streamlit UI, CI/CD through GitHub Actions, and Prometheus and Grafana for monitoring.

I built this to tie together two things I'd been learning separately: running real workloads on Kubernetes, and wiring a language model into a proper service instead of a notebook.

## What it does

You give it one of: current cluster state, pod logs, an incident, or a manifest. It gives back:

- Root-cause analysis for what's failing
- Incident triage with a severity and an ordered set of next steps
- A deployment health score
- Security recommendations
- Suggested remediation, including the `kubectl` commands to run
- A generated operational runbook

### The pieces in more detail

**Cluster state analysis** — spots common Kubernetes failures, diagnoses `CrashLoopBackOff` and `OOMKilled`, and suggests the `kubectl` commands to fix them, along with a root-cause explanation.

**Incident triage** — takes a production incident, classifies its severity, and lays out a prioritized, step-by-step investigation.

**Log anomaly detection** — scans logs for `CrashLoopBackOff`, `OOMKilled`, `ImagePullBackOff`, liveness- and readiness-probe failures, certificate problems, permission-denied errors, and back-off failures.

**Deployment health scoring** — scores a deployment on its resource limits, health probes, security context, replica strategy, and image policies.

**Security audit** — checks manifests for privileged containers, missing RBAC, hardcoded secrets, missing resource limits, and Pod Security Standards violations.

**Runbook generator** — produces an incident runbook: the resolution workflow, the investigation commands, and prevention notes.

## Architecture

```
Developer / SRE
      |
      v
Streamlit frontend  (kubeops-ui)
      |
      v
Flask API  (kubeops-api)
      |
      v
Groq LLaMA 3.3 70B

Infrastructure:
  GitHub Actions CI/CD
        |
        v
  AWS EKS cluster
    |-- kubeops-api
    |-- kubeops-ui
    |-- Prometheus
    |-- Grafana
    |-- Node Exporter
        |
        v
  AWS ECR  (image registry)
```

## Tech stack

| Category | Technology |
|---|---|
| Backend | Flask + Gunicorn |
| Frontend | Streamlit |
| AI model | Groq LLaMA 3.3 70B |
| Kubernetes | AWS EKS |
| Registry | AWS ECR |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus |
| Dashboards | Grafana |
| Containers | Docker |
| Language | Python 3.11 |

## Project structure

```
KubeIntel-AI-Platform/
|-- .github/          # GitHub Actions workflows
|-- docs/             # full project documentation
|-- kubernetes/       # Kubernetes manifests
|-- monitoring/       # Prometheus + Grafana config
|-- services/         # Flask API and Streamlit UI
|-- tests/            # pytest suite
|-- docker-compose.yml
|-- Makefile
`-- README.md
```

## Running it locally

Clone the repo and set up the environment file:

```bash
git clone https://github.com/krishnakala987-byte/KubeIntel-AI-Platform.git
cd KubeIntel-AI-Platform
cp .env.example .env
```

Fill in the two values it needs:

```env
GROQ_API_KEY=your_groq_api_key
AWS_ACCOUNT_ID=your_aws_account_id
```

Then bring it up:

```bash
docker-compose up --build
```

- UI: http://localhost:8501
- API: http://localhost:5000

## Deploying to AWS EKS

Create the cluster:

```bash
eksctl create cluster \
  --name kubeintel-cluster \
  --region us-east-1 \
  --nodegroup-name kubeintel-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --managed
```

Apply the manifests:

```bash
kubectl apply -f kubernetes/
```

## CI/CD

The GitHub Actions pipeline runs pytest, builds the Docker images, pushes them to AWS ECR, deploys to EKS, and then checks that the rollout is healthy.

## Kubernetes setup

On the cluster the deployments use a Horizontal Pod Autoscaler, liveness and readiness probes, rolling updates, pod anti-affinity, Kubernetes Secrets, and non-root containers.

## Monitoring

Prometheus collects Flask, Kubernetes, and node metrics. Grafana charts CPU, memory, cluster health, and per-pod metrics.

## API endpoints

| Method | Endpoint |
|---|---|
| GET | /health |
| GET | /ready |
| GET | /metrics |
| POST | /api/v1/analyze |
| POST | /api/v1/analyze/logs |
| POST | /api/v1/incident/triage |
| POST | /api/v1/incident/runbook |
| POST | /api/v1/deployment/score |
| POST | /api/v1/security/audit |

## More detail

The full architecture, deployment steps, observability setup, troubleshooting, API reference, and operational commands live in `docs/PROJECT_DOCUMENTATION.md`.

## Author

Krishna Kala
