# DevOps & Infrastructure — Q&A

## Docker

### Q. What is Docker? How different from VMs?
**A:** Containerization platform that packages app + dependencies into isolated units. VMs virtualize hardware with a full guest OS. Docker virtualizes the OS kernel — containers are lightweight, fast-start, less resource-heavy. In LLM Cost Guard, I use Docker to spin up FastAPI + Redis + PostgreSQL instantly.

### Q. Walk through a Dockerfile.
**A:** Start with `python:3.10-slim` → `WORKDIR` → copy `requirements.txt` → `pip install` → copy source code → `EXPOSE 8000` → `CMD ["uvicorn", "main:app"]`.

### Follow-up: Why copy requirements.txt before source code?
**A:** Docker layer caching. Dependencies layer is cached. When code changes, only the code layer rebuilds — saves build time.

### Q. Multi-stage builds?
**A:** Use a builder stage to compile dependencies/C++ extensions, then copy only compiled artifacts into a lean production image. Leaves build tools behind. Massively reduces image size.

### Q. Docker Compose?
**A:** YAML-defined multi-container orchestration. Used in RAGOS (FastAPI + PostgreSQL + ChromaDB) and LLM Cost Guard (FastAPI + Redis + PostgreSQL + RabbitMQ). `docker-compose up` spins up entire architecture.

### Q. Debug a failing container?
**A:** `docker logs <id>` → `docker inspect` (env vars, entrypoint) → `docker exec -it <id> /bin/bash` (shell in, check files/network/scripts).

---

## Kubernetes

> **Bridge:** "I wrote K8s manifests (Deployments, Services) to run LLM Cost Guard on Minikube, ensuring it was cloud-native. I haven't managed a production K8s cluster at scale yet."

### Q. What is K8s?
**A:** Container orchestrator — manages thousands of containers. Handles rollouts, self-healing (restarting crashed containers), and scaling.

### Q. Pod vs Service vs Deployment?
- **Pod:** Smallest unit, usually one container.
- **Deployment:** Manages Pod replicas, handles rolling updates.
- **Service:** Stable network endpoint. Pods are ephemeral with changing IPs; Service gives static IP/DNS. (ClusterIP=internal, NodePort=node-level, LoadBalancer=external).

---

## CI/CD

### Q. What is CI/CD?
**A:** Automates build, test, deploy. Ensures main branch is always stable. At Cordum, all 26 integration tests had to pass via CI before my PR could merge.

### Q. GitHub Actions?
**A:** YAML in `.github/workflows/`. Triggers: `push`/`pull_request` to `main`. Runner: `ubuntu-latest`. Steps: checkout → setup Python → install deps → lint → pytest. RAGOS has `ci.yml`, LLM Cost Guard has `main.yml`.

---

## Git

### Q. Rebase vs merge?
**A:** `merge` creates a merge commit preserving full history (messy). `rebase` rewrites history onto target branch tip (clean, linear). Use rebase to clean PR history, merge to finalize into main.

### Q. Undo mistakes?
**A:** Not pushed: `git reset --soft` (keeps staging) or `--hard` (wipes). Already pushed: `git revert` (new undo commit). Lost commit: `git reflog` to find dangling hashes.

---

## Linux One-Liners

| Task | Command |
|------|---------|
| .py files modified in 24h | `find . -name "*.py" -mtime -1` |
| Count lines in all Python | `find . -name "*.py" \| xargs wc -l` |
| Process on port 8000 | `lsof -i :8000` |
| Tail logs live | `tail -f application.log` |
| Top 10 largest files | `du -ah . \| sort -rh \| head -n 10` |

### Q. File permissions?
**A:** Read(4) Write(2) Execute(1) for User/Group/Others. `chmod 755` = owner rwx, group+others rx.

---

## AWS

> **Bridge:** "I've used EC2 for hosting and S3 for storage within free tiers. I write cloud-native containerized code (Docker/FastAPI) so it deploys to any provider."

- **EC2:** Virtual servers. **S3:** Object storage. **RDS:** Managed databases. **IAM:** Users, roles, policies (never hardcode keys).

---

## OpenTelemetry

### Q. What is it?
**A:** Open-source framework for generating telemetry data — metrics, logs, traces. Vendor-neutral instrumentation.

### Q. Traces vs Metrics vs Logs?
- **Logs:** Discrete events ("User logged in at 10:00").
- **Metrics:** Aggregated numbers over time ("CPU at 85%", "RPS").
- **Traces:** Request lifecycle across services. In LLM Cost Guard, tracing shows exactly how much latency is Redis reservation vs LLM API call.
