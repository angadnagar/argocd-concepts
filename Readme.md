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
# ArgoCD Notfications:
## ArgoCD Notifications is a built-in component of ArgoCD since v1.7+.

## It consists of:

## Notification Controller: watches ArgoCD Applications and triggers notifications.
## Triggers: define when to send (e.g., on sync failure).
## Templates: define what to send (message content).
## Subscriptions: define where to send (Slack, Email, etc.).
## Common use cases: Send an email when an Application fails to sync, Post a Slack message when sync succeeds.

## For Email, make sure to create App Password in Gmail account.(Open Google Account → Security → App passwords )

## 1. Install Triggers and Templates from the catalog
```yaml
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/notifications_catalog/install.yaml
```
## 2. Configure SMTP Secret
```yaml
# secret-smtp.yaml
# This file defines a Kubernetes Secret for storing SMTP credentials used by ArgoCD Notifications.

apiVersion: v1  # Specifies the API version for the Kubernetes object.
kind: Secret    # Declares that this object is a Secret.
metadata:
  name: argocd-notifications-secret  # Name of the Secret resource.
  namespace: argocd                 # Namespace where the Secret will be created.
type: Opaque  # Generic secret type for arbitrary user-defined data.
stringData:   # Allows you to provide secret data as unencoded strings (Kubernetes will encode them).
  email-username: "your-email@example.com"   # SMTP sender email address (replace with your own).
  email-password: "your-smtp-password"       # SMTP password or app password (replace with your own).

# Note:
# - This Secret is used by ArgoCD Notifications to authenticate with your SMTP server for sending emails.
# - Replace the placeholder values with your actual SMTP credentials.
# - Using stringData is convenient for writing secrets in plain text; Kubernetes will convert them to base64.
```

## kubectl apply -f secret-smtp.yaml

## 3. Configure Notification ConfigMap
## ArgoCD Notifications configuration lives in argocd-notifications-cm ConfigMap.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Context section: defines variables available in notification templates
  context: |
    argocdUrl: "http://<your-argocd-server>:8080"  # Replace with your ArgoCD server URL

  # Email service configuration: SMTP settings for sending emails
  service.email: |
    username: $email-username         # Email username (set as secret)
    password: $email-password         # Email password (set as secret)
    host: smtp.gmail.com              # SMTP server host
    port: 465                         # SMTP server port (SSL)
    from: $email-username             # Sender email address

  # Template for Degraded health: email content when app health is degraded
  template.email-health-degraded: |
    email:
      subject: "[ArgoCD] {{.app.metadata.name}} health is {{.app.status.health.status}}"
    message: |
      🚨 Application: {{.app.metadata.name}}
      📂 Namespace: {{.app.metadata.namespace}}
      🔄 Sync status: {{.app.status.sync.status}}
      📌 Revision: {{.app.status.sync.revision}}
      ❤️ Health status: {{.app.status.health.status}}
      📝 Health message: {{.app.status.health.message}}
      ⚙️ Last operation: {{ if .app.operationState }}{{ .app.operationState.phase }}{{ else }}<none>{{ end }}
      🔗 Details: {{.context.argocdUrl}}/applications/{{.app.metadata.name}}

  # Template for Deployed (synced + healthy): email content when app is successfully deployed
  template.email-deployed: |
    email:
      subject: "[ArgoCD] {{.app.metadata.name}} successfully deployed 🎉"
    message: |
      ✅ Application: {{.app.metadata.name}}
      📂 Namespace: {{.app.metadata.namespace}}
      🔄 Sync status: {{.app.status.sync.status}}
      📌 Revision: {{.app.status.sync.revision}}
      ❤️ Health status: {{.app.status.health.status}}
      ⚙️ Last operation: {{ if .app.operationState }}{{ .app.operationState.phase }}{{ else }}<none>{{ end }}
      Finished at: {{ if .app.operationState }}{{ .app.operationState.finishedAt }}{{ end }}
      🔗 Details: {{.context.argocdUrl}}/applications/{{.app.metadata.name}}

  # Trigger for Degraded health: sends email when app health is degraded
  trigger.on-health-degraded: |
    - when: app.status.health.status == 'Degraded'
      send: [email-health-degraded]

  # Trigger for Deployed: sends email when app is synced and healthy
  trigger.on-deployed: |
    - when: app.status.operationState.phase == 'Succeeded' and app.status.health.status == 'Healthy'
      send: [email-deployed]

