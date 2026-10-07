# Shared workflows

## `cloud-run.yml`

Builds a repository's container image on GitHub Actions, pushes it to Google Artifact Registry
(`europe-west1-docker.pkg.dev/mawadao/mawadao/<image>`) and deploys it to Cloud Run in the
`mawadao` Google Cloud project. Every build and deploy is visible in the calling repository's
Actions tab.

| Event | Builds | Pushes the image | Deploys |
| --- | --- | --- | --- |
| Pull request (including forks) | Yes | No | No |
| Push to `main` | Yes | `:<sha>` and `:main` | Yes, if `service` is set |
| Tag `vX.Y.Z` | Yes | `:<sha>` and `:vX.Y.Z` | No |

GitHub signs in to Google Cloud with Workload Identity Federation, so no keys are stored. Google
Cloud only accepts workflows from repositories in the `mawadao` organisation, running on `main` or
a tag, and they act as the `github-deploy` service account, which can push images and deploy to
Cloud Run but nothing else.

Call it from a repository:

```yaml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read
  id-token: write

jobs:
  cloud-run:
    uses: mawadao/.github/.github/workflows/cloud-run.yml@main
    with:
      image: my-service
      service: my-service      # leave out to build and push only
      public: true             # for websites and public APIs
```

Inputs: `image`, `service`, `context`, `dockerfile`, `build-args` (one `KEY=value` per line, never
secrets), `public`, `flags` (extra `gcloud run deploy` flags) and `region` (default `europe-west1`).
Runtime secrets belong in Secret Manager, mounted on the Cloud Run service.
