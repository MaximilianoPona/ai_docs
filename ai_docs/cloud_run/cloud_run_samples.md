# Cloud Run Samples

This repository contains sample applications and demonstrations for Google Cloud Run, a fully managed serverless platform for deploying and scaling containerized applications. The samples showcase various Cloud Run capabilities including HTTP services, batch jobs, GPU-accelerated workloads, multi-container deployments, VPC networking, volume mounting, and CI/CD testing infrastructure.

The repository provides practical, production-ready code examples across multiple languages (Go, Python, Bash) and use cases. Each sample includes complete Docker configurations, deployment scripts, and Cloud Build integration for automated testing. These samples serve as reference implementations for Cloud Run best practices including security patterns, resource optimization, container networking, and integration with other Google Cloud services like Secret Manager, Cloud Storage, and Pub/Sub.

## Shell Script as HTTP Service

HTTP service that wraps shell scripts and makes them accessible via web endpoints using Go.

```go
// invoke.go
package main

import (
	"log"
	"net/http"
	"os"
	"os/exec"
)

func main() {
	http.HandleFunc("/", scriptHandler)

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
		log.Printf("Defaulting to port %s", port)
	}

	log.Printf("Listening on port %s", port)
	if err := http.ListenAndServe(":"+port, nil); err != nil {
		log.Fatal(err)
	}
}

func scriptHandler(w http.ResponseWriter, r *http.Request) {
	cmd := exec.CommandContext(r.Context(), "/bin/bash", "script.sh")
	cmd.Stderr = os.Stderr
	out, err := cmd.Output()
	if err != nil {
		w.WriteHeader(500)
	}
	w.Write(out)
}
```

```bash
# script.sh
#!/bin/bash
set -e
echo "Hello ${NAME:-World}!"
```

```dockerfile
# Dockerfile - Multi-stage build
FROM golang:1.25-trixie as builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY invoke.go ./
RUN go build -mod=readonly -v -o server

FROM debian:trixie-slim
RUN set -x && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    --no-install-recommends ca-certificates && \
    rm -rf /var/lib/apt/lists/*
WORKDIR /
COPY --from=builder /app/server /app/server
COPY script.sh ./
CMD ["/app/server"]
```

```bash
# Build and deploy
gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld-shell
gcloud run deploy helloworld-shell \
  --image gcr.io/PROJECT_ID/helloworld-shell \
  --set-env-vars NAME="Cloud Run" \
  --platform managed
```

## Batch Jobs with Retry Logic

Cloud Run Job for batch processing with configurable failure rates and automatic retry handling.

```bash
#!/bin/bash
# jobs-shell/script.sh
set -euo pipefail

CLOUD_RUN_TASK_INDEX=${CLOUD_RUN_TASK_INDEX:=0}
CLOUD_RUN_TASK_ATTEMPT=${CLOUD_RUN_TASK_ATTEMPT:=0}

echo "Starting Task #${CLOUD_RUN_TASK_INDEX}, Attempt #${CLOUD_RUN_TASK_ATTEMPT}..."

# Parse and validate inputs
SLEEP_MS=$(printf '%.1f' "${SLEEP_MS:=0}")
FAIL_RATE=$(printf '%.1f' "${FAIL_RATE:=0}")

# Simulate work with sleep
SLEEP_SEC=$(echo print\("${SLEEP_MS}"/1000\) | perl)
sleep "$SLEEP_SEC"

# Random failure based on FAIL_RATE
FAIL_RATE_INT=$(echo print\("int(${FAIL_RATE:=0}*100"\)\) | perl)
RAND=$(( RANDOM % 100))

if (( RAND < FAIL_RATE_INT )); then
    echo "Task #${CLOUD_RUN_TASK_INDEX}, Attempt #${CLOUD_RUN_TASK_ATTEMPT} failed."
    exit 1
else
    echo "Completed Task #${CLOUD_RUN_TASK_INDEX}."
fi
```

