# argocd-app-sbandaog

This repository contains the Kubernetes manifests deployed by Argo CD for the **Inception of Things – Part 3** project.

The K3d cluster and Argo CD installation are managed from the main project repository:

> **Main repository:** https://github.com/s-t-e-v/inception-of-things

## Repository structure

```text
.
├── application.yaml
└── dev
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

* `application.yaml`: defines the Argo CD `Application` resource.
* `dev/deployment.yaml`: deploys the application container.
* `dev/service.yaml`: exposes the application inside the Kubernetes cluster.
* `dev/ingress.yaml`: exposes the application through the K3d load balancer.

## Application

The project uses the prebuilt application:

```text
wil42/playground
```

The application listens on port `8888` and provides two image versions:

```text
wil42/playground:v1
wil42/playground:v2
```

The currently deployed version is defined in:

```text
dev/deployment.yaml
```

## GitOps workflow

Argo CD monitors the `dev` directory of this repository.

When a manifest is updated and pushed to GitHub, Argo CD automatically synchronizes the changes with the Kubernetes cluster.

For example, the application version can be changed by updating the image tag:

```yaml
image: wil42/playground:v1
```

to:

```yaml
image: wil42/playground:v2
```

After committing and pushing the change, Argo CD detects it and updates the running application.

```bash
git add dev/deployment.yaml
git commit -m "feat: deploy application v2"
git push
```

The deployed version can then be checked with:

```bash
curl http://localhost:8888
```

Expected response for version 2:

```json
{
  "status": "ok",
  "message": "v2"
}
```

## Synchronization policy

The Argo CD application uses automated synchronization with:

* automatic deployment of Git changes;
* self-healing when the cluster state differs from Git;
* pruning of resources removed from the repository;
* automatic creation of the `dev` namespace.

The Git repository therefore represents the desired state of the application deployed in the cluster.
