# Lambda Deploy App

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)
![AWS Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?logo=awslambda&logoColor=white)
![CI](https://github.com/pafful/lamdaDeployApp/actions/workflows/deploy.yaml/badge.svg)

## Executive Summary

Lambda Deploy App is a small Python AWS Lambda example with a GitHub Actions pipeline. The function accepts an event containing `input` and returns `World` when the value is `Hello`. The repository currently packages the function and its Python dependencies into a ZIP release artifact; Docker, Kubernetes, ECR, and production-grade infrastructure are documented as deployment extensions rather than existing resources.

> **Repository status:** this is a starter deployment project. There are no Dockerfiles, Kubernetes manifests, tests, infrastructure-as-code files, or HTTP API gateway resources in the current repository.

## Problem Statement

The project demonstrates a repeatable path from a small Python function to a versioned deployment artifact. It provides a minimal baseline for validating code, installing Lambda dependencies, packaging the function, and publishing the resulting ZIP through GitHub Releases.

## Architecture

The current flow is described in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md). In brief, a push triggers GitHub Actions, which lints the function, builds a dependency-inclusive ZIP, and publishes it as a release asset. AWS Lambda invocation is an intended deployment target; the workflow does not currently call AWS APIs.

## Repository Structure

```text
.
├── .github/
│   ├── dependabot.yaml
│   └── workflows/
│       ├── action.yml       # Self-hosted runner smoke test
│       └── deploy.yaml      # Lint, package, and GitHub Release workflow
├── function/
│   ├── lambda_function.py   # Lambda handler
│   └── requirements.txt     # Runtime dependencies
├── azure-pipelines.yml      # Starter Azure pipeline for main
├── azure-pipelines-1.yml   # Starter Azure pipeline for dev
├── azure-pipelines-2.yml   # Duplicate starter Azure pipeline for dev
├── buildspec.yaml           # AWS CodeBuild starter specification
├── docs/
└── README.MD
```

## Technologies

- Python 3.8 in the current CI workflow (new deployments should use a supported Lambda runtime).
- AWS Lambda and Boto3 as the intended AWS runtime.
- PyGithub 1.55 and Boto3 1.17.96.
- GitHub Actions, GitHub Releases, and Actions artifacts.
- Optional documented targets: Docker, Amazon ECR, and Kubernetes.
- Starter integrations: Azure Pipelines and AWS CodeBuild.

## Local Development

From the repository root:

```bash
python -m venv .venv
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
# macOS/Linux
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r function/requirements.txt
```

Invoke the handler without AWS:

```bash
python -c "from function.lambda_function import lambda_handler; print(lambda_handler({'input': 'Hello'}, None))"
```

Expected output: `World`. Install `flake8` and run `flake8 function --count --select=E9,F63,F7,F82` before opening a pull request. The current implementation raises a bare exception for unsupported input; see [docs/API.md](docs/API.md).

## Docker and Docker Compose

No Docker assets are committed yet. Lambda ZIP packaging is the current deployment model. A container image should only be added when the team chooses Lambda container images or a container platform. Recommended additions are a pinned base image, a non-root local runtime, a health check, and a `docker-compose.yml` containing only local dependencies. See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for a safe reference workflow.

## AWS ECR and Kubernetes

ECR repositories and Kubernetes resources do not currently exist in this repository. The documented image flow in [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) covers authentication, immutable image tags, vulnerability scanning, and deployment promotion. Do not infer that a successful GitHub Release has deployed to AWS or Kubernetes.

## CI/CD with GitHub Actions

`.github/workflows/deploy.yaml` currently runs on every push:

1. Lint on `ubuntu-latest` with Python 3.8.
2. Install dependencies into `function/` and create `${{ github.sha }}.zip`.
3. Upload the ZIP as an Actions artifact.
4. Create a GitHub Release and upload the ZIP asset.

The workflow has no AWS credentials or deployment step. For production, pin or update action versions, trigger releases from protected branches/tags, use GitHub OIDC instead of long-lived AWS keys, and add tests and an explicit approval gate.

## Configuration and Environment Variables

There is no application configuration file and the handler reads no environment variables. The event contract is documented in [docs/API.md](docs/API.md). If configuration is introduced, keep non-secret defaults in version control and inject secrets through AWS Secrets Manager or SSM Parameter Store; never commit `.env` files.

## API Endpoints

The repository exposes no HTTP endpoint. `lambda_handler(event, context)` is an AWS Lambda invocation contract. An API Gateway or Lambda Function URL would need to be provisioned separately. See [docs/API.md](docs/API.md).

## Security

- Use a least-privilege Lambda execution role and separate roles for CI deployment.
- Prefer GitHub Actions OIDC with an AWS trust policy restricted by repository and branch.
- Pin dependencies and action versions; review Dependabot updates.
- Do not log events containing credentials or personal data.
- Enable ECR image scanning if container packaging is adopted.
- Encrypt artifacts, logs, and any data stores; define retention periods.

## Monitoring

For AWS deployment, use CloudWatch Logs and metrics for invocations, errors, duration, throttles, and concurrency. Add alarms and a trace/correlation ID before exposing the function through an HTTP gateway. The current repository contains no monitoring configuration.

## Troubleshooting

| Symptom | Check |
|---|---|
| `KeyError: 'input'` | Send an event containing the required `input` key. |
| Unsupported input fails | Only `Hello` is supported by the current handler. |
| Import error in Lambda | Ensure dependencies were installed into the ZIP root, not only into a build environment. |
| Workflow package missing | Confirm the build job completed and the SHA-named artifact is available. |
| AWS deployment does not occur | The current workflow publishes to GitHub only; add/configure an AWS deployment job. |
| Self-hosted smoke test is queued | Verify a runner with the required `self-hosted` label is online. |

## Future Roadmap

1. Add unit tests and structured error responses.
2. Upgrade the runtime and dependency versions under compatibility testing.
3. Add infrastructure as code for Lambda, IAM, API Gateway, and CloudWatch.
4. Add OIDC-based AWS deployment with dev/staging/production promotion.
5. Add optional Docker/ECR and Kubernetes manifests if a container target is required.
6. Add security scanning, dependency locking, SBOM generation, and deployment rollback.

## Contributing and License

Development standards and pull-request guidance are in [docs/CONTRIBUTING.md](docs/CONTRIBUTING.md). No license file is currently present; add one before accepting external contributions or distributing the project.