# digirati-ops-actions

This repository contains shared GitHub Actions that can be used across Digirati repositories for common CI/CD operations.

## Available Actions

### archive-artifacts

Uploads artifacts produced during a workflow run to S3 and makes them available via CloudFront.

#### Outputs

- `url`: Base URL of assets available on CloudFront

#### Example Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Create some artifacts
      - run: mkdir -p artifacts/build
      - run: echo "Hello World" > artifacts/build/index.html
      
      # Upload artifacts to GitHub Actions
      - uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: artifacts/build
      
      # Archive artifacts to S3/CloudFront
      - name: Archive artifacts
        id: archive
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/archive-artifacts@main
      
      # Use the CloudFront URL
      - name: Output artifact URL
        run: echo "Artifacts available at ${{ steps.archive.outputs.url }}"
```

### download-archived-artifacts

Downloads artifacts that were previously archived to S3.

#### Inputs

- `branch`: The name of the branch to pull artifacts for (default: current branch)

#### Example Usage

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Download artifacts from S3
      - name: Download artifacts
        uses: digirati-ops-actions/.github/actions/download-archived-artifacts@main
        with:
          branch: main  # Optional, defaults to current branch
      
      # Use the downloaded artifacts
      - name: Use artifacts
        run: cat artifacts/build/index.html
```

## Authentication

Actions invoking AWS authenticate using Cognito Identity Pool credentials. The actions handle authentication automatically, no additional setup is required for repositories within the Digirati GitHub organization.
