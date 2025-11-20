#### Deploying flux operator on EKS cluster n configuring Github repo sync

##### Add flux helm repo
```
$ helm repo add fluxcd-community https://fluxcd-community.github.io/helm-charts
```

##### Helm repo update
```
$ helm repo update
```

##### Create flux namespace
```
$ kubectl create namespace flux-system
```

#####  Install Flux Using Helm
```
$ helm install flux fluxcd-community/flux2 \
  --namespace flux-system \
  --create-namespace
```

##### Verify Flux Installation
```
$ kubectl get pods -n flux-system
$ kubectl get deployments -n flux-system
```
- You should see pods for:
  - source-controller
  - kustomize-controller
  - helm-controller
  - notification-controller

##### Configure Git Repository Source
```
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/sachinratan/go-app-code-repo.git
  ref:               
    branch: main
  secretRef:
    name: flux-system
```

- Apply the configuration:
```
kubectl apply -f git-repository.yaml
gitrepository.source.toolkit.fluxcd.io/flux-system created
```

##### Create Helm Repository Source
```
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: my-helm-repo
  namespace: flux-system
spec:
  interval: 5m
  url: https://github.com/sachinratan/jnks-k8s-helm-int-repo.git
EOF
```
- Apply the configuration:
```
kubectl apply -f git-repository.yaml
Warning: v1beta2 HelmRepository is deprecated, upgrade to v1
helmrepository.source.toolkit.fluxcd.io/my-helm-repo created
```

##### Create Helm Release for Automatic Sync
```
apiVersion: helm.toolkit.fluxcd.io/v2     
kind: HelmRelease
metadata:
  name: my-application
  namespace: default
spec:
  interval: 5m
  chart:
    spec:
      chart: my-chart-name
      version: '>=1.0.0'
      sourceRef:
        kind: HelmRepository
        name: my-helm-repo
        namespace: flux-system
  values:
    # Your custom values here
    replicaCount: 2
```
- Apply the configuration:
```
kubectl apply -f git-repository.yaml
helmrelease.helm.toolkit.fluxcd.io/my-application created
```

##### Monitor Flux Synchronization

###### - Check HelmRepository status
```
kubectl get helmrepositories -n flux-system
```
###### - Check HelmRelease status
```
kubectl get helmreleases -A
```
###### - View detailed status
```
kubectl describe helmrelease my-application -n default
```
###### - Check Flux logs
```
kubectl logs -n flux-system deployment/helm-controller -f
```
