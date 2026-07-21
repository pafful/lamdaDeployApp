# Deployment Guide

## Deployment Status

The current GitHub Actions workflow builds a Lambda ZIP and publishes it to a GitHub Release. It does **not** deploy to AWS. No Dockerfile, Compose file, ECR repository, Kubernetes manifest, Helm chart, or IaC module is checked in.

## Lambda ZIP Deployment

1. Create or select an AWS Lambda function using a supported Python runtime.
2. Build the bundle on a compatible Linux environment:

   ```bash
   python -m pip install -r function/requirements.txt -t function/
   cd function
   zip -r ../lambda.zip .
   cd ..
   ```

3. Upload `lambda.zip` to Lambda and set the handler to `lambda_function.lambda_handler`.
4. Configure an execution role with only the permissions the function needs.
5. Invoke with `{ "input": "Hello" }` and confirm the `World` response.

The existing workflow performs the packaging step and produces a SHA-named ZIP. Review the ZIP before deployment and avoid shipping unnecessary files.

## Docker

No Docker configuration exists yet. If Lambda container images are selected, add a `Dockerfile` using the AWS Lambda Python base image, copy the function and dependencies into the Lambda task root, and set the handler as the image command. If Kubernetes is selected, use a minimal supported Python image, run as non-root, expose only the required port, and add readiness/liveness probes.

## Docker Compose (Local Only)

Compose is not required by the current stateless handler. If introduced for local HTTP emulation, keep it development-only, do not put production credentials in it, and provide health checks. A typical future stack would contain an app service and an optional local emulator; there is no Compose file to run today.

## Amazon ECR

For an image-based deployment, a secure flow is:

1. Create a private ECR repository with image scanning and lifecycle retention.
2. Authenticate using an AWS role, preferably through GitHub Actions OIDC.
3. Build and tag with an immutable commit SHA, not only `latest`.
4. Push the image and record the digest.
5. Scan the image and promote the digest to the target environment.

The CI role should be limited to the required ECR actions (`GetAuthorizationToken`, repository upload actions, and image read actions as appropriate). Do not store static AWS access keys in GitHub secrets.

## Kubernetes Deployment

Kubernetes deployment is a future option. Before enabling it, add a Deployment, Service, resource requests/limits, security context, probes, and a NetworkPolicy. The image should be referenced by an immutable ECR digest. Use a namespace per environment and an external secret mechanism rather than committing credentials.

A production rollout should be:

1. Push and scan the SHA-tagged image.
2. Apply manifests through an approved identity.
3. Wait for rollout completion and verify readiness.
4. Run a smoke test.
5. Roll back to the previous digest if checks fail.

## GitHub Actions CI/CD

The current workflow has `lint`, `build`, and `publish` jobs. To make it a deployment pipeline:

- trigger deployment from protected tags or environments;
- add unit tests and dependency/security scans;
- configure AWS OIDC with a repository/branch-restricted trust policy;
- deploy the ZIP with `aws lambda update-function-code`, or push an ECR image and update the target;
- add environment approval and rollback verification.

The current action versions are older (`actions/checkout@v2`, `actions/setup-python@v2`) and should be reviewed and upgraded as part of hardening.

## Azure Pipelines and CodeBuild

The Azure YAML files and `buildspec.yaml` are starter templates containing echo/install commands. They are not equivalent to a production deployment and currently duplicate some CI responsibilities. Choose one primary CI/CD system to reduce drift.

## Rollback

For ZIP deployments, retain the last known-good artifact and use Lambda versions/aliases for controlled promotion. For images, redeploy the prior ECR digest. Record deployment metadata (commit, artifact digest, operator, and environment) for auditability.
