# GCP CI/CD Sample App

A lightweight Go application demonstrating CI/CD automation on Google Cloud Platform. This project showcases a complete pipeline from source code to containerized deployment across development and production environments.

## Overview

**Sample App** is a simple HTTP server built in Go that serves dynamically generated PNG images. It's designed to demonstrate best practices for containerization and automated deployment using Google Cloud Build.

### Features

- **Dual Endpoint HTTP Server**: Serves colored PNG images (blue and red) via HTTP
- **Multi-stage Docker Build**: Optimized image size using distroless base
- **GCP CI/CD Pipeline**: Automated build and deploy workflows for dev and prod environments
- **Environment-Specific Deployments**: Separate Cloud Build configurations for development and production

## Architecture

### Application Structure

```
sample-app/
├── main.go                 # Go HTTP server with image generation endpoints
├── Dockerfile             # Multi-stage build for optimized container image
├── cloudbuild.yaml        # Production CI/CD pipeline
├── cloudbuild-dev.yaml    # Development CI/CD pipeline
├── dev/                   # Development Kubernetes manifests
├── prod/                  # Production Kubernetes manifests
└── .github/               # GitHub workflows (optional)
```

### Technology Stack

- **Language**: Go 1.19.2
- **Containerization**: Docker (distroless base)
- **CI/CD**: Google Cloud Build
- **Orchestration**: Google Kubernetes Engine (GKE)
- **Registry**: Google Artifact Registry

## Getting Started

### Prerequisites

- Go 1.19 or later
- Docker
- Google Cloud SDK (`gcloud`)
- Access to a GCP project with Cloud Build and GKE enabled

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/FayssalObeid/sample-app.git
   cd sample-app
   ```

2. **Build locally**
   ```bash
   go build -o hello-app main.go
   ```

3. **Run the application**
   ```bash
   ./hello-app
   ```

4. **Test the endpoints**
   ```bash
   # Get blue image
   curl http://localhost:8080/blue -o blue.png
   
   # Get red image
   curl http://localhost:8080/red -o red.png
   ```

### Docker Build

```bash
docker build -t hello-app:latest .
docker run -p 8080:8080 hello-app:latest
```

## CI/CD Pipeline

This project uses **Google Cloud Build** to automate the build, test, and deployment process.

### Production Pipeline (`cloudbuild.yaml`)

The production pipeline:
1. **Compiles** the Go application
2. **Builds** a Docker image with versioning (v2.0)
3. **Pushes** the image to Google Artifact Registry
4. **Deploys** to the production namespace (`prod/deployment.yaml`) on GKE

**Trigger**: Merge to main/master branch
**Deployment Region**: us-central1-b
**Cluster**: hello-cluster

### Development Pipeline (`cloudbuild-dev.yaml`)

The development pipeline:
1. **Compiles** the Go application
2. **Builds** a Docker image (version as placeholder)
3. **Pushes** the image to Google Artifact Registry
4. **Deploys** to the development namespace (`dev/deployment.yaml`) on GKE

**Trigger**: Commits to feature/dev branches
**Deployment Region**: us-central1-b
**Cluster**: hello-cluster

### Configuration

Before running the pipelines, update the following in the Cloud Build configs:

```yaml
# Update artifact registry location
us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild

# Update cluster details
CLOUDSDK_COMPUTE_REGION=us-central1-b
CLOUDSDK_CONTAINER_CLUSTER=hello-cluster
```

## API Endpoints

### GET /blue
Returns a 100x100 PNG image with blue color (RGB: 0, 0, 255)

```bash
curl http://localhost:8080/blue -o blue.png
```

### GET /red
Returns a 100x100 PNG image with red color (RGB: 255, 0, 0)

```bash
curl http://localhost:8080/red -o red.png
```

## Building Blocks

### main.go

The application exposes two HTTP handlers:
- `blueHandler`: Generates and serves a blue PNG image
- `redHandler`: Generates and serves a red PNG image

Uses Go's standard `image` and `image/png` packages for image generation.

### Dockerfile

Multi-stage build for minimal image size:

**Stage 1 (Builder)**:
- Uses `golang:1.19.2` to compile the application
- Builds a statically-linked binary with `CGO_ENABLED=0`

**Stage 2 (Runtime)**:
- Uses `gcr.io/distroless/base-debian11` for minimal footprint
- Runs as non-root user (`nonroot:nonroot`)
- Exposes port 8080

## Deployment

### Kubernetes Manifests

The project includes deployment manifests in `dev/` and `prod/` directories:

- `deployment.yaml`: Kubernetes Deployment manifest
- Configure replicas, resource limits, and image references as needed

Example deployment structure:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  namespace: prod  # or dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - name: hello-app
        image: us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild:v2.0
        ports:
        - containerPort: 8080
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `PORT` | Server port (default: 8080) |
| `PROJECT_ID` | GCP Project ID (used in Cloud Build) |
| `GOPATH` | Go workspace path (Cloud Build) |

## License

Licensed under the Apache License, Version 2.0. See the LICENSE file in the source code headers for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Push to your branch
5. Open a pull request

## Support

For issues and feature requests, please open an issue on GitHub.

## References

- [Google Cloud Build Documentation](https://cloud.google.com/build/docs)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs)
- [Google Artifact Registry](https://cloud.google.com/artifact-registry/docs)
- [Go Image Package](https://golang.org/pkg/image/)
- [Distroless Container Images](https://github.com/GoogleContainerTools/distroless)