```dockerfile
# Dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y perl && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY script.sh .
RUN chmod +x script.sh
ENTRYPOINT ["./script.sh"]
```

```bash
# Deploy job with retry configuration
gcloud run jobs create jobs-shell \
  --image gcr.io/PROJECT_ID/jobs-shell \
  --set-env-vars FAIL_RATE=0.4,SLEEP_MS=5000 \
  --max-retries 10 \
  --tasks 5 \
  --region us-central1

# Execute job
gcloud run jobs execute jobs-shell

# Monitor execution
gcloud run jobs executions describe JOB_EXECUTION_NAME
```

## GPU-Accelerated Video Transcoding

Cloud Run Job using NVIDIA NVENC for hardware-accelerated video encoding with FFmpeg.

```bash
#!/bin/bash
# jobs-video-encoding/entrypoint.sh

# Verify NVIDIA environment
echo "Checking NVIDIA environment..."
nvidia-smi
retVal=$?
if [ $retVal -ne 0 ]; then
    echo "‼️ Error: nvidia-smi not available: exit code $retVal"
    exit $retVal
fi

nvcc --version
retVal=$?
if [ $retVal -ne 0 ]; then
    echo "‼️ nvcc not available: exit code $retVal"
    exit $retVal
fi

# Parse arguments: input, output, ffmpeg args
INPUT_FILE=$1
OUTPUT_FILE=$2
shift 2
FFMPEG_ARGS=("$@")

# Build paths
INPUT_PATH=/inputs/$INPUT_FILE
OUTPUT_PATH=/outputs/$OUTPUT_FILE
WORK_DIR="/tmp/transcode"
LOCAL_INPUT="${WORK_DIR}/${INPUT_FILE}"
LOCAL_OUTPUT="${WORK_DIR}/${OUTPUT_FILE}"

# Copy to local temp
cp "$INPUT_PATH" "$LOCAL_INPUT"

echo "Starting transcode job"
echo "Source: ${INPUT_PATH}"
echo "Target: ${OUTPUT_PATH}"

# Extract video metadata
VIDEO_INFO=$(ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height,r_frame_rate,nb_frames,duration \
  -of json "${LOCAL_INPUT}")

VIDEO_WIDTH=$(echo "$VIDEO_INFO" | python3 -c "import sys, json; data=json.load(sys.stdin); print(data['streams'][0].get('width', 'unknown'))")
VIDEO_HEIGHT=$(echo "$VIDEO_INFO" | python3 -c "import sys, json; data=json.load(sys.stdin); print(data['streams'][0].get('height', 'unknown'))")
DURATION=$(echo "$VIDEO_INFO" | python3 -c "import sys, json; data=json.load(sys.stdin); print(data['streams'][0].get('duration', 'unknown'))")

echo "Input: ${VIDEO_WIDTH}x${VIDEO_HEIGHT}, Duration: ${DURATION}s"

# GPU-accelerated transcoding with NVENC
TRANSCODE_START=$(date +%s.%N)

ffmpeg -y -c:v h264_cuvid -i "${LOCAL_INPUT}" \
  "${FFMPEG_ARGS[@]}" -preset p7 "${LOCAL_OUTPUT}" 2>&1 | tee /tmp/ffmpeg.log

FFMPEG_EXIT_CODE=${PIPESTATUS[0]}
TRANSCODE_END=$(date +%s.%N)
TRANSCODE_DURATION=$(echo "$TRANSCODE_END - $TRANSCODE_START" | bc)

if [ "$FFMPEG_EXIT_CODE" -ne 0 ]; then
    echo "ERROR: FFmpeg failed with exit code ${FFMPEG_EXIT_CODE}"
    exit "$FFMPEG_EXIT_CODE"
fi

echo "Transcoding complete in ${TRANSCODE_DURATION}s"
echo "Output size: $(du -h ${LOCAL_OUTPUT} | cut -f1)"

# Copy to output location
cp "$LOCAL_OUTPUT" "$OUTPUT_PATH"
exit 0
```

