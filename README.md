# Kubernetes Deployment Guide
## 1. Comparing Application Logs Before and After Service Exposure

![Image](https://github.com/user-attachments/assets/f13f75e8-10fa-4848-a6ab-fd94e05c858a)

### Before Service Exposure
Before exposing the application as a Service, the logs only showed startup messages such as:
- "Started HTTP server on port 8080"
- "Started UDP server on port 8081"

This indicated that the container was running but not receiving any external traffic.

### After Service Exposure
After using `kubectl expose` to expose the app as a LoadBalancer Service and accessing it through `minikube service hello-node`, the logs started to include additional entries like `GET /`. These repeated log entries confirmed that:
- Each browser access generated a new HTTP request
- The container successfully received and processed external traffic
- The Service was working as intended

This demonstrated that exposing the app as a Service successfully made the Pod accessible to external traffic.

## 2. Understanding the `-n` Option in kubectl get

The `-n` option in `kubectl get` is used to specify the namespace from which to list resources.

### Default Namespace Query
```bash
kubectl get services
```

![Image](https://github.com/user-attachments/assets/172f1ec3-3ef8-42eb-a5a4-549a838f0cb3)

Shows resources from the default namespace, including user-created deployments and services.

### System Namespace Query
```bash
kubectl get pods,services -n kube-system
```

![Image](https://github.com/user-attachments/assets/8c20307b-a04a-43b1-ba46-23dde4943047)

Shows only system-managed components like:
- metrics-server
- kube-dns
- etcd
- kube-apiserver

**Key Point:** The `-n` option is crucial for switching namespace context in Kubernetes to view resources specific to that namespace.

## 3. Deployment Strategies: Rolling Update vs Recreate

### Rolling Update Strategy
- Updates Pods gradually, one at a time or in small batches
- **No downtime** - maintains availability during updates
- Keeps some Pods of the previous version running while rolling out the new version

### Recreate Strategy
- Terminates **all existing Pods first**
- Creates new Pods with the updated version
- **Causes brief downtime** due to the gap between old Pods termination and new Pods readiness

## 4. Deploying Spring Petclinic REST with Recreate Strategy

To deploy using the Recreate strategy, update your `deployment.yaml` manifest:

```yaml
spec:
  strategy:
    type: Recreate
```

### Deployment Commands
```bash
# Apply the manifest
kubectl apply -f deployment.yaml

# Check rollout status
kubectl rollout status deployment/spring-petclinic-rest
```

### Observed Behavior
Kubernetes deleted all existing Pods first, then created new ones, confirming the Recreate strategy was in effect with brief downtime.

## 5. Complete Recreate Deployment Manifest

Here's a complete `deployment-recreate.yaml` example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-petclinic-rest
  labels:
    app: spring-petclinic-rest
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: spring-petclinic-rest
  template:
    metadata:
      labels:
        app: spring-petclinic-rest
    spec:
      containers:
      - name: spring-petclinic-rest
        image: docker.io/springcommunity/spring-petclinic-rest:3.0.2
        ports:
        - containerPort: 9966
```

Apply with:
```bash
kubectl apply -f deployment-recreate.yaml
```

## 6. Benefits of Using Kubernetes Manifest Files

Using manifest files like `deployment.yaml` and `service.yaml` provides significant advantages over manual `kubectl create` and `kubectl expose` commands:

### Key Benefits
- **Version Control**: Manifest files can be tracked, versioned, and shared with the team
- **Consistency**: Ensures identical deployments across environments (dev, staging, production)
- **Efficiency**: `kubectl apply -f` is faster and simpler than remembering complex CLI commands
- **Reliability**: Easy to rollback or reapply configurations when needed
- **Documentation**: Serves as living documentation of your deployment configuration

### Professional Workflow
Compared to manual deployment commands, using manifests provides a more professional, automated, and reliable approach to Kubernetes deployments.

---

## Quick Reference Commands

```bash
# View resources in default namespace
kubectl get pods,services

# View resources in specific namespace
kubectl get pods,services -n kube-system

# Apply manifest files
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check deployment status
kubectl rollout status deployment/<deployment-name>

# Expose service in minikube
minikube service <service-name>
```