# Kargo helm demo

Demo repository for [Kargo](https://kargo.akuity.io/) using helm chart. 

Stuctured in the same way as the Kustomize [Kargo Demo](https://github.com/akuity/kargo-demo)

The following changes must be done to the configuration from the [Kargo Quickstart](https://kargo.akuity.io/quickstart/) guide

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kargo-helm-demo-test
  namespace: argocd
  annotations:
    kargo.akuity.io/authorized-stage: kargo-helm-demo:test
spec:
  project: default
  source:
    # Change Argo application to use helm with env specific value files 
    helm:
      valueFiles:
      - ../stages/test/values.yaml
    repoURL: ${GIT_REPO_URL}
    targetRevision: stage/test
    path: base
  destination:
    server: https://kubernetes.default.svc
    namespace: kargo-helm-demo-test
  syncPolicy:
    automated:
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Warehouse
metadata:
  name: kargo-helm-demo
  namespace: kargo-helm-demo
spec:
  subscriptions:
  - image:
      repoURL: nginx
      semverConstraint: ^1.24.0
  # Change warehouse to listen both nginx and the git repo
  - git:
      branch: master
      repoURL: ${GIT_REPO_URL}
```

```yaml
apiVersion: kargo.akuity.io/v1alpha1
kind: Stage
metadata:
  name: test
  namespace: kargo-helm-demo
spec:
  subscriptions:
    warehouse: kargo-helm-demo
  promotionMechanisms:
    gitRepoUpdates:
    - repoURL: ${GIT_REPO_URL}
      writeBranch: stage/test
      # Change the stages to update the helm chart values file instead
      helm:
        images:
        - image: nginx
          key: nginx.version
          value: Tag
          valuesFilePath: base/values.yaml
    argoCDAppUpdates:
    - appName: kargo-helm-demo-test
      appNamespace: argocd
```
