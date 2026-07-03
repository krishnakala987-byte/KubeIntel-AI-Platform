# KubeIntel

An AI-assisted operations tool for Kubernetes clusters on AWS EKS. It reads live cluster state and logs and turns them into something useful: incident triage, root-cause guesses, and auto-generated runbooks.

## What it does

- Exposes 9 REST endpoints (Flask + Gunicorn) with a Streamlit front end.
- Detects log anomalies across 8 common failure signatures, including CrashLoopBackOff, OOMKilled, ImagePullBackOff, probe failures, and certificate and permission errors.
- Scores deployment health across 5 dimensions.
- Audits manifests against 5 Pod Security checks.
- Uses a Groq-hosted LLaMA 3.3 70B model to turn raw cluster state and logs into readable root-cause analysis and runbooks.

## How it runs

- Containerized with Docker and deployed to AWS EKS, with images pushed to ECR.
- A 5-stage GitHub Actions pipeline: pytest-gated tests, build, push to ECR, deploy to EKS, and verify rollout health.
- Hardened with OIDC/IRSA least-privilege IAM, Kubernetes Secrets, pod anti-affinity, non-root containers, and liveness/readiness probes.
- Scaled and watched with an HPA and a Prometheus, Grafana, and Node Exporter stack tracking CPU, memory, pod health, and API metrics.

## Stack

Python, Flask, Gunicorn, Streamlit, AWS EKS, ECR, Docker, GitHub Actions, Prometheus, Grafana, Groq LLaMA 3.3 70B.
