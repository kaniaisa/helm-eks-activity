# Deploying Applications to Amazon EKS Using Helm

Individual hands-on activity completed on 26 October 2026 using AWS CloudShell from a Windows laptop.

## Environment

- AWS Region: us-east-1
- EKS Cluster: shared-eks-cluster
- Namespace: kaniaisa-helm-activity
- Helm: v3.22.1
- Bitnami Nginx chart: 25.2.12

All application deployments and release changes were scoped to my namespace.

## 1. Tool Setup

Verified AWS CLI and kubectl:

```bash
aws --version
kubectl version --client
```

Installed and verified Helm:

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 -o get_helm.sh
bash get_helm.sh
helm version
```

Connected to the class cluster and created my namespace:

```bash
aws eks update-kubeconfig --region us-east-1 --name shared-eks-cluster
kubectl create namespace kaniaisa-helm-activity
kubectl get namespace kaniaisa-helm-activity
```

## 2. Deploy Nginx Using Helm

Added the repository and searched for charts:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/nginx
```

Installed and verified Nginx:

```bash
helm install my-nginx bitnami/nginx --namespace kaniaisa-helm-activity
helm list -n kaniaisa-helm-activity
kubectl get all -n kaniaisa-helm-activity
kubectl get svc my-nginx -n kaniaisa-helm-activity
```

Result: Nginx was running and the LoadBalancer URL displayed the “Welcome to nginx!” page.

## 3. Customise the Helm Configuration

Downloaded the chart values:

```bash
helm show values bitnami/nginx --version 25.2.12 > nginx-values.yaml
```

Confirmed the service configuration:

```yaml
service:
  type: LoadBalancer
```

The default was already LoadBalancer, so no service type change was required.

Applied the values file:

```bash
helm upgrade my-nginx bitnami/nginx --version 25.2.12 -f nginx-values.yaml -n kaniaisa-helm-activity
kubectl get svc my-nginx -n kaniaisa-helm-activity
```

Result: Revision 2 showed “Upgrade complete”.

## 4. Manage Helm Releases

Viewed release history and rolled back to revision 1:

```bash
helm history my-nginx -n kaniaisa-helm-activity
helm rollback my-nginx 1 -n kaniaisa-helm-activity
helm history my-nginx -n kaniaisa-helm-activity
```

Result:

| Revision | Description |
|---|---|
| 1 | Install complete |
| 2 | Upgrade complete |
| 3 | Rollback to 1 |

Uninstalled the initial release:

```bash
helm uninstall my-nginx -n kaniaisa-helm-activity
```

Result: The release was successfully uninstalled.

## 5. Create and Deploy a Custom Helm Chart

Created the chart:

```bash
helm create my-app
cd my-app
```

Updated my-app/values.yaml:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 80
```

Returned to the parent directory and deployed:

```bash
cd ..
helm install my-app ./my-app -n kaniaisa-helm-activity
kubectl get all -n kaniaisa-helm-activity
```

Result: Three pods were running and the Deployment showed 3/3 ready replicas.

## 6. Deploy Metrics-Enabled Nginx

Confirmed that the ServiceMonitor resource type was available:

```bash
kubectl api-resources --api-group=monitoring.coreos.com
```

Enabled these settings in the existing nginx-values.yaml sections:

```yaml
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
```

Installed Nginx with metrics enabled:

```bash
helm install my-nginx bitnami/nginx --version 25.2.12 -f nginx-values.yaml -n kaniaisa-helm-activity
kubectl rollout status deployment/my-nginx -n kaniaisa-helm-activity --timeout=120s
```

Verified the resources:

```bash
kubectl get namespace kaniaisa-helm-activity
kubectl get pods,deployments,services,serviceaccounts,servicemonitors -n kaniaisa-helm-activity
```

## Final Results

| Resource | Verified result |
|---|---|
| Namespace | Active |
| my-app Deployment | 3/3 replicas ready |
| my-app pods | Three pods, each 1/1 Running |
| my-nginx Deployment | 1/1 replica ready |
| my-nginx pod | 2/2 containers ready |
| Application Services | LoadBalancer |
| my-nginx-metrics Service | ClusterIP, port 9113 |
| ServiceAccounts | my-app and my-nginx created |
| ServiceMonitor | my-nginx created |

ServiceMonitor creation and container readiness were verified. Prometheus scraping and dashboard integration were not tested in this activity.

The initial Nginx release was uninstalled in the release-management section, then installed again with metrics enabled. Its release revision therefore restarted at 1.

## Activity Files

The archive `helm-eks-activity-files.tar.gz` contains:

- `my-app/` — the complete custom Helm chart, including templates.
- `nginx-values.yaml` — the final metrics-enabled Bitnami Nginx configuration.

Extract the archive before using the complete chart:

```bash
tar -xzf helm-eks-activity-files.tar.gz
```

## Screenshot Evidence

Submission evidence includes:

- The activity namespace.
- Deployments, Services and ServiceAccounts within that namespace.
- The Nginx welcome page.
- Helm upgrade and rollback history.
- Final custom chart and metrics-enabled deployment results.
