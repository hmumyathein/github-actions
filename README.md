# GitHub Actions for Docker and ECR

This repository contains reusable GitHub workflows for Docker operations with Amazon ECR (Elastic Container Registry).

## Available Workflows

1. **Docker Build and Push to ECR** - Build and push Docker images to ECR
2. **Docker Meta Publish** - Create and push Docker manifests with metadata
3. **ECR Tag Validation** - Validate that specific tags exist in ECR repositories
4. **ECS Deploy** - Deploy applications to Amazon ECS services

## Usage

### Basic Usage

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    uses: hmumyathein/github-actions/.github/workflows/docker-build-push-ecr.yml@main
    with:
      role_to_assume: arn:aws:iam::123456789012:role/GitHubActionsRole
      ecr_uri: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app
      ecr_registries: 123456789012
      docker_meta_tags: type=ref,event=branch
      docker_build_platforms: linux/amd64,linux/arm64
```

### Advanced Usage

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: hmumyathein/github-actions/.github/workflows/docker-build-push-ecr.yml@main
    with:
      # Required inputs
      role_to_assume: arn:aws:iam::123456789012:role/GitHubActionsRole
      ecr_uri: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app
      ecr_registries: 123456789012
      docker_meta_tags: type=ref,event=branch
      docker_build_platforms: linux/amd64,linux/arm64
      
      # Optional inputs
      aws_region: us-east-1
      docker_build_context: ./src
      dockerfile_path: ./src/Dockerfile
      docker_build_args: |
        BUILD_VERSION=${{ github.sha }}
        NODE_ENV=production
      docker_build_secrets: |
        GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }}
      docker_build_secret_files: |
        npmrc=./src/.npmrc
      ecr_registries: 123456789012,987654321098
      emulation_required: true
      include_checkout: true
      buildx_image_uri: moby/buildkit:latest
      qemu_image_uri: tonistiigi/binfmt:latest
      trigger_push: true
      additional_tags: latest,stable
```

## Inputs

### Required

| Input | Type | Description |
|-------|------|-------------|
| `role_to_assume` | string | ARN of the role that is authorized to publish the image |
| `ecr_uri` | string | ECR URI to push image to |
| `ecr_registries` | string | AWS account IDs included with ECR login (comma separated) |
| `docker_meta_tags` | string | Tags argument to docker meta |
| `docker_build_platforms` | string | Target architectures for Docker build, e.g. linux/amd64 OR linux/arm64 |

### Optional

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `aws_region` | string | `ap-southeast-1` | AWS Region |
| `docker_build_context` | string | `${{ github.workspace }}` | Docker build context |
| `docker_build_args` | string | - | List of build-time variables to docker build |
| `docker_build_secrets` | string | - | List of secrets to expose to the build |
| `docker_build_secret_files` | string | - | List of secret files to expose to the build |
| `dockerfile_path` | string | `"./Dockerfile"` | Path to Dockerfile |
| `ecr_registries` | string | **Required** | AWS account IDs included with ECR login (comma separated) |
| `emulation_required` | boolean | `false` | Whether to install QEMU to enable architecture emulation |
| `include_checkout` | boolean | `true` | Whether to run the checkout action |
| `buildx_image_uri` | string | `"moby/buildkit:master"` | Optionally override buildx image |
| `qemu_image_uri` | string | `"tonistiigi/binfmt:latest"` | Optionally override of qemu image |
| `trigger_push` | boolean | `true` | Whether to push image after build |
| `additional_tags` | string | `""` | Extra tags to be pushed |

## Outputs

| Output | Description |
|--------|-------------|
| `tags` | Image tags of the build image |

## Authentication

This workflow uses AWS IAM roles with OpenID Connect (OIDC) for authentication. No AWS access keys are required.

### Prerequisites

1. **AWS IAM OIDC Provider**: Configure an OIDC identity provider in your AWS account
2. **IAM Role**: Create an IAM role with the necessary ECR permissions
3. **GitHub Repository Settings**: Configure the OIDC trust relationship in your IAM role
4. **ECR Repository**: Create an ECR repository in your AWS account

### Required IAM Permissions

The IAM role should have the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:PutImage"
      ],
      "Resource": "*"
    }
  ]
}
```

## Examples

### Multi-platform Build

```yaml
jobs:
  build:
    uses: hmumyathein/github-actions/.github/workflows/docker-build-push-ecr.yml@main
    with:
      role_to_assume: arn:aws:iam::123456789012:role/GitHubActionsRole
      ecr_uri: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app
      ecr_registries: 123456789012
      docker_meta_tags: type=ref,event=branch
      docker_build_platforms: linux/amd64,linux/arm64
      emulation_required: true
