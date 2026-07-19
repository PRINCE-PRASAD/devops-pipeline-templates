# AWS ECS Deployment Template

This folder contains a GitHub Actions pipeline template for building a Docker image, pushing it to Amazon ECR, registering a new ECS task definition, running Prisma migrations, and deploying to ECS.

## What this pipeline does

- triggers on `push` to the `main` branch
- builds a Docker image and tags it with the Git SHA
- pushes the image to Amazon ECR
- registers a new ECS task definition revision pointing to the new image
- runs Prisma migrations in ECS using the new task definition
- updates the ECS service to deploy the new task definition
- sends Slack notifications on failure or success

## Required GitHub Secrets

Add these repository secrets under `Settings` → `Secrets and variables` → `Actions`:

- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`
- `SENTRY_DSN` (optional, if your app build uses it)
- `SLACK_WEBHOOK_URL`
- `ECS_TASK_DEFINITION_FAMILY`
- `ECS_CONTAINER_NAME`
- `ECS_CLUSTER_NAME`
- `ECS_SERVICE_NAME`
- `ECS_SUBNET_IDS`
- `ECS_SECURITY_GROUP_IDS`
- `DATABASE_URL`

## How to use

1. Copy `ECS_Deploy.yaml` into `.github/workflows/ECS_Deploy.yaml` in your repository root.
2. Update the pipeline file as needed for your project.
   - Replace any generic placeholders such as `project-name`.
   - Confirm the AWS role ARN and region values match your environment.
   - Confirm the ECS task definition family, cluster, service, and container name are correct.
3. Add the required GitHub secrets listed above.
4. Commit and push to the `main` branch.
5. GitHub Actions will run the pipeline automatically on push.

## Important notes

- This is a template, not a complete production pipeline.
- The workflow uses OIDC to assume an AWS IAM role. Update `role-to-assume` to your own IAM role ARN.
- The ECS migration step uses `DATABASE_URL` and runs inside ECS.

## Database access note

- The `Run Prisma Migrations` job runs the migration command from an ECS task.
- That task must be able to connect to your database.
- If your database is publicly accessible and the ECS task can reach it, the migration will work.
- If your database is private, make sure the ECS task runs in a network that can access the database (VPC, subnet, and security group settings).

> If the DB is unreachable from ECS, the migration step will fail and the pipeline will not complete successfully.

## Recommended changes for template use

- Replace `arn:aws:iam::123456789012:role/project-name-main-role` with the actual AWS IAM role ARN for your GitHub OIDC trust.
- Replace any fixed app names or resource names with your own values.
- Keep secrets outside source control and only in GitHub Actions secrets.

## Workflow file location

The pipeline file is currently `AWS_ECS_Using_GithubAction_Docker/ECS_Deploy.yaml`.

To activate it, move or copy it to `.github/workflows/ECS_Deploy.yaml` if it is not already there.


