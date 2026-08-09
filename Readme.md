# What is Gitops?

## GitOps is a modern approach to continuous delivery (CD) for Kubernetes and cloud-native applications. It uses Git repositories as the single source of truth for declaring the desired state of your system. Instead of manually deploying or configuring applications, everything (apps, infrastructure, policies) is written as declarative code (YAML/JSON) and stored in Git. A GitOps operator (like ArgoCD) continuously ensures the cluster matches what’s in Git.

# GitOps is built on four key principles:

## Declarative
## The desired state of the system is described declaratively (e.g., YAML manifests).
## Example: A Deployment manifest defines how many replicas and which container image.

# Versioned & Immutable

## Desired state is stored in Git (or another versioned system).
## Every change is auditable and traceable.

# Automated

## A controller (like ArgoCD) continuously watches Git and the cluster.If there’s drift (difference), it automatically applies changes.

# Observable

## System health and deployment status are visible. Git history + dashboards + alerts provide observability.

# GitOps vs Traditional CICD
<img width="666" height="320" alt="Screenshot 2026-08-01 224413" src="https://github.com/user-attachments/assets/4e1f908e-da73-4f25-8177-09c705ea9b5f" />

# ArgoCD Architecture
<img width="728" height="401" alt="Screenshot 2026-08-01 224614" src="https://github.com/user-attachments/assets/e22e4ee0-e864-4df0-be0f-bd637ba63df6" />


## Api Server
## It handles all the requests and responses and then it gave this request to repository server

## Repository Server
## It will make a copy or will make a proper sync from that github repo

## Application Controller
## It will talk with Api Server and Repository Server and will deploy on our Kubernetes Cluster

# ArgoCD vs FluxCD vs Jenkins X
<img width="662" height="190" alt="Screenshot 2026-08-01 225349" src="https://github.com/user-attachments/assets/fe0f2311-f01c-4ac8-bda3-aa2738203189" />



# Setup ArgoCD
## Docker → Required for Kind to run containers as cluster nodes.
```yaml
sudo apt-get update
sudo apt install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
docker --version

docker ps
```

## Kind (Kubernetes in Docker) → To create the cluster.
## Installation: https://kind.sigs.k8s.io/docs/user/quick-start/#installation

## kubectl → To interact with the cluster.
## Installation: https://kubernetes.io/docs/tasks/tools/

## Helm (for Helm-based installation)
## Installation: https://helm.sh/docs/intro/install/

# Step 1: Create Kind Cluster
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "172.31.19.178"   # Change this to your EC2 private IP (run "hostname -I" to check or from your EC2 dashboard)
  apiServerPort: 33893
nodes:
  - role: control-plane
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
```

## Create cluster (kind create cluster --name argocd-cluster --config kind-config.yaml)

# Step 2 : ArgoCD Installation
## Method 1 Using Helm
## Add Argo Helm repo
```yaml
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

## Create namespace
```yaml
kubectl create namespace argocd
```

## Install ArgoCD
```yaml
helm install argocd argo/argo-cd -n argocd
```

## Verify installation
```yaml
kubectl get pods -n argocd
kubectl get svc -n argocd
```

## Access the ArgoCD UI
```yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address=0.0.0.0 &
```

## Get initial admin password
```yaml
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

## Method 2 using official manifests
## Create namespace
```yaml
kubectl create namespace argocd
```

## Apply ArgoCD installation manifest
```yaml
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## Verify installation
```yaml
kubectl get pods -n argocd
kubectl get svc -n argocd
```

## Access the ArgoCD UI
```yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address=0.0.0.0 &
```

## Get initial admin password
```yaml
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

## Installation of ArgoCD Cli
```yaml
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```
## Login to ArgoCD Cli
```yaml
argocd login <instance_public_ip>:8080 --username admin --password <initial_password> --insecure