```

### Build with Custom Context and Dockerfile

```yaml
jobs:
  build:
    uses: hmumyathein/github-actions/.github/workflows/docker-build-push-ecr.yml@main
    with:
      role_to_assume: arn:aws:iam::123456789012:role/GitHubActionsRole
      ecr_uri: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app
      ecr_registries: 123456789012
      docker_meta_tags: type=ref,event=branch
      docker_build_platforms: linux/amd64
      docker_build_context: ./app
      dockerfile_path: ./app/Dockerfile.prod
```

### Build Only (No Push)

```yaml
jobs:
  build:
    uses: hmumyathein/github-actions/.github/workflows/docker-build-push-ecr.yml@main
    with:
      role_to_assume: arn:aws:iam::123456789012:role/GitHubActionsRole
      ecr_uri: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app
      ecr_registries: 123456789012
      docker_meta_tags: type=ref,event=branch
      docker_build_platforms: linux/amd64
      trigger_push: false
```

## Docker Meta Publish Workflow

The Docker Meta Publish workflow creates and pushes Docker manifests with metadata to ECR.

### Usage

```yaml
jobs:
  publish-manifest:
    uses: hmumyathein/github-actions/.github/workflows/docker-meta-publish.yml@main
    with:
      role_to_assume: "arn:aws:iam::123456789012:role/GitHubActionsRole"
      ecr_uri: "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app"
      ecr_registries: "123456789012"
      docker_meta_tags: type=ref,event=branch
      amend_tags: "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest"
```

### Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `role_to_assume` | string | Yes | ARN of the role that is authorized to publish the image |
| `ecr_uri` | string | Yes | ECR URI to push image to |
| `ecr_registries` | string | Yes | AWS account IDs included with ECR login (comma separated) |
| `docker_meta_tags` | string | Yes | Tags argument to docker meta |
| `amend_tags` | string | Yes | Image tags to be linked in Docker manifest |
| `aws_region` | string | No | AWS Region (default: ap-southeast-1) |
| `buildx_image_uri` | string | No | Optionally override buildx image (default: moby/buildkit:master) |

## ECR Tag Validation Workflow

The ECR Tag Validation workflow validates that specific tags exist in ECR repositories.

### Usage

```yaml
jobs:
  validate-tags:
    uses: hmumyathein/github-actions/.github/workflows/ecr-tag-validation.yml@main
    with:
      role_to_assume: "arn:aws:iam::123456789012:role/GitHubActionsRole"
      registry: "123456789012"
      repo_name: "my-app"
      image_tag: "latest"  # Validate single tag
      # OR
      image_tags: "latest stable v1.0.0"  # Validate multiple tags
```

### Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `role_to_assume` | string | Yes | AWS IAM role to assume to perform ECR query |
| `registry` | string | Yes | AWS account ID used for ECR login |
| `repo_name` | string | Yes | Short name of the ECR repository |
| `image_tag` | string | No | The image/artifact tag that is to be validated |
| `image_tags` | string | No | The image/artifact tags to be validated, space separated |
| `aws_region` | string | No | AWS Region (default: ap-southeast-1) |

## ECS Deploy Workflow

The ECS Deploy workflow registers an Amazon ECS task definition and deploys it to an ECS service.

### Usage

```yaml
jobs:
  deploy-to-ecs:
    uses: hmumyathein/github-actions/.github/workflows/ecs-deploy.yml@main
    with:
      role_to_assume: "arn:aws:iam::123456789012:role/GitHubActionsRole"
      cluster_name: "my-cluster"
      service_name: "my-service"
      container_name: "my-container"
      image_uri: "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest"
      taskdef_path: "./task-definition.json"
      environment-variables: |
        NODE_ENV=production
        LOG_LEVEL=info
      wait_for_service_stability: true
```

### Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `role_to_assume` | string | Yes | AWS IAM role to assume to perform ECS deployment |
| `cluster_name` | string | Yes | ECS cluster name that service runs in |
| `service_name` | string | Yes | ECS service name to run the image |
| `container_name` | string | Yes | Container name for deploying image |
| `image_uri` | string | Yes | Full image URI |
| `taskdef_path` | string | No | Relative path to the ECS task definition |
| `taskdef_arn` | string | No | ARN of the ECS task definition |
| `taskdef_family` | string | No | Family name of the ECS task definition |
| `environment-variables` | string | No | Variables to add to the container (KEY=value format) |
| `wait_for_service_stability` | boolean | No | Whether to wait for ECS deployment to complete (default: true) |
| `inspect_taskdef_post_render` | boolean | No | Prints out the task definition to stdout (default: false) |
| `aws_region` | string | No | AWS Region (default: ap-southeast-1) |
| `codedeploy-application` | string | No | AWS CodeDeploy application name |
| `codedeploy-deployment-group` | string | No | AWS CodeDeploy deployment group name |
| `codedeploy-appspec` | string | No | Path to the AWS CodeDeploy AppSpec file |

### Outputs

| Output | Description |
|--------|-------------|
| `task-definition` | The rendered task definition |

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.