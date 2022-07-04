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
│   ├── config-map.yaml
│   └── kustomization.yaml
├── pathB
│   ├── config-map.yaml
│   └── kustomization.yaml
├── pathC
│   ├── config-map.yaml
│   └── kustomization.yaml
└── README.md
```
The `config-map.yaml` contains a simple `ConfigMap` resource, which describes which repository url/branch/path it is defined in. This can be useful for testing changes to the url/branch/path fields of a GitOpsDeployment or Argo CD Application.

The `kustomization.yaml` is basic, and just points to the `ConfigMap` resource.

### `ConfigMap` resource contents

Every `config-map.yaml`, in every repository/branch/path, contains an identifier (under `.data.identifier`) which indicates the specific url/branch/path that is being deployed.

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
kind: ConfigMap
metadata:
  name: identifier
data:
  identifier: "(deployment-permutations-a)-(branchB)-(pathC)"
```

The `ConfigMap` resources are otherwise equal (same name, nil namespace) to another other between permutations.

Test cases can then use the `identifier` value of the ConfigMap to ensure that the correct repository URL/branch/path has been deployed.

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

The `ConfigMap` resource that Argo CD deploys should then contain an `.data.identifier` field with the value of `(deployment-permutations-a)-(branchB)-(pathC)`.

### How to manually deploy one of these permutations

If you want to test what one of the paths would deploy, you can run these commands:
```bash
git clone (repository url)
git checkout (branch)
cd (path)
# Set your target namespace, for example, kubens "my-namespace"
kustomize build . | kubectl apply -f -
```
