# Google Cloud SDK: gcloud CLI Reference

The Google Cloud SDK gcloud CLI is the primary command-line interface for managing Google Cloud Platform resources, authentication, and developer workflows. It provides a unified command structure for interacting with over 100 Google Cloud services, from compute and storage to AI/ML and security management. The CLI supports authentication management, local configuration, resource lifecycle operations, and complex multi-service workflows through a consistent verb-based command syntax.

This documentation snapshot contains reference material for key gcloud command groups including AI/ML services (Vertex AI, AI Platform), security services (Access Context Manager, IAM), data services (AlloyDB, Bigtable, Batch), and infrastructure management (Compute Engine, Container/GKE). The gcloud CLI follows a hierarchical command structure (gcloud GROUP SUBGROUP COMMAND) with standardized flags across all commands, supporting multiple output formats, filtering, pagination, and service account impersonation for seamless integration into automated workflows and CI/CD pipelines.

## Core gcloud CLI Structure and Global Flags

Understanding the base command structure and universal flags available across all gcloud commands.

```bash
# Basic command structure
gcloud [GROUP] [SUBGROUP] [COMMAND] [POSITIONAL_ARGS] [FLAGS]

# Global flags available on all commands
gcloud compute instances list \
  --project=my-project \
  --account=user@example.com \
  --format=json \
  --filter="status:RUNNING" \
  --quiet

# Configure default properties
gcloud config set core/project my-project
gcloud config set core/account user@example.com
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# View current configuration
gcloud config list

# Output:
# [core]
# account = user@example.com
# project = my-project
# [compute]
# region = us-central1
# zone = us-central1-a

# Use named configurations for multiple environments
gcloud config configurations create production
gcloud config configurations activate production
gcloud config set project prod-project-id

# View available properties
gcloud config list --all
```

## Authentication and Service Account Impersonation

Managing OAuth2 credentials and impersonating service accounts for API access.

```bash
# Authenticate with user account
gcloud auth login

# Authenticate for application default credentials (ADC)
gcloud auth application-default login

# List authenticated accounts
gcloud auth list

# Output:
#       Credentialed Accounts
# ACTIVE  ACCOUNT
# *       user@example.com
#         admin@example.com

# Impersonate service account for single command
gcloud compute instances list \
  --impersonate-service-account=my-service-account@project.iam.gserviceaccount.com

# Impersonation delegation chain (sa1 impersonates sa2)
gcloud storage buckets list \
  --impersonate-service-account=sa1@project.iam.gserviceaccount.com,sa2@project.iam.gserviceaccount.com

# Use access token file (for CI/CD)
gcloud compute instances list \
  --access-token-file=/path/to/token.txt

# Set billing project for quota (useful for shared VPC)
gcloud services enable compute.googleapis.com \
  --billing-project=billing-project-id \
  --project=shared-vpc-host
```

## Vertex AI: Managing AI Endpoints

List and manage Vertex AI model endpoints for online predictions.

```bash
# List all Vertex AI endpoints in a region
gcloud ai endpoints list \
  --project=my-ml-project \
  --region=us-central1

# Expected output:
# ENDPOINT_ID           DISPLAY_NAME          CREATE_TIME
# 1234567890123456789   production-model      2024-01-15T10:30:00
# 9876543210987654321   staging-model         2024-01-20T15:45:00

# List only Model Garden endpoints
gcloud ai endpoints list \
  --project=my-ml-project \
  --region=us-central1 \
  --list-model-garden-endpoints-only

# Filter endpoints by display name pattern
gcloud ai endpoints list \
  --region=us-central1 \
  --filter="displayName:production*" \
  --limit=10 \
  --sort-by=createTime

# Get endpoint URIs for scripting
gcloud ai endpoints list \
  --region=us-central1 \
  --uri \
  --format="value(name)"

# Output:
# projects/123/locations/us-central1/endpoints/456
# projects/123/locations/us-central1/endpoints/789

# Describe specific endpoint
gcloud ai endpoints describe 1234567890123456789 \
  --region=us-central1 \
  --format=json
```

## AI Platform: Running Online Predictions

Execute online predictions against deployed AI Platform models.