# This ConfigMap configures ArgoCD Notifications to send email alerts for application health changes.
# It defines SMTP settings, notification templates, and triggers for sending emails on specific events.
```
## kubectl apply -f argocd-notifications-cm.yaml

## 4. Update the Application with Notification Subscription
```yaml
# chai-app.yaml
# This file defines an ArgoCD Application resource for deploying the chai-app.
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: chai-app # Name of the ArgoCD application
  namespace: argocd # Namespace where ArgoCD is installed
  annotations:
    # The email notifier must be configured in argocd-notifications-cm.yaml, The `.email` is the name of the notifier, i.e, `service.email` in the argocd-notifications-cm.yaml.
    # Subscribe to notifications when the app health is degraded.
    notifications.argoproj.io/subscribe.on-health-degraded.email: "<receiver@example.com>"  # you can add multiple email ids separated by comma like "<receiver@example.com>, <receiver2@example.com>"
    # Subscribe to notifications when the app is successfully deployed.
    notifications.argoproj.io/subscribe.on-deployed.email: "<receiver@example.com>" # you can add multiple email ids separated by comma like "<receiver@example.com>, <receiver2@example.com>"
spec:
  project: default # ArgoCD project this app belongs to
  source:
    repoURL: https://github.com/<your-username>/argocd-demos.git # Git repository containing the app manifests
    targetRevision: main # Git branch, tag, or commit to track
    path: applicationsets/chai-app # Path within the repo where manifests are located
  destination:
    server: https://kubernetes.default.svc # Kubernetes API server address (in-cluster)
    namespace: default # Namespace where the app will be deployed
  syncPolicy:
    automated:
      prune: true # Automatically delete resources that are no longer defined in Git
      selfHeal: true # Automatically correct drift between Git and cluster state

# Brief Info:
# This manifest enables GitOps for chai-app using ArgoCD.
# Notifications are set up for health degradation and deployment events.
# Automated sync ensures the cluster matches the desired state in Git.
```
## Now if application deployed then success mail will come and if application failed then app degraded mail will come.


# Argo CD Image Updater
## Argo CD Image Updater automates updating container images in ArgoCD-managed Applications. Instead of manually editing manifests every time a new image tag is pushed, Image Updater detects new versions and either updates your Git repository (recommended) or the ArgoCD Application directly.

## Why use Argo CD Image Updater?
## Saves manual effort: no more editing YAML for new image tags.
## Keeps applications updated with the latest images (patches, fixes).
## Maintains Git as the single source of truth (when using git write-back).
## Works alongside ArgoCD for a full GitOps workflow.

## Write-back mode
## When Image Updater finds a new image, it has two ways to “write back” the update:

## git write-back → creates a commit in your Git repo, updating the image tag in the manifests. ✅ Best practice for GitOps, because Git remains the source of truth.
## argocd write-back → directly changes the ArgoCD Application resource in the cluster (imperative). Faster, but changes are not recorded in Git.
## For our demo, we’ll use git write-back.

## Semantic Versioning (semver)
## semver stands for Semantic Versioning, a way of numbering versions like 1.0.0, 1.0.1, 1.1.0, 2.0.0.

## Format: MAJOR.MINOR.PATCH

## MAJOR → breaking changes (e.g., 1.x.x → 2.0.0)
## MINOR → new features (e.g., 1.1.0 → 1.2.0)
## PATCH → bug fixes (e.g., 1.1.1 → 1.1.2)
## If you set strategy to semver, Image Updater only upgrades within semver rules (e.g., from 1.0.0 → 1.0.1 or 1.1.0, but not to 2.0.0).

## The following update strategies are currently supported:

## semver - Update to the latest version of an image considering semantic versioning constraints
## latest/newest-build - Update to the most recently built image found in a registry
## digest - Update to the latest version of a given version (tag), using the tag's SHA digest
## name/alphabetical - Sorts tags alphabetically and update to the one with the highest cardinality
## For safety, we’ll use semver.

## Steps:
## 1. Create GitHub Personal Access Token (PAT) with repo permissions (for git write-back)
## 2. Install Argo CD Image Updater
```yaml
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/config/install.yaml
```
## verify pod is running
```yaml
kubectl -n argocd get pods -l app.kubernetes.io/name=argocd-image-updater
```

## 3. Prepare your Docker Hub image
## Log in to Docker Hub with your username and DockerHub PAT
```yaml
docker login
```

## Pull base image
```yaml
docker pull angadnagar/chai-devops:v1.0.0
```

## Tag the image with new version
```yaml
docker tag angadnagar/chai-devops:v1.0.0 angadnagar/chai-devops:v1.0.1
```

## Push to Docker hub
```yaml
docker push angadnagar/chai-devops:v1.0.1
```

## Update image in your deployment of chai-app


## 4. Configure Git credentials (for git write-back), create a secret with GitHub username and PAT(secret.yml)
```yaml
apiVersion: v1 # Specifies the API version for Kubernetes resources
kind: Secret # Declares that this resource is a Secret
metadata:
  name: argocd-image-updater-git-creds # Name of the Secret object
  namespace: argocd # Namespace where the Secret will be created