--insecure flag is required when using port-forward with self-signed TLS certs. For production, you’d configure proper TLS certs (then --insecure is not needed).
```


# Deployment using UI
## First we will connect Github Repo(n ArgoCD UI, go to Settings → Repositories -> Connect Repo)
<img width="750" height="498" alt="Screenshot 2026-08-06 221236" src="https://github.com/user-attachments/assets/e4d1e778-e236-47e7-b211-25d31438e896" />


## Adding cluster to ArgoCD Server 
## 1. Check your config contexts:
```yaml
kubectl config get-contexts
```
## 2. Add the cluster to ArgoCD: (cli)
```yaml
argocd cluster add kind-argocd-cluster --name argocd-cluster --insecure
```

## 3. Verify using: (on ui ->  ArgoCD Server: Settings → Clusters.)
```yaml
argocd cluster list
```

## Create Application in ArgoCD UI
<img width="697" height="506" alt="Screenshot 2026-08-06 221323" src="https://github.com/user-attachments/assets/c7974a26-c0b0-413e-b1c3-d58a16fc8559" />
<img width="740" height="509" alt="Screenshot 2026-08-06 221335" src="https://github.com/user-attachments/assets/39576b1d-dceb-4eeb-b975-37a148f608c5" />


# Deployment using CLI
## Adding cluster to ArgoCD Server 
<img width="787" height="281" alt="Screenshot 2026-08-06 221253" src="https://github.com/user-attachments/assets/c1276dad-971b-498c-9ee5-923006dd39de" />

## 1. Check your config contexts:
```yaml
kubectl config get-contexts
```
## 2. Add the cluster to ArgoCD: (cli)
```yaml
argocd cluster add kind-argocd-cluster --name argocd-cluster --insecure
```

## 3. Verify using: (on ui ->  ArgoCD Server: Settings → Clusters.)
```yaml
argocd cluster list
```

## create application using cli

```yaml
argocd app create apache-app \
  --repo https://github.com/<your-username>/argocd-demos.git \
  --path cli_approach/apache \
  --dest-server https://<your_added_cluster_url> \
  --dest-namespace default \
  --sync-policy automated \
  --self-heal \
  --auto-prune
  ```

## verify app creation
```yaml
argocd app list
```

# Deployment using declarative yaml
## create a CRD for creating application for ArgoCD
```yaml
apiVersion: argoproj.io/v1alpha1   # API group for ArgoCD resources
kind: Application                  # Resource type is "Application"
metadata:
  name: online-shop-app            # Name of this ArgoCD application
  namespace: argocd                # Must be created in the 'argocd' namespace
spec:
  project: default                 # ArgoCD Project (logical grouping of apps)
  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git   # Git repo containing manifests
    targetRevision: main           # Git branch or tag (e.g., main, dev, release-1.0)
    path: declarative_approach/online_shop   # Path inside repo where manifests live
  destination:
    server: <argocd_cluster_server_url>   # Target cluster API
    namespace: default             # Namespace in which to deploy the app
  syncPolicy:                      # Defines how ArgoCD syncs the app
    automated:                     # Enable auto-sync
      prune: true                  # Delete resources removed from Git
      selfHeal: true               # Fix drift if resources are changed manually
```

## apply the application crd
```yaml
kubectl apply -f online_shop_app.yml -n argocd
```


# ArgoCD Core Features:
## 1. Projects:
## It is similar as we have namespace in Kubernetes, it is kind of group
## project.yml
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: frontend-team
  namespace: argocd

spec:
  description: This project holds new frontend app
  sourceRepos:
    - https://github.com/<your-username>/argocd-demos.git
  destinations:
    - namespace: frontend
      server: <added_argocd_cluster_server_url>
  clusterResourceWhitelist:    # Which cluster-wide resources are allowed (e.g., CRDs)
    - group: "*"               # '*' means allow all groups
      kind: "*"                # '*' means allow all kinds
  namespaceResourceWhitelist:  # Which namespace-level resources are allowed
    - group: "*"               # Here also allowing everything
      kind: "*"
  roles:
    - name: frontend-admins
      description: Admins for frontend team
      policies:                # Policies associated with this role
        # syntax: - p, proj:<project-name>:<role-name>, applications, <action>, <project>/<app-name>, permission(allow|deny)
        - p, proj:frontend-team:frontend-admins, applications, *, frontend-team/*, allow  # Full access to all apps in this Project, Here p = policy, proj = project, applications = resource, * = action (all actions), frontend-team/* = resource name pattern, allow = effect (allow or deny)
```

## create application under project
```yaml
apiVersion: argoproj.io/v1alpha1 # ArgoCD Application resource
kind: Application               # Kind of resource
metadata:                        # Metadata section                         
  name: nginx-frontend          # App name
  namespace: argocd             # Must exist in ArgoCD namespace
spec:
  project: frontend-team        # Assigns app to the frontend-team Project
  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git  # Git repo
    targetRevision: main        # Branch to watch
    path: ui_approach/nginx     # Path in repo with manifests
  destination:
    server: <added_argocd_cluster_server_url>  # Target cluster, run `argocd cluster list` to see
    namespace: frontend          # Namespace inside that cluster
  syncPolicy:                    # Sync policy settings
    automated:                   # Enable automated sync
      prune: true                 # Delete resources removed from Git
      selfHeal: true              # Correct drift if resources change in cluster
    syncOptions:                 # Additional sync options
      - CreateNamespace=true      # Auto-create namespace if it doesn't exist
```


# App of Apps Pattern in ArgoCD
## App of Apps pattern, a technique for managing multiple applications in ArgoCD using a single root application.
## Normally, you create one Application CRD per app.
## But in large setups, you may need to manage dozens of apps.
## The App of Apps pattern solves this by:
## Defining one root application.
## That root application points to a directory in Git that contains child Application manifests.
## ArgoCD syncs the root → which then creates child apps automatically.
## we will push all our child apps in one directory and in root application yaml file which is shown below we will provide that path where all child apps are present so that it will apply and create these apps that why it is called App of Apps Pattern
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app                # Root application name
  namespace: argocd             # Always in ArgoCD namespace