```bash
# Prepare input data (instances.json)
cat > instances.json << 'EOF'
{"images": [0.0, 0.1, 0.2, 0.3], "key": 3}
{"images": [0.5, 0.6, 0.7, 0.8], "key": 2}
EOF

# Run prediction on default model version
gcloud ai-platform predict \
  --model=my-model \
  --json-instances=instances.json \
  --region=us-central1

# Expected output:
# PREDICTIONS
# [0.85, 0.12, 0.03]
# [0.23, 0.67, 0.10]

# Run prediction on specific model version
gcloud ai-platform predict \
  --model=my-model \
  --version=v2 \
  --json-instances=instances.json \
  --region=us-central1

# Use full JSON request body format
cat > request.json << 'EOF'
{
  "instances": [
    {"x": [1, 2], "y": [3, 4]},
    {"x": [-1, -2], "y": [-3, -4]}
  ]
}
EOF

gcloud ai-platform predict \
  --model=my-model \
  --json-request=request.json \
  --region=global

# Run prediction with stdin input
echo '{"features": [1.0, 2.0, 3.0]}' | \
gcloud ai-platform predict \
  --model=my-model \
  --json-instances=- \
  --region=us-central1

# Specify TensorFlow signature name
gcloud ai-platform predict \
  --model=my-model \
  --json-instances=instances.json \
  --signature-name=serving_default \
  --region=us-central1

# Use text instances for simple models
cat > text_instances.txt << 'EOF'
107,4.9,2.5,4.5,1.7
100,5.7,2.8,4.1,1.3
EOF

gcloud ai-platform predict \
  --model=iris-classifier \
  --text-instances=text_instances.txt \
  --region=us-central1
```

## AI Platform: Streaming Job Logs

Monitor training job logs in real-time for debugging and progress tracking.

```bash
# Stream logs from a running training job
gcloud ai-platform jobs stream-logs my-training-job

# Output (continuous streaming):
# INFO 2024-01-15 10:30:15 +0000 master-replica-0: Starting training...
# INFO 2024-01-15 10:30:17 +0000 master-replica-0: Epoch 1/10
# INFO 2024-01-15 10:30:25 +0000 master-replica-0: Loss: 0.456, Accuracy: 0.823
# ...

# Stream logs for specific task with custom polling interval
gcloud ai-platform jobs stream-logs my-distributed-job \
  --task-name=worker-0 \
  --polling-interval=30

# Enable multiline log support (for stack traces)
gcloud ai-platform jobs stream-logs my-training-job \
  --allow-multiline-logs

# Stream logs and save to file
gcloud ai-platform jobs stream-logs my-training-job | tee training_logs.txt

# Describe job operation status
gcloud ai-platform operations describe \
  projects/my-project/operations/12345 \
  --region=us-central1

# Expected output:
# done: true
# metadata:
#   '@type': type.googleapis.com/google.cloud.ml.v1.OperationMetadata
#   operationType: CREATE_MODEL
#   createTime: '2024-01-15T10:30:00Z'
#   endTime: '2024-01-15T11:45:00Z'
# name: projects/123/operations/12345
# response:
#   '@type': type.googleapis.com/google.cloud.ml.v1.Model
#   name: projects/123/models/my-model
```

## Access Context Manager: Cloud Access Bindings

Create and manage security policies that bind access levels to Google Groups.

```bash
# Create cloud access binding with access level
gcloud access-context-manager cloud-bindings create \
  --group-key=security-team@example.com \
  --level=accessPolicies/123456/accessLevels/high_security \
  --organization=1234567890

# Expected output:
# Created cloud access binding [//accesscontextmanager.googleapis.com/organizations/1234567890/gcpUserAccessBindings/abc123]

# Create binding with session settings (2-hour session expiry)
gcloud access-context-manager cloud-bindings create \
  --group-key=contractors@example.com \
  --organization=1234567890 \
  --session-length=2h \
  --session-reauth-method=LOGIN \
  --level=accessPolicies/123456/accessLevels/contractor_access

# Create binding with dry-run level for testing
gcloud access-context-manager cloud-bindings create \
  --group-key=beta-testers@example.com \
  --level=accessPolicies/123456/accessLevels/production_access \
  --dry-run-level=accessPolicies/123456/accessLevels/test_access \
  --organization=1234567890

# Create binding with per-application settings using YAML file
cat > binding.yaml << 'EOF'
scopedAccessSettings:
  - scope:
      clientScope:
        restrictedClientApplications:
          - clientId: "123456789.apps.googleusercontent.com"
            displayName: "Internal HR App"
    activeSettings:
      accessLevels:
        - "accessPolicies/123456/accessLevels/hr_access"
      sessionSettings:
        sessionLength: "4h"
        sessionReauthMethod: PASSWORD
EOF

gcloud access-context-manager cloud-bindings create \
  --group-key=hr-team@example.com \
  --organization=1234567890 \
  --binding-file=binding.yaml

# Create global and per-app settings together
gcloud access-context-manager cloud-bindings create \
  --group-key=engineering@example.com \
  --organization=1234567890 \
  --session-length=8h \
  --session-reauth-method=SECURITY_KEY \
  --level=accessPolicies/123456/accessLevels/engineering_base \
  --dry-run-level=accessPolicies/123456/accessLevels/engineering_strict \
  --binding-file=engineering_apps.yaml

# Session settings with infinite session (disable expiry)
gcloud access-context-manager cloud-bindings create \
  --group-key=admins@example.com \
  --organization=1234567890 \
  --session-length=0 \
  --level=accessPolicies/123456/accessLevels/admin_access
```

