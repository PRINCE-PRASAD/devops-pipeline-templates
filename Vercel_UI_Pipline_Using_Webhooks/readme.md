# Vercel UI Pipeline Using Webhooks

This repository contains a GitHub Actions workflow that triggers a Vercel deploy using a Vercel Deploy Hook.

## What this pipeline does

- Runs on `push` to the `main` branch
- Calls the Vercel deploy webhook URL via `curl`
- Uses the GitHub secret `VERCEL_HOOK_ID`

## Required environment / secrets

This workflow requires one GitHub Actions secret:

- `VERCEL_HOOK_ID`

That secret should contain the deploy hook ID from Vercel.

## How to get the Vercel deploy hook ID

1. Open your Vercel dashboard: https://vercel.com/dashboard
2. Select the project you want to deploy.
3. Go to `Settings` for that project.
4. Find the section for `Deploy Hooks` or `Git` -> `Deploy Hooks`.
5. Create a new deploy hook if you do not already have one.
6. Copy the hook URL. It should look like:

   `https://api.vercel.com/v1/integrations/deploy/<HOOK_ID>`

7. Use only the `<HOOK_ID>` part as the secret value.

Example:

- Hook URL: `https://api.vercel.com/v1/integrations/deploy/abc123`
- Secret value: `abc123`

## Add the secret in GitHub Actions

1. Open your repository on GitHub.
2. Go to `Settings` -> `Secrets and variables` -> `Actions`.
3. Click `New repository secret`.
4. Set:
   - Name: `VERCEL_HOOK_ID`
   - Value: the hook ID from Vercel
5. Save the secret.

## Activate the pipeline

1. Ensure the workflow file is available to GitHub Actions.
   - In this repository, the workflow is in `Vercel_UI_Pipline_Using_Webhooks/versal_deploy.yml`.
   - GitHub Actions only runs workflows from `.github/workflows/` in the repo root, so copy or move it there if needed.
2. Confirm the workflow branch matches your repo branch. The current workflow triggers on `main`.
3. Commit and push your workflow and README changes.
4. Push to `main` to trigger the deploy.

## Example workflow file

The current workflow file uses:

```yaml
name: Deploy Next.js App
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: main
    env:
      CI: false
    steps:
      - name: Trigger Vercel Deploy Hook
        run: |
          curl -X POST https://api.vercel.com/v1/integrations/deploy/${{ secrets.VERCEL_HOOK_ID }}
```

## Notes

- No Vercel token is needed for this workflow; the deploy hook itself is enough.
- If you want to use a different branch, update `branches:` in the workflow.
- If the deploy does not start, verify that `VERCEL_HOOK_ID` is correct and that the file is in `.github/workflows/`.
