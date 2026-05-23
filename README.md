# GoIT ArgoCD Demo

The project demonstrates a GitOps approach to deploying applications to Kubernetes (EKS) via ArgoCD. Changes in Git are automatically applied to the cluster.

## Repo structure
goit-argo/
├── namespaces/
│   ├── application/
│   │   ├── nginx.yaml      # Deployment + nginx service
│   │   └── ns.yaml         # Namespace: application
│   └── infra-tools/
│       └── ns.yaml         # Namespace: infra-tools
├── application.yaml        # ArgoCD Application manifesto
└── README.md

## Prerequisites

- AWS CLI configured with `terraform-admin` profile
- Terraform >= 1.0
- kubectl
- EKS cluster created via Terraform

## Infrastructure launch

### 1. Create EKS cluster

```bash
cd eks-vpc-cluster
terraform init
terraform apply
```

### 2. Connect kubectl to the cluster

```bash
aws eks --region eu-central-1 update-kubeconfig \
  --name eks-cluster \
  --profile terraform-admin
```

### 3. Deploy ArgoCD

```bash
cd terraform/argocd
terraform init
terraform apply
```

### 4. Check that ArgoCD has started

```bash
kubectl get pods -n infra-tools
```

There should be 7 pods with the `argocd-` prefix in the `Running` status.

## Access to the ArgoCD UI

### 1. Get password

```bash
kubectl -n infra-tools get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

### 2. Open port-forward

```bash
kubectl port-forward svc/argocd-server -n infra-tools 8080:80
```

### 3. Open browser

Перейти на `http://localhost:8080`

- Username: `admin`
- Password: from the previous step

## Application deployment via ArgoCD

### 1. Apply Application Manifest

```bash
kubectl apply -f application.yaml
```

### 2. Check status

```bash
kubectl get applications -n infra-tools
```

Expected result:
NAME    SYNC STATUS   HEALTH STATUS
nginx   Synced        Healthy

### 3. Check pods

```bash
kubectl get pods -n application
```

There should be 2 `demo-nginx` pods in `Running` status.

## Access to nginx

```bash
kubectl port-forward svc/demo-nginx 8080:80 -n application
```

Open browser: `http://localhost:8080`

## Infrastructure removal

Remove in reverse order:

```bash
# 1. Видалити ArgoCD
cd terraform/argocd
terraform destroy

# 2. Видалити EKS і VPC
cd eks-vpc-cluster
terraform destroy
```

## Link

- [Repo with application.yaml](https://github.com/Jozeph555/goit-argo)
- [Terraform Registry — ArgoCD Helm Chart](https://artifacthub.io/packages/helm/argo/argo-cd)