## AlloyDB: Database User Management

Create and configure database users with various authentication types.

```bash
# Create built-in database user with password
gcloud alpha alloydb users create db-admin \
  --cluster=production-cluster \
  --region=us-central1 \
  --password=SecurePassword123! \
  --type=BUILT_IN

# Expected output:
# Create request issued for: [db-admin]
# Waiting for operation [projects/my-project/locations/us-central1/operations/operation-123] to complete...done.
# Created user [db-admin].

# Create IAM-based user with specific database roles
gcloud alpha alloydb users create analyst-group@example.com \
  --cluster=analytics-cluster \
  --region=us-central1 \
  --type=IAM_BASED \
  --db-roles=pg_read_all_data,custom_analyst_role \
  --superuser=false

# Create superuser for administrative tasks
gcloud alpha alloydb users create superadmin \
  --cluster=production-cluster \
  --region=us-central1 \
  --password=SuperSecurePassword456! \
  --type=BUILT_IN \
  --superuser=true

# Create service account user for application access
gcloud alpha alloydb users create app-service-account@my-project.iam.gserviceaccount.com \
  --cluster=app-cluster \
  --region=us-east1 \
  --type=IAM_BASED \
  --db-roles=pg_read_all_data,pg_write_all_data \
  --superuser=false

# List all users in a cluster
gcloud alpha alloydb users list \
  --cluster=production-cluster \
  --region=us-central1

# Output:
# NAME                    USER_TYPE    SUPERUSER
# db-admin                BUILT_IN     False
# superadmin              BUILT_IN     True
# analyst-group@...       IAM_BASED    False
```

## Cloud Bigtable: Cluster Scaling and Autoscaling

Configure Bigtable cluster compute capacity with manual or automatic scaling.

```bash
# Update cluster to fixed node count
gcloud alpha bigtable clusters update my-cluster \
  --instance=production-instance \
  --num-nodes=10

# Expected output:
# Updating cluster [my-cluster] in instance [production-instance]...done.

# Enable autoscaling with CPU and storage targets
gcloud alpha bigtable clusters update analytics-cluster \
  --instance=analytics-instance \
  --autoscaling-min-nodes=3 \
  --autoscaling-max-nodes=20 \
  --autoscaling-cpu-target=70 \
  --autoscaling-storage-target=4096

# Configure autoscaling with CPU target only
gcloud alpha bigtable clusters update web-cluster \
  --instance=web-instance \
  --autoscaling-min-nodes=5 \
  --autoscaling-max-nodes=15 \
  --autoscaling-cpu-target=60

# Switch from autoscaling back to fixed node count
gcloud alpha bigtable clusters update my-cluster \
  --instance=production-instance \
  --num-nodes=8

# Update cluster in specific zone
gcloud alpha bigtable clusters update regional-cluster \
  --instance=multi-region-instance \
  --zone=us-central1-a \
  --num-nodes=12

# Describe cluster to verify configuration
gcloud alpha bigtable clusters describe my-cluster \
  --instance=production-instance \
  --format=json

# Expected output:
# {
#   "name": "projects/my-project/instances/production-instance/clusters/my-cluster",
#   "location": "projects/my-project/locations/us-central1-a",
#   "state": "READY",
#   "serveNodes": 10,
#   "defaultStorageType": "SSD"
# }
```