spec:
  project: default              # Can also assign to a Project if needed
  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git   # Repo containing child apps
    targetRevision: main        # Git branch to track
    path: app_of_apps/apps   # Path where child app manifests are stored
  destination:
    server: <argocd_cluster_server_url>  # Deploying inside the same cluster
    namespace: argocd             # Applications (CRDs) must live in argocd namespace
  syncPolicy:
    automated:
      prune: true                 # Prune removed child apps
      selfHeal: true              # Auto-fix drift
```

# Multi Cluster Management
## By default, ArgoCD manages only the cluster where it is installed (in-cluster).
## By adding more clusters with argocd cluster add, ArgoCD can deploy apps into multiple target clusters.
## This is how enterprises manage Dev → Stage → Prod from a single GitOps control plane.
## when creating Applications, in destination server we can put what cluster we want
```yaml
dev_app.yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git
    targetRevision: main
    path: ui_approach/nginx        # Path to nginx manifests
  destination:
    server: https://kubernetes.default.svc    # from `argocd cluster list`(for dev we are deployin on in-cluster)
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true



prod_app.yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: online-shop-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git
    targetRevision: main
    path: multicluster/online-shop # Path within the repo
  destination:
    server: <prod-cluster-server-url> # Production cluster API server URL here, run `argocd cluster list` to get the URL
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

# ApplicationSet
## ApplicationSets are a way to dynamically generate multiple ArgoCD Applications from a single manifest.Instead of writing many Application YAMLs, you define a template + a generator, and ArgoCD creates the apps for you.
## Generators decide how apps are created:
## List Generator → define a static list of apps.
## Cluster Generator → deploy the same app across multiple clusters.
## Git Generator → scan a repo and create an app per folder.

## List Generator
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-list
  namespace: argocd
spec:
  # The list generator lets you enumerate a static set of elements (apps).
  generators:
    - list:
        elements:
          - app: nginx
            path: ui_approach/nginx     # path in repo for this app
          - app: online-shop
            path: multicluster/online-shop   # path in repo for this app
          - app: chaiapp
            path: applicationsets/chai-app      # path in repo for this app
  # Template that will be used to generate individual Application CRs
  template:
    metadata:
      # name template uses the element key 'app'
      name: '{{app}}-list'
    spec:
      project: default
      source:
        repoURL: https://github.com/<your-username>/argocd-demos.git
        targetRevision: main
        path: '{{path}}'                # resolved from element
      destination:
        server: https://kubernetes.default.svc
        namespace: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

## Git Generator
## This will scan repo directories and auto-create apps for each folder found.
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-git
  namespace: argocd
spec:
  # Git generator scans the repo and generates an Application for each matching directory
  generators:
    - git:
        repoURL: https://github.com/<your-username>/argocd-demos.git
        revision: main
        # 'directories' accepts globs; each matched path becomes an entry
        directories:
          - path: "git_generator/*"     # will match git_generator/apache, git_generator/online-shop and git_generator/chai-app folders.
  template:
    metadata:
      # path.basename is the final folder name (e.g., 'nginx' or 'apache')
      name: '{{path.basename}}-git'
    spec:
      project: default
      source:
        repoURL: https://github.com/<your-username>/argocd-demos.git
        targetRevision: main
        path: '{{path}}'            # 'path' is the full matched path (e.g., 'git_generator/apache')
      destination:
        server: https://kubernetes.default.svc
        namespace: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true

# Notes:
# - This is ideal for monorepos or where each app lives in its own folder.
# - Ensure the repo is reachable by ArgoCD (public or added via argocd repo add).
```

## Cluster Generator
## This will deploy chai-app into all clusters registered in ArgoCD.
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-cluster
  namespace: argocd
spec:
  # Cluster generator will iterate over clusters registered in ArgoCD (via `argocd cluster add`)
  generators:
    - clusters: {}   # empty object => include all registered clusters (you can filter with labelSelectors)
  template:
    metadata:
      # Use the cluster name ({{name}}) to create unique app names per cluster
      name: '{{name}}-chai-app'
    spec:
      project: default
      source:
        repoURL: https://github.com/<your-username>/argocd-demos.git
        targetRevision: main
        path: applicationsets/chai-app    # same app deployed to all clusters
      destination:
        # destination.server will be filled with '{{server}}' for each cluster entry generated
        server: '{{server}}'
        namespace: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true

# Notes:
# - After applying this ApplicationSet ArgoCD will create one Application per cluster it knows about.
# - If you want to limit to specific clusters, use the clusters generator with labelSelectors, or use a List generator instead.
```











