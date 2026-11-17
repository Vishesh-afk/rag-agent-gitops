# RAG Agent GitOps

Desired Kubernetes state for the RAG Agent on local Minikube. The application source and Jenkins pipeline remain in [RAG-agent](https://github.com/Vishesh-afk/RAG-agent).

## Layout

- `apps/rag-agent/overlays/minikube/`: Minikube workloads, PostgreSQL persistence, and AWS Secrets Manager sync.
- `clusters/minikube/apps/`: Argo CD Application entry point.

Jenkins publishes backend and frontend images to Docker Hub, then updates the image tags in the Minikube overlay. Argo CD watches this repository and reconciles the overlay into the `rag-agent` namespace.

## Prerequisites

1. Minikube with Argo CD and External Secrets Operator (ESO) installed.
2. AWS Secrets Manager secret `rag-agent/minikube/runtime` with JSON fields `DB_USER`, `DB_PASSWORD`, `OPENAI_API_KEY`, `LLM_CHOICE`, and `EMBEDDING_MODEL`.
3. The secret should be in AWS region `us-east-2`, matching the project's last known region; verify before syncing.
4. ESO bootstrap Secret `aws-secrets-manager-auth` created in namespace `rag-agent`, using a dedicated AWS identity that can read only the named secret.
5. Docker Hub pull Secret `dockerhub-auth` in namespace `rag-agent` because the backend image is private.

Do not commit credentials or Kubernetes Secret values. Minikube does not have an AWS IAM role by default, so the ESO bootstrap identity is a local-only credential. For EKS, use workload identity instead.

## Register the Argo CD Application

Apply `clusters/minikube/apps/rag-agent.yaml` once after Argo CD is installed. Sync the Application from the Argo CD UI or CLI. Sync is manual by default.

The frontend is exposed as a NodePort. Run `minikube service frontend -n rag-agent --url` to access it. PostgreSQL uses a local 8 GiB PVC; this is not a backup.
