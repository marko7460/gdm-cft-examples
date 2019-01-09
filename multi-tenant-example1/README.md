# Intro
Example how to crete two tiers in a single GKE cluster

# Prerequisites
1. Create project `acme-corp-001`
2. `kubectl` installed

# Setup gke cluster
1. Run `cft apply gke-cluster.yaml`

# Kubectl
To access the gke cluster from your laptop you will have to manually add your laptop's public IP to Master Authorized Networks on GKE cluster.
To authenticate yourself against the cluster run `gcloud container clusters get-credentials corp-gke --region us-east1 --project acme-corp-001`

# Namespaces
To create namespaces:
1. Create `team1` namespace: `kubectl create -f namespace-team1.json`
1. Create `team1` namespace: `kubectl create -f namespace-team2.json`
1. `kubectl get namespaces --show-labels`
1. Create contexts for two namespaces:
   1. `kubectl config set-context team1 --namespace=team1 --cluster=gke_acme-corp-001_us-east1_corp-gke --user=gke_acme-corp-001_us-east1_corp-gke` 
   1. `kubectl config set-context team2 --namespace=team2 --cluster=gke_acme-corp-001_us-east1_corp-gke --user=gke_acme-corp-001_us-east1_corp-gke` 

# Create Pods
Create pods in namespace team1:
1. `kubectl config use-context team1`
1. `kubectl create snowflake --image=busybox`
1. `kubectl get deployment` or `kubectl get pods`
1. Check no pods in team2 namespace: 
   1. `kubectl config use-context team2` 
   1. `kubectl get pods` 

Create pods in namespace team2:
1. `kubectl config use-context team2`
1. `kubectl create deployment team2http --image=nginx`
1. `kubectl scale deployment team2http --replicas 5`
1. `kubectl get pods`. You should see 5 pods
1. Check no team2http pods in team1 namespace: 
   1. `kubectl config use-context team1` 
   1. `kubectl get pods`

# Cleanup
1. `cft delete gke-cluster.yaml`