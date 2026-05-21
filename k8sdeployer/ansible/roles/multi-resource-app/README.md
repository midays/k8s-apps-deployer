# multi-resource-app

A minimal Kubernetes application demonstrating all 4 basic resource types:
- ✅ **Deployment** - Runs nginx with 1 replica
- ✅ **Service** - Exposes the app on port 80
- ✅ **ConfigMap** - Stores app configuration and nginx config
- ✅ **Secret** - Stores credentials (username/password)

## Purpose

This app is designed for testing Kubernetes migration tools (like `crane validate`) that need to verify handling of multiple resource types.

## Resources Created

1. **Secret**: `multi-resource-app-secret`
   - Contains base64-encoded username and password
   - Referenced by Deployment as environment variables

2. **ConfigMap**: `multi-resource-app-config`
   - Contains app metadata (name, environment, version)
   - Contains nginx configuration
   - Contains custom index.html page
   - Referenced by Deployment as environment variables and volume mounts

3. **Service**: `multi-resource-app`
   - Type: ClusterIP (default) or NodePort
   - Port: 80 → 8080

4. **Deployment**: `multi-resource-app`
   - Image: nginx:1.25-alpine
   - Replicas: 1 (configurable)
   - Uses ConfigMap for config files and environment variables
   - Uses Secret for credentials

## Usage

### Deploy
```bash
k8sdeployer deploy multi-resource-app --namespace multi-resource-app --context your-cluster
```

### With custom variables
```bash
k8sdeployer deploy multi-resource-app \
  --namespace multi-resource-app \
  --context your-cluster \
  --extra-vars '{"app_environment": "development", "replicas": 2, "service_type": "NodePort"}'
```

### Validate
```bash
k8sdeployer validate multi-resource-app --namespace multi-resource-app --context your-cluster
```

### Cleanup
```bash
k8sdeployer remove multi-resource-app --namespace multi-resource-app --context your-cluster
```

## Available Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `app_name` | `multi-resource-app` | Application name |
| `namespace` | Required | Kubernetes namespace |
| `replicas` | `1` | Number of pod replicas |
| `app_environment` | `production` | Environment name |
| `app_version` | `1.0.0` | Application version |
| `service_type` | `ClusterIP` | Service type (ClusterIP or NodePort) |
| `node_port` | `30090` | NodePort (if service_type=NodePort) |
| `nginx_image` | `nginx:1.25-alpine` | Nginx container image |

## Testing with Crane

This app is ideal for testing `crane validate`:

```bash
# Deploy on source cluster
k8sdeployer deploy multi-resource-app --namespace multi-resource-app --context source-cluster

# Export with crane
crane export --namespace multi-resource-app --export-dir ./export --context source-cluster

# Validate against target
crane validate --export-dir ./export --context target-cluster

# Transform if needed
crane transform --export-dir ./export

# Validate transformed output
crane validate --transform-dir ./transform --context target-cluster

# Apply to target
crane apply --transform-dir ./transform --context target-cluster
```

## Accessing the App

If deployed with `service_type: NodePort`:
```bash
curl http://<node-ip>:30090
```

If deployed with `service_type: ClusterIP`:
```bash
kubectl port-forward -n multi-resource-app svc/multi-resource-app 8080:80
curl http://localhost:8080
```

You should see an HTML page listing all the resource types the app uses.