## Cloud Batch: Job and Task Management

List and monitor tasks within Batch jobs for distributed workload processing.

```bash
# List all tasks for a specific job
gcloud alpha batch tasks list \
  --job=my-batch-job \
  --location=us-central1

# Expected output:
# NAME    STATUS    START_TIME            END_TIME
# task-0  SUCCEEDED 2024-01-15T10:00:00Z  2024-01-15T10:15:00Z
# task-1  SUCCEEDED 2024-01-15T10:00:00Z  2024-01-15T10:18:00Z
# task-2  RUNNING   2024-01-15T10:00:00Z  -

# List tasks with filtering by status
gcloud alpha batch tasks list \
  --job=projects/my-project/locations/us-central1/jobs/data-processing \
  --filter="status.state=SUCCEEDED" \
  --limit=50

# List failed tasks with full resource names
gcloud alpha batch tasks list \
  --job=my-batch-job \
  --location=us-central1 \
  --filter="status.state=FAILED" \
  --format="table(name, status.statusEvents[0].description)"

# Get task details as JSON for scripting
gcloud alpha batch tasks list \
  --job=my-batch-job \
  --location=us-central1 \
  --format=json | jq '.[].status.state'

# Output:
# "SUCCEEDED"
# "SUCCEEDED"
# "RUNNING"
# "FAILED"

# List tasks sorted by start time
gcloud alpha batch tasks list \
  --job=my-batch-job \
  --location=us-central1 \
  --sort-by=status.runDuration

# Describe resource allowance for job capacity planning
gcloud alpha batch resource-allowances describe my-allowance \
  --location=us-central1

# Expected output:
# name: projects/123/locations/us-central1/resourceAllowances/my-allowance
# usageResourceAllowance:
#   limit:
#     cpu: '1000'
#     memory: 2000Gi
#   usage:
#     cpu: '450'
#     memory: 890Gi
```

## Apigee: Organization Provisioning

Provision Apigee API management infrastructure with network peering.

```bash
# Step 1: Create VPC network for Apigee peering
gcloud compute networks create apigee-network \
  --bgp-routing-mode=global \
  --description="Network for Apigee X instance"

# Step 2: Allocate IP address range for VPC peering
gcloud compute addresses create apigee-range \
  --global \
  --prefix-length=16 \
  --network=apigee-network \
  --purpose=VPC_PEERING \
  --description="Peering range for Apigee"

# Step 3: Establish VPC peering connection
gcloud services vpc-peerings connect \
  --service=servicenetworking.googleapis.com \
  --network=apigee-network \
  --ranges=apigee-range

# Step 4: Provision Apigee organization (takes 10-60 minutes)
gcloud alpha apigee organizations provision \
  --authorized-network=projects/my-project/global/networks/apigee-network \
  --project=my-project \
  --analytics-region=us-central1 \
  --runtime-location=us-central1

# Expected output:
# Provisioning Apigee organization [my-project]...
# This operation may take up to 1 hour to complete.
# ...
# Organization [my-project] provisioned successfully.

# Provision asynchronously and get operation name
OPERATION=$(gcloud alpha apigee organizations provision \
  --authorized-network=projects/my-project/global/networks/apigee-network \
  --runtime-location=us-east1 \
  --analytics-region=us-east1 \
  --async \
  --format="value(name)")

# Monitor provisioning operation
gcloud alpha apigee operations describe $OPERATION

# Verify organization status
gcloud alpha apigee organizations describe --format=json

# Expected output (when complete):
# {
#   "name": "organizations/my-project",
#   "displayName": "my-project",
#   "state": "ACTIVE",
#   "analyticsRegion": "us-central1",
#   "runtimeType": "CLOUD",
#   "authorizedNetwork": "projects/my-project/global/networks/apigee-network"
# }
```

## Output Formatting and Filtering

Customize command output for scripts, reporting, and data processing.

