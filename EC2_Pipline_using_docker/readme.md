# EC2 Pipeline using Docker

This folder contains a generic GitHub Actions pipeline template for building a Docker image and deploying it to an AWS EC2 instance.

## What this pipeline does

- runs on `push` to the `main` branch
- builds a Docker image from the repository
- tags the image using `project-name` and the commit SHA
- pushes the image to a Docker registry
- connects to an AWS VM over SSH and deploys the new container
- optionally sends Slack notifications for success or failure

## Files

- `Dockerfile`: builds the application image
- `ec2-deploy.yaml`: GitHub Actions workflow for build, push, and deployment
- `readme.md`: this file

## Placeholders to update

In `ec2-deploy.yaml`, replace the placeholders with your own values:

- `${{ secrets.DOCKER_REPOSITORY }}`: Docker repository path, e.g. `myusername/myrepo`
- `project-name`: generic image/container prefix for your project
- `5000`: port used by the container; update if your app uses a different port
- `/home/ubuntu/app.env`: path to environment file on the EC2 host

## Required GitHub Actions secrets

Add these secrets in your repository settings under `Settings` → `Secrets and variables` → `Actions`:

- `DOCKER_USERNAME`
- `DOCKER_PASSWORD`
- `DOCKER_REPOSITORY`
- `DB_URL` (if your Docker build needs a database connection string)
- `AWS_VM_HOST`
- `AWS_VM_USERNAME`
- `AWS_VM_SSH_KEY`
- `SLACK_WEBHOOK_URL` (optional)

## How to use

1. Copy `ec2-deploy.yaml` to `.github/workflows/ec2-deploy.yml` in your repository root.
2. Update placeholders in the file for your project name, Docker repository, and AWS host.
3. Commit and push the workflow to the `main` branch.
4. Push to `main` to trigger the pipeline.

## Notes

- This is a template, not a production-ready deployment script.
- The workflow uses SSH access with the private key stored in `AWS_VM_SSH_KEY`.
- Adjust the Docker image name, container name, and network name to match your environment.
- Remove debug output or Slack steps if you do not need them.