stringData:
  username: "<github-username>" # Your GitHub username for authentication
  password: "<personal-access-token>" # Your GitHub personal access token for authentication
```

## Create imageUpdater.yml
```yaml
apiVersion: argocd-image-updater.argoproj.io/v1alpha1
kind: ImageUpdater

metadata:
  name: chai-image-updater
  namespace: argocd

spec:
  applicationRefs:
    - namePattern: "chai-app"
      useAnnotations: true
```
## This is the bridge between the new v1.x architecture and the annotation configuration. When useAnnotations: true is set, Image Updater reads the argocd-image-updater.argoproj.io/* annotations from your Application.

## 5. Annotate your Application (chai-app)
```yaml
apiVersion: argoproj.io/v1alpha1 # Specifies the API version for the ArgoCD Application CRD
kind: Application               # Declares this resource as an ArgoCD Application
metadata:
  name: chai-app                # Name of the ArgoCD Application
  namespace: argocd             # Namespace where the Application resource will be created
  annotations:
    # Assign alias 'chai-app' to your image
    argocd-image-updater.argoproj.io/image-list: chai-app=<your-dockerhub-username>/chai-devops # Maps the image alias 'chai-app' to your DockerHub image

    # Use git write-back
    argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/argocd-image-updater-git-creds # Configures image updater to write changes back to Git using provided secret

    # Update strategy for chai-app image
    argocd-image-updater.argoproj.io/chai-app.update-strategy: semver # Uses semantic versioning for image updates

spec:
  project: default              # Associates this Application with the 'default' ArgoCD project
  source:
    repoURL: https://github.com/<your-github-username>/argocd-demos.git # Git repository containing the app manifests
    targetRevision: main        # Branch or tag to track in the repository
    path: image_updater/chai-app # Path within the repo where the manifests(kustomization) are located
  destination:
    server: https://kubernetes.default.svc # Kubernetes API server address (in-cluster)
    namespace: default           # Namespace in the cluster where app resources will be deployed
  syncPolicy:
    automated:                  # Enables automated sync for the application
      prune: true               # Automatically deletes resources that are no longer defined in Git
      selfHeal: true            # Automatically corrects drift between live and desired state

# Brief: This ArgoCD Application manifest configures automated deployment and image updates for the 'chai-app' using ArgoCD Image Updater, with changes written back to Git.
```


## Now push any new image to docker hub
```yaml
docker tag <your-dockerhub-username>/chai-devops:v1.0.1 <your-dockerhub-username>/chai-devops:v1.0.2
docker push <your-dockerhub-username>/chai-devops:v1.0.2
```

## check logs of image-updater
```yaml
kubectl -n argocd logs deploy/argocd-image-updater -f
```

## You should see logs like found newer image tag and creating git commit.

## Check git repo, you should see commit from  v1.0.1 → v1.0.2

## ArgoCD Detects the commit and syncs
## You can check your application deployment in ArgoCD server, that image is updated in deployment



# Monitoring ArgoCD(Prometheus + Grafana)
## 1. Verify Metrics Endpoints
```yaml
kubectl get svc -n argocd
```
## You should see services like argocd-metrics, argocd-server-metrics, argocd-repo-server, ArgoCD exposes metrics by default, if you installed ArgoCD with Manifests method

## 2. Install Prometheus and Grafana
```yaml
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create namespace monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```
## 3. Create ServiceMonitors
## We need to tell Prometheus to scrape ArgoCD metrics endpoints. We do this by creating ServiceMonitor resources.
```yaml
# argocd-service-monitors.yaml
# This file defines ServiceMonitor resources for monitoring ArgoCD components with Prometheus Operator.

apiVersion: monitoring.coreos.com/v1 # API version for ServiceMonitor
kind: ServiceMonitor                 # Resource type
metadata:
  name: argocd-metrics               # Name of the ServiceMonitor
  namespace: argocd                  # Namespace where ServiceMonitor is deployed
  labels:
    release: kube-prometheus-stack   # Label to associate with Prometheus release
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics # Selects services with this label, must match with ArgoCD metrics service, check with: kubectl get svc -n argocd
  endpoints:
  - port: metrics                    # Monitors the 'metrics' port