```dockerfile
# Multi-stage build with CUDA and FFmpeg
FROM nvidia/cuda:12.1.0-devel-ubuntu22.04 as builder
RUN apt-get update && apt-get install -y \
    build-essential yasm nasm pkg-config \
    libx264-dev libx265-dev && \
    rm -rf /var/lib/apt/lists/*
# Build FFmpeg with NVENC support
# ... (compilation steps omitted for brevity)

FROM nvidia/cuda:12.1.0-runtime-ubuntu22.04
COPY --from=builder /usr/local/bin/ffmpeg /usr/local/bin/
COPY --from=builder /usr/local/bin/ffprobe /usr/local/bin/
COPY entrypoint.sh /app/
RUN chmod +x /app/entrypoint.sh
ENTRYPOINT ["/app/entrypoint.sh"]
```

```bash
# Deploy GPU job
gcloud run jobs create video-transcode \
  --image gcr.io/PROJECT_ID/jobs-video-encoding \
  --gpu 1 \
  --gpu-type nvidia-l4 \
  --memory 8Gi \
  --cpu 4 \
  --task-timeout 3600 \
  --region us-central1

# Execute with Cloud Storage mounts
gcloud run jobs execute video-transcode \
  --args="input.mp4,output.mp4,-vcodec,h264_nvenc,-b:v,5M"
```

## VPC Service-to-Service Authentication

Python service demonstrating VPC egress/ingress controls with authenticated service-to-service requests.

```python
# vpc-sample/main.py
import os
import urllib

import google.auth.transport.requests
import google.oauth2.id_token


def get_hello_world(request):
    """Makes authenticated request to internal Cloud Run service via VPC"""
    try:
        url = os.environ.get("URL")
        req = urllib.request.Request(url)

        # Mint ID token for service-to-service auth
        auth_req = google.auth.transport.requests.Request()
        id_token = google.oauth2.id_token.fetch_id_token(auth_req, url)
        req.add_header("Authorization", f"Bearer {id_token}")

        response = urllib.request.urlopen(req)
        return response.read()

    except Exception as e:
        print(e)
        return str(e)


def hello_world(request):
    """Target function with internal-only ingress"""
    return "Hello World!"
```

```dockerfile
# Dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY main.py .
CMD exec functions-framework --target=get_hello_world
```

```bash
# requirements.txt
functions-framework==3.5.0
google-auth==2.28.0
```

```bash
# Deploy internal service with VPC ingress restriction
gcloud run deploy restricted-function \
  --source . \
  --function hello_world \
  --ingress internal \
  --region us-central1

# Deploy calling service with VPC egress
gcloud run deploy vpc-caller \
  --source . \
  --function get_hello_world \
  --set-env-vars URL=https://restricted-function-xyz.run.app \
  --vpc-egress all-traffic \
  --vpc-connector projects/PROJECT_ID/locations/us-central1/connectors/vpc-conn \
  --region us-central1

# Test authenticated request
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  https://vpc-caller-xyz.run.app
```

## Cloud Run Reporting Service

Go service using gcloud CLI to generate Cloud Run service reports and store them in Cloud Storage.

```go
// gcloud-report/invoke.go
package main

import (
	"log"
	"net/http"
	"os"
	"os/exec"
	"regexp"
)

func main() {
	http.HandleFunc("/", scriptHandler)

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
		log.Printf("defaulting to port %s", port)
	}

	log.Printf("listening on port %s", port)
	if err := http.ListenAndServe(":"+port, nil); err != nil {
		log.Fatal(err)
	}
}

func scriptHandler(w http.ResponseWriter, r *http.Request) {
	search := r.URL.Query().Get("search")

	// Validate input - only allow alphanumeric and hyphens
	re := regexp.MustCompile(`^[a-z]+[a-z0-9\-]*$`)
	if !re.MatchString(search) {
		log.Printf("invalid search criteria %q, using default", search)
		search = "."
	}

	cmd := exec.CommandContext(r.Context(), "/bin/bash", "script.sh", search)
	cmd.Stderr = os.Stderr
	out, err := cmd.Output()
	if err != nil {
		log.Printf("Command.Output: %v", err)
		http.Error(w, http.StatusText(http.StatusInternalServerError),
			http.StatusInternalServerError)
		return
	}
	w.Write(out)
}
```

