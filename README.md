# Skaffold GitHub Action

A flexible GitHub Action for building and deploying applications using Skaffold.

## Inputs

### Required Inputs
- `default-repo`: Container registry URL (e.g., 'gcr.io/my-project')

### Optional Inputs
- `skaffold-version`: Skaffold version (default: 'latest')
- `command`: Skaffold command to run (default: 'run')
- `profile`: Skaffold profile name
- `filename`: Path to skaffold.yaml (default: 'skaffold.yaml')
- `namespace`: Kubernetes namespace
- `tag`: Custom tag for images
- `short-hash`: Include git short hash in tag (default: 'false')
- `cache-file`: Path to cache file
- `build-artifacts`: Path to build artifacts file
- `collect-metrics`: Enable/disable Skaffold metrics collection (default: 'false')

## Usage Examples

### Basic Build
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'build'
    default-repo: 'gcr.io/my-project'
```

### Build with Custom Tag and Git Hash
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'build'
    profile: 'prod'
    tag: 'release-1.0'
    short-hash: 'true'
    default-repo: 'gcr.io/my-project'
```

### Deploy with Specific Profile and Namespace
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'deploy'
    profile: 'prod'
    namespace: 'production'
    default-repo: 'gcr.io/my-project'
```

### Run Diagnostics
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'diagnose'
    filename: './path/to/skaffold.yaml'
```

## Complete Workflow Example

```
name: Build and Deploy
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Important for git operations
      
      # Optional: Set up Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # Login to Container Registry
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ secrets.CONTAINER_REGISTRY }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      
      # Build Phase
      - uses: your-org/skaffold-github-action@main
        with:
          command: 'build'
          profile: 'prod'
          tag: 'release'
          short-hash: 'true'
          filename: './infrastructure/stage/skaffold.yaml'
          default-repo: ${{ vars.CONTAINER_REGISTRY }}
          collect-metrics: 'false'
          
      # Deploy Phase
      - uses: your-org/skaffold-github-action@main
        with:
          command: 'deploy'
          profile: 'prod'
          namespace: 'production'
          default-repo: ${{ vars.CONTAINER_REGISTRY }}
```

## Common Command Flags

The action supports these Skaffold command flags:

- `filename/-f`: Set skaffold.yaml location
- `profile/-p`: Use specific profile
- `namespace/-n`: Deploy to specific namespace
- `tag`: Set custom tag
- `default-repo`: Override default repository
- `cache-file`: Specify cache file location
- `build-artifacts`: Specify build artifacts file

## Directory Structure Example

```
your-project/
├── apps/
├── infrastructure/
│   ├── environments/
│   ├── k8s-scripts/
│   ├── manifests/
│   ├── stage/
│   │   └── skaffold.yaml
│   └── scripts/
├── [...]
└── README.md
```

## Environment Variables

The action supports all standard Skaffold environment variables:

- `SKAFFOLD_DEFAULT_REPO`
- `SKAFFOLD_CACHE_FILE`
- `SKAFFOLD_NAMESPACE`
- `SKAFFOLD_PROFILE`

## Tips

### Caching
Use GitHub's cache action to speed up builds:
```
- uses: actions/cache@v3
  with:
    path: ~/.skaffold/
    key: ${{ runner.os }}-skaffold-${{ hashFiles('**/skaffold.yaml') }}
```

### Multiple Profiles
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'run'
    profile: 'prod,monitoring'
```

### Debug Mode
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'debug'
    profile: 'dev'
```

### Custom Working Directory
```
- uses: your-org/skaffold-github-action@main
  with:
    command: 'build'
    filename: './services/myapp/skaffold.yaml'
```

## Notes

- The action automatically handles Skaffold installation
- Git short hash in tags is optional and controlled by the `short-hash` input
- Metrics collection is disabled by default
- The action supports all major Skaffold commands and configurations
- Registry URLs should be stored as GitHub Variables, not Secrets
