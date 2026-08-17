# digirati-ops-actions

This repository contains shared GitHub Actions that can be used across Digirati repositories for common CI/CD operations.

## Available Actions

### Get Commit Version

Gets the commit version (e.g. 'sha-d3a7880') 

#### Outputs

- `commit_version`: The Commit Version from github

#### Example Usage

```yaml
jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        id: checkout
        uses: actions/checkout@v6
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ env.AWS_ROLE }}
          aws-region: ${{ env.AWS_REGION }}
      - name: Get Commit Version
        id: get-commit-version
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/get_commit_version@main
      - name: Query for existing backend image.
        id: ecr_image_query
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/query_image_version@main
        with:
          repository_name: ${{ env.REPOSITORY_NAME }}
          app_version: ${{steps.get-commit-version.outputs.commit_version}}
          environment_tag: ${{ env.ENVIRONMENT_TAG }}
```

### Get Deployment Version

Gets the deployed version by hitting the /version endpoint

#### Inputs

- `deployment_url`: The url of the deployed web app e.g. www.digirati.com

#### Outputs

- `deployed_version`: The deployed version from the website version endpoint

#### Example Usage

```yaml
jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - name: Get Deployed Version
        id: get-deployed-version
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/get_deployed_version@main
        with:
          deployment_url: ${{env.DEPLOYMENT_URL}}
```

### Image Build ECR Push

Builds the image using the specified dockerfile and then uploads the image to the ECR

#### Inputs

- `repository_name`: The name of the ECR repository to deploy to.
- `docker_context`: Path in the repository to the Dockerfile to build.
- `app_version`: The App Version which this deployment will return on the /version endpoint
- `environment_tag`: The environment tag for this deployment e.g. 'Dev', 'Test', 'Prod' (Optional)

#### Example Usage

```yaml
jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        id: checkout
        uses: actions/checkout@v6
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ env.AWS_ROLE }}
          aws-region: ${{ env.AWS_REGION }}
      - name: Get Commit Version
        id: get-commit-version
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/get_commit_version@main
      - name: Build and push the backend image
        id: build-push-backend-image
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/image_build_ecr_push@main
        with:
          repository_name: ${{ env.REPOSITORY_NAME }}
          docker_context: ./exemplar_python/
          app_version: ${{steps.get-commit-version.outputs.commit_version}}
```

### Query Image Version

Gets the image version from the ECR

#### Inputs

- `step_name`: The step name to append to the summary for this stage.
- `repository_name`: The name of the ECR repository to check in.
- `app_version`: The app version to check for.

#### Outputs

- `exists`: Whether or not the image exists.

#### Example Usage

```yaml
jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        id: checkout
        uses: actions/checkout@v6
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ env.AWS_ROLE }}
          aws-region: ${{ env.AWS_REGION }}
      - name: Get Commit Version
        id: get-commit-version
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/get_commit_version@main
      - name: Query for existing backend image.
        id: ecr_image_query
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/query_image_version@main
        with:
          repository_name: ${{ env.REPOSITORY_NAME }}
          app_version: ${{steps.get-commit-version.outputs.commit_version}}
```

### Tag Image Version

Tags the image version in the ECR using the provided version

#### Inputs

- `repository_name`: The name of the ECR repository to check in.
- `app_version`: The app version to tag.
- `environment_tag`: The tag to apply.

#### Example Usage

```yaml
jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        id: checkout
        uses: actions/checkout@v6
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ env.AWS_ROLE }}
          aws-region: ${{ env.AWS_REGION }}
      - name: Get Commit Version
        id: get-commit-version
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/get_commit_version@main
      - name: Tag ECR Image.
        id: tag_image_version
        uses: digirati-co-uk/digirati-ops-actions/.github/actions/tag_image_version@main
        with:
          repository_name: ${{ env.REPOSITORY_NAME }}
          app_version: ${{steps.get-commit-version.outputs.commit_version}}
          environment_tag: ${{ env.ENVIRONMENT_TAG }}
```


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
