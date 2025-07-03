
The `applications/` directory holds the declarative YAML definitions (Argo CD `Application` resources) for each individual application that Argo CD will monitor and synchronize.

## How it Works (The GitOps Flow)

Argo CD is configured to **self-manage** by continuously observing this repository.

1.  **Argo CD Monitors This Repo:** A dedicated "bootstrap" Argo CD Application (applied directly to the cluster once) tells Argo CD to watch the `applications/` path within *this* repository.
2.  **Discover and Apply Application Definitions:** When an `Application` manifest is added, updated, or removed here (e.g., `vinted-tryout-app.yaml`), Argo CD automatically detects the change and applies the corresponding `Application` resource to itself.
3.  **Monitor Application Source Repositories:** Each `Application` resource defined here then instructs Argo CD to monitor its own specified `source` repository (e.g., `vinted-tryout-infra-demo` for `vinted-tryout-app`).
4.  **Synchronize to Cluster:** Any changes in those application source repositories (like new Docker image tags pushed by CI/CD) are automatically detected by Argo CD and synchronized to the target Kubernetes cluster.

This ensures that your entire application deployment landscape, including the definitions for Argo CD itself, is version-controlled and auditable in Git.

## Getting Started

To enable Argo CD to pick up the application configurations from this repository, a "bootstrap" Argo CD `Application` resource must be applied to your cluster **once**.

```bash
# This manifest tells Argo CD to manage its own configurations from this repository.
# Apply this directly to your cluster (e.g., via kubectl apply -f <filename>)

kubectl apply -n argocd -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd-self-manage-apps # A distinct name for this bootstrap application
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/mariusmatonis/argocd-config.git # URL of THIS repository
    targetRevision: HEAD # Or 'main'
    path: applications # The path where your application YAMLs are located within this repo

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd # Argo CD manages its own application resources in the argocd namespace

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
