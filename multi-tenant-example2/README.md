# Intro
Example how to create two tier gke clusters where one tier serves developers and the other tier serves as staging

# Prerequisites
1. Manually create projects:
   1. acme-infra-001
   1. acme-dev-001
   1. acme-staging-001
2. Run `gcloud compute shared-vpc enable acme-infra-001` to enable shared VPC on acme-infra-001
3. Run `gcloud compute shared-vpc associated-projects add acme-dev-001 --host-project acme-infra-001` to add acme-dev-001 as a service project
3. Run `gcloud compute shared-vpc associated-projects add acme-staging-001 --host-project acme-infra-001` to add acme-staging-001 as a service project

# Setup Networking
1. Run `cft apply network.yaml` to create shared VPC and all the subnets.
2. Add dev-gke-nodes-subnet to acme-dev-001 project:
    1. `gcloud beta compute networks subnets get-iam-policy dev-gke-nodes-subnet --region us-east1 --project acme-infra-001 --format json > subnet-dev-policy.json`
    1. Modify the subnet-dev-policy.json file, adding the IAM members who will become Service Project Admins with access to the subnet. Replace each `[SERVICE_PROJECT_ADMIN]` with the email address of an IAM user from the service project
        ```json
        {
        "bindings": [
           {
             "members": [
                 "user:[SERVICE_PROJECT_ADMIN]",
                 "user:[SERVICE_PROJECT_ADMIN]"
             ],
             "role": "roles/compute.networkUser"
           }
        ],
        "etag": "[ETAG_STRING]"
        }
        ```
    1. `gcloud beta compute networks subnets set-iam-policy dev-gke-nodes-subnet subnet-dev-policy.json --region us-east1 --project acme-infra-001`
3. Add staging-gke-nodes-subnet to acme-staging-001 project:
    1. `gcloud beta compute networks subnets get-iam-policy dev-staging-nodes-subnet --region us-east1 --project acme-infra-001 --format json > subnet-staging-policy.json`
    1. Modify the subnet-staging-policy.json file, adding the IAM members who will become Service Project Admins with access to the subnet. Replace each `[SERVICE_PROJECT_ADMIN]` with the email address of an IAM user from the service project
        ```json
        {
        "bindings": [
           {
             "members": [
                 "user:[SERVICE_PROJECT_ADMIN]",
                 "user:[SERVICE_PROJECT_ADMIN]"
             ],
             "role": "roles/compute.networkUser"
           }
        ],
        "etag": "[ETAG_STRING]"
        }
        ```
    1. `gcloud beta compute networks subnets set-iam-policy staging-gke-nodes-subnet subnet-staging-policy.json --region us-east1 --project acme-infra-001`

# Setup gke clusters
1. Run `cft apply dev-gke.yaml`
2. Run `cft apply staging-gke.yaml`

# References
https://cloud.google.com/vpc/docs/provisioning-shared-vpc#enable-shared-vpc-host