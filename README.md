### Introduction

This repository exists as part of a test case. It is one of 3 similar repositories defined within this organization:
- [deployment-permutations-a](https://github.com/managed-gitops-test-data/deployment-permutations-a)
- [deployment-permutations-b](https://github.com/managed-gitops-test-data/deployment-permutations-b)
- [deployment-permutations-c](https://github.com/managed-gitops-test-data/deployment-permutations-c)

Each repository contains three branches:
- branchA
- branchB
- branchC

Each branch contains three folders, with 2 `ConfigMap` resources and a `kustomization.yaml` defined within each folder:
```
.
├── pathA
│   ├── config-map-field.yaml
│   ├── config-map-resource.yaml
│   └── kustomization.yaml
├── pathB
│   ├── config-map-field.yaml
│   ├── config-map-resource.yaml
│   └── kustomization.yaml
├── pathC
│   ├── config-map-field.yaml
│   ├── config-map-resource.yaml
│   └── kustomization.yaml
└── README.md
```

The `config-map-field.yaml` contains a simple `ConfigMap` resource, which has a `data` field describing which repository url/branch/path it is defined in. This can be useful for testing changes to the url/branch/path fields of a GitOpsDeployment or Argo CD Application.

The `config-map-resource.yaml` likewise contains a simple `ConfigMap` resource, but this one is named after the url/branch/path it exists in. This can be useful for testing whether resources have been pruned (deleted) when switching between paths/branches/repo urls in a GitOpsDeployment.


The `kustomization.yaml` is basic, and just points to the `ConfigMap` resources.

### `ConfigMap` resource contents

Every `config-map-field.yaml`, in every repository/branch/path, contains an identifier (under `.data.identifier`) which indicates the specific url/branch/path that is being deployed.

Every `config-map-indicator.yaml`, in every repository/branch/path, contains a `.metadata.name` field which indicates the specific url/branch/path that is being deployed.

For example:
```bash
# Repository A
git clone https://github.com/managed-gitops-test-data/deployment-permutations-a
cd deployment-permutations-a

# Branch B
git checkout branchB

# Path C
cd pathC
kustomize build .
```

will output:
```yaml
apiVersion: v1
data: {}
kind: ConfigMap
metadata:
  name: deployment-permutations-a-branchB-pathC
---
apiVersion: v1
data:
  identifier: (deployment-permutations-a)-(branchB)-(pathC)
kind: ConfigMap
metadata:
  name: identifier
```

The `ConfigMap` resources are otherwise equal to another other between permutations.

Test cases can then use the `identifier` or `name` value of the ConfigMaps to ensure that the correct repository URL/branch/path has been deployed.

### How to deploy one of these permutations using a `GitOpsDeployment` resource or Argo CD `Application` resource

Create a `GitOpsDeployment` resource or Argo CD `Application` resource with the following spec field:
```yaml
spec:
  source:
    repoURL: https://github.com/managed-gitops-test-data/deployment-permutations-(a/b/c)
    path: path(A/B/C)
    targetRevision: branch(A/B/C)
```

Example:
```yaml
apiVersion: managed-gitops.redhat.com/v1alpha1
kind: GitOpsDeployment
metadata:
  name: my-gitops-depl
spec:
  source:
    repoURL: https://github.com/managed-gitops-test-data/deployment-permutations-a
    path: pathC
    targetRevision: branchB
```

The `ConfigMap` resource that Argo CD deploys should then contain an `.data.identifier` or `.metadata.name` field with the value of `(deployment-permutations-a)-(branchB)-(pathC)`.

### How to manually deploy one of these permutations

If you want to test what one of the paths would deploy, you can run these commands:
```bash
git clone (repository url)
git checkout (branch)
cd (path)
# Set your target namespace, for example, kubens "my-namespace"
kustomize build . | kubectl apply -f -
```