```bash
#!/usr/bin/env bash
# gcloud-report/script.sh
set -eo pipefail

# Validate required environment variables
requireEnv() {
  test "${!1}" || (echo "gcloud-report: '$1' not found" >&2 && exit 1)
}
requireEnv GCLOUD_REPORT_BUCKET

# Configure report format
search=${1:-'.'}
limits='spec.template.spec.containers.resources.limits.flatten("", "", " ")'
format='table[box, title="Cloud Run Services"](name,status.url,metadata.annotations.[serving.knative.dev/creator],'${limits}')'

# Generate unique object name with timestamp
obj="gs://${GCLOUD_REPORT_BUCKET}/report-${search}-$(date +%s).txt"

# Generate report and upload to Cloud Storage
gcloud run services list \
  --format "${format}" \
  --filter "metadata.name~${search}" | gsutil -q cp -J - "${obj}"

echo "gcloud-report: wrote to ${obj}" >&2
echo "Wrote report to ${obj}"
```

```dockerfile
# Dockerfile with Google Cloud SDK
FROM gcr.io/google.com/cloudsdktool/cloud-sdk:slim as builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY invoke.go ./
RUN go build -v -o server

FROM gcr.io/google.com/cloudsdktool/cloud-sdk:slim
WORKDIR /app
COPY --from=builder /app/server .
COPY script.sh .
CMD ["./server"]
```

```bash
# Deploy with service account for Cloud Storage access
gcloud run deploy gcloud-report \
  --image gcr.io/PROJECT_ID/gcloud-report \
  --set-env-vars GCLOUD_REPORT_BUCKET=my-reports-bucket \
  --service-account cloud-run-reporter@PROJECT_ID.iam.gserviceaccount.com \
  --region us-central1

# Generate report
curl "https://gcloud-report-xyz.run.app?search=production"
```

## Volume Mount Verification

Go service for testing Cloud Run volume mounts and file system access.

```go
// volume-checker/main.go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func main() {
	http.HandleFunc("/", healthCheck)
	http.HandleFunc("/read", readDir)

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
		log.Printf("defaulting to port %s", port)
	}

	if err := http.ListenAndServe(":"+port, nil); err != nil {
		log.Fatal(err)
	}
}

func healthCheck(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Hello World!")
}

func readDir(w http.ResponseWriter, r *http.Request) {
	dir := r.URL.Query().Get("dir")

	if dir == "" {
		w.WriteHeader(http.StatusOK)
		fmt.Fprintf(w, "'dir' query parameter not set")
		return
	}

	entries, err := os.ReadDir(dir)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		fmt.Fprintf(w, "error reading directory '%s' within container: %v", dir, err)
		return
	}

	fmt.Fprintf(w, "entries in '%s':\n\n", dir)

	for _, e := range entries {
		fmt.Fprintf(w, "%s/%s\n", dir, e.Name())
	}
}
```

```dockerfile
FROM golang:1.20-buster as builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY main.go ./
RUN go build -v -o server

FROM debian:buster-slim
COPY --from=builder /app/server /app/server
CMD ["/app/server"]
```