---
apiVersion: monitoring.coreos.com/v1 # API version for ServiceMonitor
kind: ServiceMonitor                 # Resource type
metadata:
  name: argocd-server-metrics        # Name of the ServiceMonitor
  namespace: argocd                  # Namespace where ServiceMonitor is deployed
  labels:
    release: kube-prometheus-stack   # Label to associate with Prometheus release
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server-metrics # Selects services with this label, must match with ArgoCD server metrics service, check with: kubectl get svc -n argocd
  endpoints:
  - port: metrics                    # Monitors the 'metrics' port

---
apiVersion: monitoring.coreos.com/v1 # API version for ServiceMonitor
kind: ServiceMonitor                 # Resource type
metadata:
  name: argocd-repo-server-metrics   # Name of the ServiceMonitor
  namespace: argocd                  # Namespace where ServiceMonitor is deployed
  labels:
    release: kube-prometheus-stack   # Label to associate with Prometheus release
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-repo-server # Selects services with this label, must match with ArgoCD repo server metrics service, check with: kubectl get svc -n argocd
  endpoints:
  - port: metrics                    # Monitors the 'metrics' port
```

## 4. Deploy apps

## 5. Access Prometheus and check on ui that metrics are coming or not, access Grafana & Import Dashboards



## Grafana Pre-Added Datasource (You can see Prometheus is already added)

## Now we can import Dashboards
## example: ArgoCD Operational Overview (ID: 19993): Detailed operational metrics



# Security & Scaling in ArgoCD
## RBAC in ArgoCD (Role-Based Access Control)
## RBAC Components
## Role → Named set of permissions (e.g., role:readonly)
## Policy → Maps roles to allowed/denied actions
## Subject → User or group bound to a role

## Group Assignment:
```yaml
g, <user/group>, <role>
```
## Policy Assignment:
```yaml
p, <role/user/group>, <resource>, <action>, <object>, <effect>
```

## Creating local users

```yaml
#argocd-user-cm.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm # standard label for ArgoCD config maps, used for selection by ArgoCD components
    app.kubernetes.io/part-of: argocd # It is useful to identify all resources that are part of ArgoCD
data:
  # Add local users with capabilities
  accounts.alice: apiKey, login  # Can generate tokens and login to UI
  accounts.bob: login            # Can only login to UI
  accounts.ci-user: apiKey       # Can only generate tokens (for automation)
```

## after applying this file, set password for this users
```yaml
argocd account update-password --account alice
argocd account update-password --account bob
```

## If needed, you can disabled admin user, using:
```yaml
kubectl patch -n argocd configmap argocd-cm --patch='{"data":{"admin.enabled": "false"}}'
```
```yaml
## Similarly, You can enable admin user, using
kubectl patch -n argocd configmap argocd-cm --patch='{"data":{"admin.enabled": "false"}}'
```

## Example RBAC Policy
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    # Built-in roles
    p, role:readonly, applications, get, */*, allow
    p, role:readonly, applications, sync, */*, deny
    p, role:admin, applications, *, */*, allow

    # Custom roles
    p, role:developer, applications, get, myproject/*, allow
    p, role:developer, applications, sync, myproject/*, allow

    # Bind users to roles
    g, alice, role:readonly
    g, bob, role:admin
    g, my-org:dev-team, role:developer  # SSO group

  # Default role for authenticated users
  policy.default: role:readonly
  
  # Control which scopes to examine for RBAC
  scopes: '[groups, email]'
```


## argocd-rbac-cm.yml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  # RBAC policy definitions in CSV format
  policy.csv: |
    # Developers can get applications in 'myproject'
    p, role:developer, applications, get, myproject/*, allow
    # Developers can sync applications in 'myproject'
    p, role:developer, applications, sync, myproject/*, allow 
    # Admins can perform any action on any application
    p, role:admin, applications, *, *, allow
    # Assign 'developer' role to user 'alice'
    g, alice, role:developer 
    # Assign 'admin' role to user 'bob'
    g, bob, role:admin 
  # Default role for users not explicitly assigned
  policy.default: role:readonly
```

## we can also validate rbac policy
```yaml
argocd admin settings rbac validate --policy-file argocd-rbac-cm.yaml
```
## Check user specific permissions
```yaml
argocd admin settings rbac can alice get applications "myproject/*" -n argocd
```











