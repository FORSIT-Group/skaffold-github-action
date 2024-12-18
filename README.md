Here's an expanded README.md for your skaffold-github-action repository:

# Skaffold GitHub Action

A flexible GitHub Action for building and deploying applications using Skaffold.

## Inputs

### Required Inputs
- `default-repo`: Container registry URL (e.g., 'gcr.io/my-project')

### Optional Inputs
- `skaffold-version`: Skaffold version (default: 'latest')
- `profile`: Skaffold profile name
- `filename`: Path to skaffold.yaml (default: 'skaffold.yaml')
- `command`: Skaffold command to run (default: 'run')
- `namespace`: Kubernetes namespace
- `tag`: Custom tag for images
- `cache-file`: Path to cache file
- `build-artifacts`: Path to build artifacts file

## Commands

### Basic Build and Deploy
```
- uses: your-username/skaffold-github-action@main
  with:
    default-repo: 'gcr.io/my-project'
    command: 'run'
    profile: 'prod'
```

### Build Only
```
- uses: your-username/skaffold-github-action@main
  with:
    default-repo: 'gcr.io/my-project'
    command: 'build'
    filename: './apps/service/skaffold.yaml'
```

### Deploy Only
```
- uses: your-username/skaffold-github-action@main
  with:
    default-repo: 'gcr.io/my-project'
    command: 'deploy'
    build-artifacts: 'build.json'
```

### Debug Mode
```
- uses: your-username/skaffold-github-action@main
  with:
    default-repo: 'gcr.io/my-project'
    command: 'debug'
    namespace: 'dev'
```

### Run Diagnostics
```
- uses: your-username/skaffold-github-action@main
  with:
    command: 'diagnose'
    filename: './path/to/skaffold.yaml'
```

## Complete Workflow Example

```
name: Skaffold CI/CD
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Optional: Set up Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      # Optional: Login to Container Registry
      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: gcr.io
          username: _json_key
          password: ${{ secrets.GCR_JSON_KEY }}
      
      # Build Phase
      - uses: your-username/skaffold-github-action@main
        with:
          default-repo: 'gcr.io/my-project'
          command: 'build'
          profile: 'prod'
          filename: './services/app/skaffold.yaml'
          cache-file: '.cache/skaffold'
          tag: '${{ github.sha }}'
          
      # Deploy Phase
      - uses: your-username/skaffold-github-action@main
        with:
          default-repo: 'gcr.io/my-project'
          command: 'deploy'
          profile: 'prod'
          namespace: 'production'
          build-artifacts: 'build.json'
```

## Common Command Flags

You can append these flags to the command input:

- `--filename/-f`: Set skaffold.yaml location
- `--profile/-p`: Use specific profile
- `--namespace/-n`: Deploy to specific namespace
- `--tag`: Set custom tag
- `--default-repo`: Override default repository
- `--cache-file`: Specify cache file location
- `--build-artifacts`: Specify build artifacts file

## Directory Structure Example

```
your-project/
├── skaffold.yaml
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── .github/
│   └── workflows/
│       └── deploy.yml
└── src/
    └── app/
```

## Environment Variables

The action supports all standard Skaffold environment variables:

- `SKAFFOLD_DEFAULT_REPO`
- `SKAFFOLD_CACHE_FILE`
- `SKAFFOLD_NAMESPACE`
- `SKAFFOLD_PROFILE`

## Tips

- Use caching to speed up builds:
  ```
  - uses: actions/cache@v3
    with:
      path: ~/.skaffold/
      key: ${{ runner.os }}-skaffold-${{ hashFiles('**/skaffold.yaml') }}
  ```

- For multiple profiles:
  ```
  - uses: your-username/skaffold-github-action@main
    with:
      command: 'run'
      profile: 'prod,monitoring'
  ```

- For debugging:
  ```
  - uses: your-username/skaffold-github-action@main
    with:
      command: 'debug --verbosity=debug'
  ```
