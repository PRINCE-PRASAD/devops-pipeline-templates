# AWS CodeBuild deploy pipeline

This pipeline triggers an existing AWS CodeBuild project from GitHub Actions, waits for the build to finish, and optionally sends Slack notifications for success or failure.

## What it does

- Checks out repository code.
- Uses GitHub OIDC to assume an AWS IAM role.
- Starts an AWS CodeBuild build with `IMAGE_TAG` set to the current commit SHA.
- Polls the CodeBuild status until the build succeeds or fails.
- Sends Slack notifications for build success or failure.

## Trigger

The workflow runs on `push` to the `main` branch by default.

## Required GitHub Secrets

- `CODEBUILD_PROJECT_NAME`
- `SLACK_WEBHOOK_URL`

## Required AWS IAM role

The workflow uses a placeholder OIDC role ARN:

```yaml
arn:aws:iam::123456789012:role/project-name-codebuild-role
```

The real role must have permissions to:

- assume the role via GitHub OIDC
- start CodeBuild builds
- read CodeBuild build status

## Customization

- Replace `main` with the branch you want to trigger from.
- Update `AWS_REGION` if your CodeBuild project is in a different region.
- Replace the placeholder IAM role ARN with your actual role.
- Add additional `--environment-variables-override` entries if your build requires more environment variables.

## GitHub Actions secrets to add

In your repository settings, add these secrets:

- `CODEBUILD_PROJECT_NAME` — the name of your existing AWS CodeBuild project.
- `SLACK_WEBHOOK_URL` — the Slack webhook URL used for pipeline notifications.

If your CodeBuild project or app uses additional sensitive values such as database credentials, API keys, or ECR repository names, store them in GitHub secrets and pass them through CodeBuild environment variables or buildspec overrides.

## CodeBuild environment and VPC configuration

This pipeline triggers CodeBuild, but the actual build and deployment happen inside your CodeBuild project.

### CodeBuild environment variables to add

Configure these environment variables inside your CodeBuild project:

- `AWS_REGION` — the AWS region where CodeBuild, ECR, ECS, and the database live.
- `AWS_ACCOUNT_ID` — your AWS account ID.
- `ECR_REPOSITORY` — the ECR repository name where Docker images are pushed.
- `ECS_CLUSTER_NAME` — the ECS cluster that runs your service.
- `ECS_SERVICE_NAME` — the ECS service name to deploy the updated task.
- `ECS_TASK_DEFINITION_FAMILY` — the task definition family name used by ECS.
- `ECS_CONTAINER_NAME` — the container name inside the ECS task definition.
- `DATABASE_URL` — the database connection string used for migrations.
- `SENTRY_DSN` — optional, if your app uses Sentry during build or runtime.
- `APP_URL` — optional, if the build or tests need the deployed application URL.

If your build performs additional steps, add any extra environment variables required by your Docker build or migration commands.

### CodeBuild network setup for private VPC apps

For production, especially when your application and database are inside a VPC and private subnet, configure your CodeBuild project to:

- use the same AWS region as your application resources
- enable VPC access and attach the CodeBuild project to private subnets
- attach security groups that allow access to the database, ECR, and any internal services
- enable NAT or VPC endpoints if CodeBuild needs outbound internet access for package downloads or external APIs

If the app is deployed inside a private VPC, CodeBuild should also have access to the database migration target and any backend services it needs to validate the build.

## Production recommendations

- Keep the application inside private subnets whenever possible.
- Use AWS managed services such as ECS, ECR, and CodeBuild for scalable deployment.
- Run database migration steps from a secure, VPC-enabled environment.
- Use IAM roles with least privilege and GitHub OIDC instead of long-lived AWS keys.

## Pipeline components and responsibilities

- `GitHub Actions` — the pipeline orchestrator that triggers CodeBuild and waits for completion.
- `AWS CodeBuild` — the build engine that runs your project build, Docker image creation, tests, migrations, and deployment logic.
- `Docker image` — the application container built during the CodeBuild run.
- `Amazon ECR` — the container registry where Docker images are stored.
- `AWS ECS` — the runtime platform where your containerized application is deployed (if your CodeBuild process deploys to ECS).
- `CodeBuild project` — defines the actual build environment, buildspec, and VPC settings used when GitHub triggers the pipeline.
- `Slack notifications` — optional alerts sent by GitHub Actions after the CodeBuild job completes.

## Example flow

1. GitHub Actions starts when code is pushed to `main`.
2. The workflow assumes an AWS role via OIDC and triggers an AWS CodeBuild build.
3. CodeBuild runs the build, optionally builds a Docker image, runs migrations, and deploys the application.
4. GitHub Actions polls CodeBuild until it finishes.
5. When CodeBuild completes, GitHub Actions sends Slack success or failure notifications.

## Detailed step-by-step pipeline behavior

1. **Checkout source code**
   - GitHub Actions checks out the repository using `actions/checkout`.
   - This ensures the current commit contents are available for the pipeline.

2. **Authenticate to AWS using OIDC**
   - The workflow uses `aws-actions/configure-aws-credentials` with `id-token: write`.
   - It assumes the IAM role specified by `role-to-assume`.
   - This avoids storing long-lived AWS keys in GitHub secrets.

3. **Trigger AWS CodeBuild**
   - The workflow calls `aws codebuild start-build`.
   - It passes the current commit SHA as `IMAGE_TAG` to CodeBuild.
   - The build is started in the preconfigured CodeBuild project.

4. **CodeBuild executes the build and deployment logic**
   - Inside CodeBuild, your `buildspec.yml` or custom build commands should:
     - build the Docker image
     - tag it with `IMAGE_TAG`
     - push the image to Amazon ECR
     - optionally create or update an ECS task definition
     - optionally run database migration commands
     - optionally update an ECS service
   - This is the place where Docker image creation and migration work actually happen.

5. **Wait for CodeBuild completion**
   - GitHub Actions polls CodeBuild using `batch-get-builds`.
   - It waits until the build status becomes `SUCCEEDED`, `FAILED`, `FAULT`, `STOPPED`, or `TIMED_OUT`.

6. **Report final status**
   - If CodeBuild succeeds, GitHub Actions sends a Slack success notification.
   - If CodeBuild fails, GitHub Actions sends a Slack failure notification.

## What this pipeline is responsible for

- **GitHub Actions**: coordinates the pipeline, authenticates to AWS, triggers CodeBuild, waits for completion, and sends notifications.
- **AWS CodeBuild**: runs the actual build and deployment steps defined in your CodeBuild project.
- **Docker image build**: typically happens inside CodeBuild and is tagged with the Git commit SHA.
- **Amazon ECR push**: the built Docker image is pushed to ECR from CodeBuild.
- **ECS task definition**: CodeBuild can create or update a new ECS task definition revision that points to the new image.
- **Database migration**: CodeBuild can run migrations inside your VPC or on a secure instance before deployment.
- **Deployment**: CodeBuild can deploy the updated ECS task definition to the ECS service.

## Note about VPC and private subnet deployments

- If your app lives inside a private VPC and subnet, configure CodeBuild to use the same VPC/subnets.
- Ensure CodeBuild can reach the database and any internal services needed for migration.
- If CodeBuild needs external access, provide NAT gateway access or VPC endpoints.

## Usage

1. Place `ecs_codebuild_deploy.yaml` under `.github/workflows/` in your target repository.
2. Configure the required GitHub secrets.
3. Ensure the referenced AWS IAM role is trusted for GitHub OIDC.
4. Make sure your CodeBuild project is configured with VPC/subnet settings if the app and database are inside a private VPC.
5. Push to `main` to start the pipeline.
