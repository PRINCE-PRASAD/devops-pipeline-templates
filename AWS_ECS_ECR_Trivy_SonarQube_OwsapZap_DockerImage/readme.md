# AWS ECS + ECR + Trivy + SonarQube + OWASP ZAP Pipeline

This repository contains a generic GitHub Actions workflow for building, scanning, and deploying a containerized backend application to AWS ECS.

## What it does

The workflow performs these stages in sequence:

1. **SonarQube code quality scan**
   - Runs tests, generates coverage, and sends results to SonarQube.
   - Fails the workflow if the SonarQube quality gate does not pass.

2. **Build and push Docker image to ECR**
   - Uses AWS OIDC role assumption for secure authentication.
   - Builds the Docker image locally and pushes it to a configured Amazon ECR repository.

3. **Trivy image security scan**
   - Pulls the pushed image from ECR.
   - Runs Trivy to scan for known vulnerabilities and uploads a report artifact.

4. **Register ECS task definition revision**
   - Fetches the current ECS task definition.
   - Replaces the container image with the new image URI.
   - Registers a new task definition revision.

5. **Run Prisma migrations in ECS**
   - Starts an ECS Fargate task that executes `prisma migrate deploy`.
   - Waits for the task to finish and checks the exit code.

6. **Deploy updated task definition to ECS service**
   - Updates the ECS service to use the new task definition revision.
   - Waits until the service is stable.

7. **OWASP ZAP DAST scan**
   - Runs a ZAP baseline scan against the deployed application URL.
   - Sends a Slack notification when the scan completes.

## Trigger

The workflow is configured to run on `push` events to the `main` branch.

## Required GitHub Secrets

Set these repository secrets before using the workflow:

- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`
- `ECS_CLUSTER_NAME`
- `ECS_SERVICE_NAME`
- `ECS_TASK_DEFINITION_FAMILY`
- `ECS_CONTAINER_NAME`
- `ECS_SUBNET_IDS`
- `ECS_SECURITY_GROUP_IDS`
- `DATABASE_URL`
- `SENTRY_DSN`
- `SONAR_TOKEN`
- `SONAR_HOST_URL`
- `SONAR_PROJECT_KEY`
- `SLACK_WEBHOOK_URL`
- `APP_URL`

## Required AWS IAM Role

The workflow currently uses a placeholder role ARN:

```yaml
arn:aws:iam::123456789012:role/project-name-main-role
```

Replace it with your real AWS role that has permission to:

- assume the role via GitHub OIDC
- access ECR
- describe and register ECS task definitions
- run ECS tasks
- update ECS services
- read CloudWatch logs if migration debugging is needed

## Workflow jobs explained

### 1. `sonarqube`
- Checks out source code.
- Sets up Node.js and pnpm.
- Installs dependencies.
- Generates Prisma client and test coverage.
- Executes SonarQube scanning and waits for the quality gate.
- Sends a Slack notification on success or failure.

### 2. `build`
- Requires successful SonarQube scan.
- Configures AWS credentials using OIDC.
- Logs in to ECR.
- Builds the Docker image and tags it with the commit SHA.
- Pushes the image to the configured ECR repository.
- Exports the image tag for later jobs.
- Sends Slack alerts on failure.

### 3. `trivy`
- Requires the `build` job.
- Authenticates to AWS/ECR again.
- Verifies the image exists in ECR and pulls it.
- Runs Trivy on the image.
- Uploads the Trivy report as an artifact.
- Sends a Slack notification when the scan is complete.

### 4. `register-task-def`
- Requires `build` and `trivy`.
- Loads the current ECS task definition.
- Replaces the container image with the new image URI.
- Removes fields that cannot be re-registered.
- Registers a new ECS task definition revision.

### 5. `migrate`
- Requires `build` and `register-task-def`.
- Starts an ECS task to apply Prisma migrations.
- Waits for task completion.
- Checks the container exit code to determine success.
- Prints logs only on failure.

### 6. `deploy`
- Requires `register-task-def` and `migrate`.
- Updates the ECS service to use the new task definition revision.
- Waits for the service to stabilize.
- Sends Slack notifications on success or failure.

### 7. `zap-scan`
- Requires the `deploy` job.
- Runs the OWASP ZAP baseline scan against the URL in `APP_URL`.
- Continues on error so the deployment itself is not blocked by scan findings.
- Sends a final Slack notification with the scan result.

## Customization notes

- Change `main` to another branch if you want a different deployment branch.
- Update the Docker build args if your application needs additional build-time values.
- If you do not use Prisma migrations, either remove the `migrate` job or adapt it to your database process.
- Verify the ZAP `target` URL matches the deployed application address.
- Adjust `ECS_SUBNET_IDS` and `ECS_SECURITY_GROUP_IDS` formatting to be valid for the AWS CLI.

## How to use

1. Create the GitHub repository and add this workflow file under `.github/workflows/deploy_ecs_dast_sast.yml`.
2. Add the required GitHub Secrets.
3. Ensure your AWS role is configured for GitHub OIDC trust.
4. Push changes to `main`.
5. Monitor Actions for the full pipeline run.

## Notes

- This workflow is generic and uses placeholder names like `project-name`.
- Replace placeholders with real project values before production use.
- Slack notifications are optional, but the workflow assumes `SLACK_WEBHOOK_URL` is configured.
