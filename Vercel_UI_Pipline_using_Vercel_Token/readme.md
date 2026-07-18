# Vercel UI Pipeline using Vercel Token

This repository contains a GitHub Actions workflow that triggers a Vercel deployment using a Vercel API token.

## What this pipeline does

- Runs on `push` to the `main` branch
- Calls the Vercel Deployments API via `curl`
- Uses the GitHub secret `VERCEL_TOKEN`
- Sends deployment configuration including project name, target, and GitHub repo details

## Required GitHub Actions secrets

Add the following secret in GitHub:

- `VERCEL_TOKEN`
  - A Vercel personal token with `Deployments` permission.

## Values to configure in the workflow

Open `Vercel_UI_Pipline_using_Vercel_Token/vercel_deploy.yml` and update:

- `name`: the Vercel project name under your account
- `gitSource.repoId`: your GitHub repository ID or Vercel Git source identifier
- `ref`: repository branch to deploy (currently `main`)

Example section to update:

```yaml
"name": "your-project-name",
"target": "production",
"gitSource": {
  "type": "github",
  "repoId": "Put your repo ID here",
  "ref": "main"
}
```

## How to get the Vercel token

1. Open Vercel Dashboard: https://vercel.com/dashboard
2. Click the gear icon in the lower-left corner of the sidebar.
3. Choose `Settings` from the left menu.
4. Select `Tokens` or `Personal Tokens`.
5. Click `Create Token`.
6. Give the token a name and copy it.

> Keep the token secret. Do not commit it to GitHub.

## Add the secret in GitHub Actions

1. Open your repository on GitHub.
2. Go to `Settings` -> `Secrets and variables` -> `Actions`.
3. Click `New repository secret`.
4. Add:
   - Name: `VERCEL_TOKEN`
   - Value: your Vercel personal token
5. Save the secret.

## Make the pipeline active

1. Ensure the workflow file is placed in `.github/workflows/` in the repository root.
   - GitHub Actions only runs workflows from `.github/workflows/`.
   - If the current file is not there, move or copy `Vercel_UI_Pipline_using_Vercel_Token/vercel_deploy.yml` to `.github/workflows/vercel_deploy.yml`.
2. Commit and push the workflow file and any README changes.
3. Push to the `main` branch to trigger the workflow.

## Workflow example

The workflow currently uses:

```yaml
name: Vercel Deployment Pipeline
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: main
    permissions:
      contents: read
    env:
      CI: false
    steps:
      - name: Trigger Vercel Production Deployment
        run: |
          RESPONSE=$(curl -s -o /tmp/vercel_response.json -w "%{http_code}" \
            -X POST https://api.vercel.com/v13/deployments \
            -H "Authorization: Bearer ${{ secrets.VERCEL_TOKEN }}" \
            -H "Content-Type: application/json" \
            -d '{
              "name": "your-project-name",
              "target": "production",
              "gitSource": {
                "type": "github",
                "repoId": "Put your repo ID here",
                "ref": "main"
              }
            }')
```

## Troubleshooting

- If deployment fails, verify `VERCEL_TOKEN` is correct and active.
- Make sure the token has deployment permissions in Vercel.
- Confirm GitHub Actions workflow is in `.github/workflows/` and the branch is `main`.
- Check GitHub Actions logs for errors from `curl` or the Vercel API.
