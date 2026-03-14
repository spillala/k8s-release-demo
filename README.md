# k8s-release-demo

A portfolio project for demonstrating a Go microservice deployed to `MicroK8s` through a `GitHub Actions` CI pipeline and an `Argo CD` GitOps workflow.

## What This Shows

- Go service development with tests
- Docker packaging
- Helm-based Kubernetes deployment
- Argo CD continuous delivery into MicroK8s
- A structure that feels close to real platform work, but is still finishable on one laptop

## Why This Is Freelance-Relevant

- shows a complete developer-to-cluster workflow instead of just app code
- demonstrates Kubernetes packaging, deployment, and operational endpoints
- mirrors the kind of internal platform work startups ask DevOps freelancers to handle
- gives you a concrete repo to use in proposals for Kubernetes, platform, and release engineering gigs

## Service Endpoints

- `GET /healthz`
- `GET /readyz`
- `GET /version`
- `GET /config`
- `POST /tasks/cache-warm`

## Local Run

```bash
make test
make build
make run
```

Then verify:

```bash
curl http://localhost:8080/healthz
curl http://localhost:8080/version
curl -X POST http://localhost:8080/tasks/cache-warm
```

## Local Kubernetes Flow

1. Install tooling on the remote Ubuntu laptop:

```bash
./scripts/bootstrap-ubuntu.sh
```

2. Re-login to refresh group membership, then verify `MicroK8s`:

```bash
newgrp microk8s
microk8s status --wait-ready
microk8s kubectl get nodes
```

3. Enable cluster add-ons and namespaces:

```bash
make bootstrap-local
```

4. Render Helm templates:

```bash
make helm-template
```

5. Deploy to the `dev` namespace:

```bash
make local-deploy IMAGE_TAG=dev
```

6. Access the service:

```bash
make port-forward
```

In another terminal:

```bash
curl http://localhost:8080/healthz
curl http://localhost:8080/readyz
curl http://localhost:8080/version
curl http://localhost:8080/config
curl -X POST http://localhost:8080/tasks/cache-warm
```

## Argo CD Flow

- Argo CD watches the Helm chart at `deploy/helm/release-api`
- The app manifest lives at `deploy/argocd/release-api-dev.yaml`
- The target namespace is `dev`
- CI validates the project on each push to `main`

## Architecture

```text
Developer change
  -> Go service build and tests
  -> Docker image build
  -> image import into MicroK8s
  -> Helm deploy into dev namespace
  -> Service exposure through ClusterIP + port-forward

Future state
  -> GitHub Actions CI
  -> registry publish
  -> Argo CD sync into MicroK8s
```

## Suggested Demo Evidence

- `microk8s kubectl -n dev get pods,svc,deploy`
- `curl http://localhost:8080/version`
- `curl -X POST http://localhost:8080/tasks/cache-warm`
- `microk8s kubectl -n dev logs deploy/release-api`

## Next Steps

- Push container images to `GHCR`
- Update Helm values automatically from CI after image publish
- Install Argo CD into MicroK8s and apply the application manifest
- Add screenshots and a short demo video before publishing this as a portfolio repo

For local laptop development, the `dev` environment uses a locally imported image with `imagePullPolicy: Never`. For GitHub-hosted CI/CD later, override `image.repository` with your remote registry image and switch the pull policy back to `IfNotPresent`.
