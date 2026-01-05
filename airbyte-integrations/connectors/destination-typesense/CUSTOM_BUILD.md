# Building & Publishing Custom destination-typesense Connector

## Overview

This documents how to build and publish a custom version of the destination-typesense connector to AWS Public ECR for use in our Airbyte deployment.

**ECR Repository:** `public.ecr.aws/w7s0q8d5/destination-typesense`

## Prerequisites

- Docker with buildx support
- AWS CLI configured with permissions to push to Public ECR
- Python 3.10+ and Poetry (for local development/testing)

## Steps

### 1. Make Your Changes

Edit the connector code in this directory. Key files:
```
destination_typesense/
├── destination.py       # Main connector logic
├── writer.py            # Typesense writer
└── ...
pyproject.toml           # Dependencies (including airbyte-cdk version)
metadata.yaml            # Connector metadata & version
```

### 2. Bump the Version

Update `metadata.yaml`:
```yaml
dockerImageTag: 0.1.54  # Increment from current version
```

### 3. Build the Docker Image

Create a `Dockerfile` in this directory (if it doesn't exist):
```dockerfile
FROM airbyte/destination-typesense:latest

COPY . ./airbyte/integration_code
RUN pip install ./airbyte/integration_code
```

Build for `linux/amd64` architecture (required for K8s workers):
```bash
cd airbyte-integrations/connectors/destination-typesense

docker buildx build --platform linux/amd64 -t public.ecr.aws/w7s0q8d5/destination-typesense:0.1.54 .
```

### 4. Push to Public ECR

```bash
# Login to AWS Public ECR
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws

# Push the image
docker push public.ecr.aws/w7s0q8d5/destination-typesense:0.1.54
```

### 5. Update Airbyte to Use New Version

In the Airbyte UI:
1. Go to **Settings > Destinations**
2. Find the custom Typesense connector
3. Update the Docker image tag to the new version

## Local Testing

```bash
# Install dependencies
poetry install

# Run tests
poetry run pytest unit_tests/

# Test connector locally
poetry run python main.py spec
poetry run python main.py check --config secrets/config.json
```

## Related Links

- **Fork:** https://github.com/koodoslabs/airbyte
- **Upstream PR:** https://github.com/airbytehq/airbyte/pull/70420
