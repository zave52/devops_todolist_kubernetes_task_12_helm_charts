# Validation Instructions for DevOps ToDo List Kubernetes Task

This document provides step-by-step instructions to validate the implementation of the DevOps ToDo List application deployed on Kubernetes using Helm charts.

## Prerequisites

Before starting validation, ensure you have the following tools installed:

- Docker
- Kind (Kubernetes in Docker)
- kubectl
- Helm 3.x

## Validation Steps

### 1. Apply Cluster Configuration and Deploy Application

```bash
# Run the bootstrap script to create Kind cluster and deploy the application
chmod +x bootstrap.sh
./bootstrap.sh
```

**Expected Results:**
- Kind cluster should be created successfully with 2 nodes (1 control-plane, 1 worker)
- Worker node should be tainted with `app=mysql:NoSchedule`
- Ingress controller should be deployed and ready
- Metrics server should be installed
- TodoApp helm chart should be deployed with mysql sub-chart

### 2. Verify Cluster and Node Configuration

```bash
# Check cluster status
kubectl cluster-info

# Verify nodes are created with correct labels
kubectl get nodes --show-labels

# Verify node taints (mysql nodes should have app=mysql:NoSchedule taint)
kubectl describe nodes | grep -A 5 -B 5 "Taints"

# Check that control-plane node has ingress-ready=true label
kubectl get nodes -l ingress-ready=true
```

**Expected Results:**
- 2 nodes should be present (1 control-plane, 1 worker)
- Worker node should have `app=mysql` label
- Worker node should have `app=mysql:NoSchedule` taint

### 3. Verify Helm Chart Structure and Dependencies

```bash
# Check helm chart structure
ls -la helm-chart/todoapp/
ls -la helm-chart/todoapp/charts/mysql/

# Verify chart dependencies
helm dependency list helm-chart/todoapp/

# Validate chart templates without installing
helm template todoapp-release helm-chart/todoapp/ --dry-run
```

**Expected Results:**
- TodoApp chart should exist with mysql sub-chart in charts/ directory
- Dependencies should show mysql chart as a dependency
- Template validation should pass without errors

### 4. Verify Deployment Status

```bash
# Check helm release status
helm list -A

# Verify all pods are running
kubectl get pods -A

# Check services
kubectl get svc -A

# Verify persistent volumes and claims
kubectl get pv,pvc -A

# Check secrets and configmaps
kubectl get secrets,cm -A
```

**Expected Results:**
- Helm release `todoapp-release` should be deployed successfully
- All pods should be in `Running` state
- Services should be accessible
- PVs and PVCs should be bound
- Secrets and ConfigMaps should be created

### 5. Validate Namespace Configuration

```bash
# Check if namespaces are created from values.yaml
kubectl get namespaces

# Verify todoapp namespace
kubectl get all -n todoapp

# Verify mysql namespace
kubectl get all -n mysql
```

**Expected Results:**
- `todoapp` namespace should exist with todoapp resources
- `mysql` namespace should exist with mysql StatefulSet

### 6. Verify Resource Naming Convention

```bash
# Check that all resources use Chart.Name as prefix
kubectl get all,cm,secret,pv,pvc -n todoapp -o name | grep todoapp
kubectl get all,cm,secret,pv,pvc -n mysql -o name | grep mysql
```

**Expected Results:**
- All todoapp resources should have `todoapp` prefix
- All mysql resources should have `mysql` prefix

### 7. Validate Secrets Configuration

```bash
# Check todoapp secrets
kubectl get secret todoapp-secret -n todoapp -o yaml

# Check mysql secrets
kubectl get secret mysql-secret -n mysql -o yaml

# Verify secrets are populated using range function (check templates)
cat helm-chart/todoapp/templates/secret.yml
cat helm-chart/todoapp/charts/mysql/templates/secret.yml
```

**Expected Results:**
- Secrets should be created with base64 encoded values
- Secret templates should use range function to populate data

### 8. Verify Environment Variables Mapping

```bash
# Check deployment environment variables
kubectl describe deployment todoapp-deployment -n todoapp | grep -A 20 "Environment"

# Verify environment variables are mapped from secrets using range
cat helm-chart/todoapp/templates/deployment.yml | grep -A 10 "env:"
```

**Expected Results:**
- Environment variables should be mapped from secrets
- Deployment template should use range function for environment variables

### 9. Validate Resource Limits and Requests

```bash
# Check todoapp pod resource configuration
kubectl describe pod -l app=todoapp -n todoapp | grep -A 10 "Requests\|Limits"

# Check mysql pod resource configuration
kubectl describe pod -l app=mysql -n mysql | grep -A 10 "Requests\|Limits"
```

**Expected Results:**
- Pods should have resource requests and limits defined
- Values should match those specified in values.yaml files

### 10. Verify Rolling Update Configuration

```bash
# Check deployment rolling update strategy
kubectl describe deployment todoapp-deployment -n todoapp | grep -A 5 "RollingUpdateDeployment"
```

**Expected Results:**
- Rolling update strategy should be configured
- maxSurge and maxUnavailable should match values.yaml

### 11. Validate Node Affinity and Tolerations

```bash
# Check todoapp node affinity
kubectl describe deployment todoapp-deployment -n todoapp | grep -A 10 "Node-Selectors\|Affinity"

# Check mysql tolerations and affinity
kubectl describe statefulset mysql -n mysql | grep -A 15 "Tolerations\|Affinity"
```

**Expected Results:**
- TodoApp should have node affinity configured
- MySQL should have tolerations for `app=mysql:NoSchedule` taint
- MySQL should have node affinity to prefer nodes with `app=mysql` label

### 12. Verify HPA (Horizontal Pod Autoscaler)

```bash
# Check HPA configuration
kubectl get hpa -n todoapp
kubectl describe hpa todoapp-hpa -n todoapp
```

**Expected Results:**
- HPA should be created with min/max replicas from values.yaml
- CPU and memory utilization targets should match values.yaml

### 13. Validate RBAC Configuration

```bash
# Check service account
kubectl get serviceaccount -n todoapp

# Check role and rolebinding
kubectl get role,rolebinding -n todoapp

# Verify RBAC is configured in deployment
kubectl describe deployment todoapp-deployment -n todoapp | grep "Service Account"
```

**Expected Results:**
- Service account should be created with name from values.yaml
- Role and RoleBinding should exist
- Deployment should use the specified service account

### 14. Test Application Functionality

```bash
# Port forward to access the application (if needed)
kubectl port-forward svc/todoapp-service 8080:80 -n todoapp &

# Test application endpoints
curl http://localhost:8080/api/health
curl http://localhost:8080/api/ready

# Stop port forward
kill %1
```

**Expected Results:**
- Health check should return successful response
- Ready check should return successful response

### 15. Verify Ingress Configuration

```bash
# Check ingress resources
kubectl get ingress -A

# Verify ingress controller is running
kubectl get pods -n ingress-nginx
```

**Expected Results:**
- Ingress resources should be created
- Ingress controller pods should be running

### 16. Final Validation - Complete Resource List

```bash
# Generate complete output as specified in task
kubectl get all,cm,secret,ing -A > output.log

# Verify output.log file is created
cat output.log
```

**Expected Results:**
- output.log should contain all resources across all namespaces
- File should show running state of all components
