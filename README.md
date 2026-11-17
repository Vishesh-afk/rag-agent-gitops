# RAG Agent GitOps

Kubernetes deployment files for the RAG Agent on local Minikube. Application code and the Jenkins pipeline stay in [RAG-agent](https://github.com/Vishesh-afk/RAG-agent).

## Layout

- `apps/rag-agent/overlays/minikube/`: app, database, and service manifests.
- `clusters/minikube/apps/`: Argo CD Application.

Jenkins publishes backend and frontend images to Docker Hub, then updates the SHA image tags in the Minikube overlay. Argo CD watches this repository and deploys into namespace `rag-agent`.

## One-time setup

Install Minikube and Argo CD. The backend image is private, so create a Docker Hub pull Secret named `dockerhub-auth` in namespace `rag-agent`.

From the application repository, create the namespace and runtime Secret from the local ignored `.env` file:

```sh
kubectl create namespace rag-agent --dry-run=client -o yaml | kubectl apply -f -
kubectl create secret generic rag-agent-runtime --from-env-file=.env --namespace rag-agent
```

The env file needs `DB_USER`, `DB_PASSWORD`, and `OPENAI_API_KEY`; model settings are optional. Secret values stay in the local cluster and must never be committed.

Apply `clusters/minikube/apps/rag-agent.yaml` once to register the Argo CD Application. Sync it from the Argo CD UI or CLI. Sync is manual by default.

The frontend is exposed as a NodePort. Run `minikube service frontend -n rag-agent --url` to access it. PostgreSQL uses a local 8 GiB PVC; this is not a backup.