```bash
# Default table output
gcloud compute instances list

# JSON output for programmatic parsing
gcloud compute instances list --format=json

# YAML output
gcloud compute instances list --format=yaml

# Custom table with specific columns
gcloud compute instances list \
  --format="table(name, zone.basename(), status, networkInterfaces[0].networkIP:label=INTERNAL_IP)"

# Expected output:
# NAME         ZONE          STATUS   INTERNAL_IP
# instance-1   us-central1-a RUNNING  10.128.0.2
# instance-2   us-east1-b    RUNNING  10.142.0.3

# Extract specific values (newline-separated)
gcloud compute instances list \
  --format="value(name, zone)"

# Output:
# instance-1 us-central1-a
# instance-2 us-east1-b

# CSV output for spreadsheet import
gcloud compute instances list \
  --format=csv

# Get resource URIs only
gcloud compute instances list --uri

# Filter by field values
gcloud compute instances list \
  --filter="zone:us-central1* AND status:RUNNING"

# Complex filter with AND/OR logic
gcloud compute instances list \
  --filter="machineType:n1-standard-1 OR machineType:n1-standard-2" \
  --filter="labels.env=production"

# Filter and limit results
gcloud compute instances list \
  --filter="labels.team=engineering" \
  --limit=10 \
  --sort-by=creationTimestamp

# Flatten nested fields
gcloud compute instances list \
  --flatten="networkInterfaces[]" \
  --format="table(name, networkInterfaces.networkIP)"

# Use jq for advanced JSON processing
gcloud compute instances list --format=json | \
  jq '[.[] | {name: .name, ip: .networkInterfaces[0].networkIP}]'
```

## Advanced Usage: Flags Files and Scripting

Manage complex commands with YAML/JSON configuration files for repeatability.

```bash
# Create flags file for complex command
cat > deploy-flags.yaml << 'EOF'
--project: my-project
--region: us-central1
--format: json
--quiet: true
--labels:
  environment: production
  team: platform
  cost-center: eng-001
EOF

# Use flags file (combines with command-line flags)
gcloud compute instances create my-instance \
  --flags-file=deploy-flags.yaml \
  --machine-type=n1-standard-4 \
  --image-family=debian-11 \
  --image-project=debian-cloud

# Scripting with error handling
#!/bin/bash
set -euo pipefail

PROJECT_ID="my-project"
REGION="us-central1"

# Create instance and capture output
if INSTANCE_OUTPUT=$(gcloud compute instances create my-instance \
  --project="$PROJECT_ID" \
  --zone="${REGION}-a" \
  --machine-type=n1-standard-2 \
  --format=json 2>&1); then

  INSTANCE_IP=$(echo "$INSTANCE_OUTPUT" | jq -r '.[0].networkInterfaces[0].networkIP')
  echo "Instance created with IP: $INSTANCE_IP"
else
  echo "Failed to create instance: $INSTANCE_OUTPUT" >&2
  exit 1
fi

# Batch operations with parallel execution
ZONES=("us-central1-a" "us-central1-b" "us-east1-a")
for zone in "${ZONES[@]}"; do
  (
    gcloud compute instances create "instance-${zone}" \
      --zone="$zone" \
      --machine-type=e2-micro \
      --async
  ) &
done
wait

# Retry logic for transient failures
MAX_RETRIES=3
RETRY_COUNT=0

while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
  if gcloud compute instances start my-instance --zone=us-central1-a --quiet; then
    echo "Instance started successfully"
    break
  else
    RETRY_COUNT=$((RETRY_COUNT + 1))
    echo "Retry $RETRY_COUNT of $MAX_RETRIES..."
    sleep 5
  fi
done
```

## Summary

The gcloud CLI is the comprehensive command-line interface for managing all aspects of Google Cloud Platform infrastructure and services. Primary use cases include infrastructure provisioning and management (Compute Engine VMs, GKE clusters, Cloud Storage buckets), AI/ML model deployment and prediction (Vertex AI endpoints, AI Platform training jobs), security and access control (IAM policies, Access Context Manager bindings, VPC Service Controls), database administration (Cloud SQL, AlloyDB, Bigtable cluster management), and API/application management (Apigee provisioning, Cloud Functions deployment).

Integration patterns emphasize service account impersonation for automated workflows, configuration management with named configurations for multi-environment deployments, output format customization (JSON/YAML) for programmatic parsing, filtering and pagination for large-scale resource queries, flags files for complex repeatable operations, and async operation handling for long-running tasks. The CLI's consistent command structure, comprehensive global flags, and extensive filtering capabilities make it suitable for both interactive use and CI/CD pipeline automation across heterogeneous Google Cloud architectures.