```bash
# Deploy with Cloud Storage volume mount
gcloud run deploy volume-checker \
  --image gcr.io/PROJECT_ID/volume-checker \
  --execution-environment gen2 \
  --add-volume name=gcs-bucket,type=cloud-storage,bucket=my-bucket \
  --add-volume-mount volume=gcs-bucket,mount-path=/mnt/gcs \
  --region us-central1

# Verify mount
curl "https://volume-checker-xyz.run.app/read?dir=/mnt/gcs"
# Output: entries in '/mnt/gcs':
# /mnt/gcs/file1.txt
# /mnt/gcs/file2.txt
```

## Multi-Container Deployment

Multi-container Cloud Run service with nginx ingress and sidecar container using Secret Manager for configuration.

```yaml
# multi-container/hello-nginx-sample/service.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello-nginx
  labels:
    cloud.googleapis.com/location: us-central1
  annotations:
    run.googleapis.com/launch-stage: BETA
    run.googleapis.com/ingress: all
spec:
  template:
    metadata:
      annotations:
        # Container startup ordering
        run.googleapis.com/container-dependencies: "{nginx: [hello]}"
    spec:
      containers:
        # Ingress container
        - image: nginx
          name: nginx
          ports:
            - name: http1
              containerPort: 8080
          resources:
            limits:
              cpu: 500m
              memory: 256Mi
          volumeMounts:
            - name: nginx-conf-secret
              readOnly: true
              mountPath: /etc/nginx/conf.d/
          startupProbe:
            timeoutSeconds: 240
            periodSeconds: 240
            failureThreshold: 1
            tcpSocket:
              port: 8080

        # Sidecar container
        - image: us-docker.pkg.dev/cloudrun/container/hello
          name: hello
          env:
            - name: PORT
              value: "8888"
          resources:
            limits:
              cpu: 1000m
              memory: 512Mi
          startupProbe:
            timeoutSeconds: 240
            periodSeconds: 240
            failureThreshold: 1
            tcpSocket:
              port: 8888

      # Secret Manager volume
      volumes:
        - name: nginx-conf-secret
          secret:
            secretName: nginx_config
            items:
              - key: latest
                path: default.conf
```

```nginx
# nginx.conf
server {
    listen 8080;
    server_name _;
    gzip on;

    location / {
        # Proxy to sidecar container
        proxy_pass http://127.0.0.1:8888;
    }
}
```

```bash
# Create secret with nginx configuration
gcloud secrets create nginx_config \
  --replication-policy="automatic" \
  --data-file="./nginx.conf"

# Grant access to compute service account
export PROJECT_NUMBER=$(gcloud projects describe $(gcloud config get-value project) \
  --format='value(projectNumber)')
gcloud secrets add-iam-policy-binding nginx_config \
  --member=serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
  --role='roles/secretmanager.secretAccessor'

# Deploy multi-container service
export MC_SERVICE_NAME=hello-nginx
export REGION=us-central1
sed -i -e s/MC_SERVICE_NAME/${MC_SERVICE_NAME}/g -e s/REGION/${REGION}/g service.yaml
gcloud run services replace service.yaml

# Test authenticated request
curl --header "Authorization: Bearer $(gcloud auth print-identity-token)" \
  https://hello-nginx-xyz.run.app

# Allow public access
gcloud run services add-iam-policy-binding hello-nginx \
  --member="allUsers" \
  --role="roles/run.invoker"
```

## CI/CD Testing Infrastructure

Cloud Build templates and automation for testing Cloud Run samples with flaky test detection.

