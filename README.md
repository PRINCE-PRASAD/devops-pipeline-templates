# DevOps Pipeline Templates

A collection of reusable GitHub Actions and CI/CD pipeline templates for AWS, Vercel, and Docker deployments.

## Available pipeline templates

- [AWS ECS + ECR + Docker + GitHub Actions](AWS_ECS_ECR_GithubAction_DockerImage/readme.md)
  - A generic ECS deployment pipeline that builds and pushes a Docker image to ECR, and deploys it to an ECS service.

- [AWS ECS + ECR + Trivy + SonarQube + OWASP ZAP](AWS_ECS_ECR_Trivy_SonarQube_OwsapZap_DockerImage/readme.md)
  - A full security-aware ECS pipeline with SonarQube quality gate, Trivy image scanning, Prisma migration support, and OWASP ZAP DAST.

- [EC2 Docker deployment with DockerHub](EC2_DockerImage_DockerHub_GithubAction/readme.md)
  - Builds a Docker image and deploys it to an AWS EC2 instance using DockerHub and GitHub Actions.

- [Vercel UI deployment using Vercel token](Vercel_UI_Pipline_using_Vercel_Token/readme.md)
  - Deploys a frontend app to Vercel using a Vercel API token.

- [Vercel UI deployment using webhooks](Vercel_UI_Pipeline_Using_Webhooks/readme.md)
  - Deploys a frontend app to Vercel through webhook-based workflow triggers.

## How to use

1. Open the pipeline folder you want.
2. Review the folder `readme.md` for configuration, required secrets, and usage instructions.
3. Copy the workflow file into your own repository under `.github/workflows/`.
4. Replace placeholder values with your real AWS, Vercel, or Docker settings.

## Notes

- Each folder contains a template workflow and a `readme.md` describing how to configure it.
- Replace placeholder names like `project-name`, AWS ARNs, and secret keys before using in production.
- If a folder's `readme.md` is missing or empty, open the workflow file and review the job names and secrets manually.