```yaml
# testing/cloudbuild-templates/cloud-run-template.cloudbuild.yaml
steps:
  # Build container
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/$_SERVICE:$SHORT_SHA', '.']

  # Push to registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/$_SERVICE:$SHORT_SHA']

  # Deploy to Cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - '$_SERVICE'
      - '--image=gcr.io/$PROJECT_ID/$_SERVICE:$SHORT_SHA'
      - '--region=$_REGION'
      - '--platform=managed'
      - '--no-allow-unauthenticated'

  # Run integration tests
  - name: 'gcr.io/cloud-builders/gcloud'
    entrypoint: '/bin/bash'
    args:
      - '-c'
      - |
        set -e
        SERVICE_URL=$(gcloud run services describe $_SERVICE \
          --region $_REGION --format 'value(status.url)')
        ID_TOKEN=$(gcloud auth print-identity-token)
        curl -f -H "Authorization: Bearer $ID_TOKEN" $SERVICE_URL

  # Cleanup
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['run', 'services', 'delete', '$_SERVICE', '--region=$_REGION', '--quiet']

substitutions:
  _SERVICE: 'test-service'
  _REGION: 'us-central1'

timeout: '1200s'
```

```python
# testing/flakybot-function/main.py
import base64
import json
import os
from datetime import datetime

from google.cloud import storage
import junit_xml
import requests


def process_build_result(event, context):
    """Processes Cloud Build Pub/Sub messages and sends to Flakybot"""

    # Decode Pub/Sub message
    pubsub_message = base64.b64decode(event['data']).decode('utf-8')
    build = json.loads(pubsub_message)

    # Extract build details
    build_id = build['id']
    status = build['status']
    project_id = build['projectId']

    if status != 'FAILURE':
        print(f"Build {build_id} status: {status}, skipping")
        return

    # Get build logs
    storage_client = storage.Client()
    bucket = storage_client.bucket('cloud-build-logs')
    log_path = f"log-{build_id}.txt"
    blob = bucket.blob(log_path)
    logs = blob.download_as_text()

    # Parse test results
    test_cases = parse_test_results(logs)

    # Convert to JUnit XML
    test_suite = junit_xml.TestSuite("Cloud Run Tests", test_cases)
    xml_output = junit_xml.TestSuite.to_xml_string([test_suite])

    # Send to Flakybot
    flakybot_url = os.environ['FLAKYBOT_URL']
    response = requests.post(
        flakybot_url,
        headers={'Content-Type': 'application/xml'},
        data=xml_output
    )

    print(f"Sent results to Flakybot: {response.status_code}")


def parse_test_results(logs):
    """Parse test results from build logs"""
    test_cases = []
    # Implementation varies by test framework
    return test_cases
```

```bash
# testing/runner.sh
#!/bin/bash
set -euo pipefail

# Test a Cloud Run sample
SERVICE=$1
REGION=${2:-us-central1}

echo "Testing service: $SERVICE in region: $REGION"

# Build and deploy
gcloud builds submit --config=tests.cloudbuild.yaml \
  --substitutions=_SERVICE=$SERVICE,_REGION=$REGION

# Run integration tests
SERVICE_URL=$(gcloud run services describe $SERVICE \
  --region $REGION --format 'value(status.url)')

ID_TOKEN=$(gcloud auth print-identity-token)

# Test with retries
for i in {1..3}; do
  if curl -f -H "Authorization: Bearer $ID_TOKEN" $SERVICE_URL; then
    echo "Test passed"
    exit 0
  fi
  echo "Attempt $i failed, retrying..."
  sleep 5
done

echo "All test attempts failed"
exit 1
```

This repository provides comprehensive Cloud Run examples demonstrating serverless HTTP services, batch processing jobs, and advanced deployment patterns. The samples are production-ready and include best practices for container optimization, security, networking, and observability. Key use cases include wrapping existing shell scripts as HTTP services, running GPU-accelerated workloads, implementing service-to-service authentication within VPCs, deploying multi-container microservices with sidecars, and testing volume mounts for persistent storage.

Integration patterns include using Cloud Build for CI/CD automation, Secret Manager for configuration management, Cloud Storage for data persistence, VPC connectors for private networking, and Pub/Sub for event-driven architectures. The testing infrastructure enables automated validation of all samples with flaky test detection and reporting. All samples support both authenticated and public access modes, demonstrate proper error handling, and follow container best practices with multi-stage builds for minimal image sizes.
