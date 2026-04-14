# GitLab CI/CD Setup for nzq-blog-backend

This document explains how to set up the GitLab CI/CD pipeline for the Spring Boot backend application.

## Prerequisites

1. A GitLab repository with the project code
2. GitLab Runner configured with Docker executor
3. Docker installed on the GitLab Runner (for building Docker images)
4. Access to a Docker registry (GitLab Container Registry, Docker Hub, etc.)

## GitLab Environment Variables

The CI/CD pipeline requires the following environment variables to be set in GitLab:

### For Docker Registry (Required)
- `CI_REGISTRY_USER` - Docker registry username
- `CI_REGISTRY_PASSWORD` - Docker registry password
- `CI_REGISTRY` - Docker registry URL (e.g., `registry.gitlab.com` for GitLab Container Registry)

Note: GitLab automatically sets `CI_REGISTRY_IMAGE` for projects with Container Registry enabled.

### For Deployment (Optional - only needed if using deploy stages)
- `DEPLOY_USER` - SSH username for the deployment server
- `DEPLOY_SERVER` - IP address or hostname of the deployment server
- `DEPLOY_SSH_KEY` - Private SSH key for accessing the deployment server
- `STAGING_USER` - SSH username for the staging server (if using staging)
- `STAGING_SERVER` - IP address or hostname of the staging server
- `STAGING_SSH_KEY` - Private SSH key for accessing the staging server

## Pipeline Stages

The pipeline consists of the following stages:

1. **build** - Compiles the Spring Boot application
2. **test** - Runs unit tests
3. **package** - Creates the executable JAR file
4. **docker-build** - Builds Docker image from the JAR
5. **docker-push** - Pushes Docker image to registry
6. **deploy** - Deploys to production (auto) and staging (manual)

## Trigger Conditions

- **Main/develop branches** - Full pipeline runs (build, test, package, docker-build, docker-push)
- **Merge requests** - Only build and test stages run
- **Tags** - Full pipeline including deployment to production
- **Manual trigger** - Staging deployment requires manual approval

## Customization

### Using Different Docker Registry

If you're not using GitLab Container Registry, update the variables in `.gitlab-ci.yml`:

```yaml
variables:
  DOCKER_REGISTRY: "your-registry-url"  # e.g., "docker.io"
  DOCKER_IMAGE: "your-username/nzq-blog-backend"
```

### Changing Java Version

Update the Maven image in the build, test, and package stages:

```yaml
image: maven:3.8.8-eclipse-temurin-21  # For Java 21
```

### Disabling Deployment

If you don't need automatic deployment, comment out or remove the `deploy` and `deploy-staging` jobs.

## Manual Deployment to Staging

The staging deployment is manual. To trigger it:

1. Go to GitLab CI/CD → Pipelines
2. Find the pipeline for the develop branch
3. Click the play button (▶) next to "deploy-staging"

## Monitoring

- Pipeline status is visible in GitLab CI/CD → Pipelines
- Docker images are stored in the configured registry
- Application logs can be viewed with `docker logs <container-name>`

## Troubleshooting

### Docker Build Fails
- Ensure Docker is installed on the GitLab Runner
- Check Dockerfile syntax
- Verify Maven build works locally

### SSH Deployment Fails
- Verify SSH keys are correctly set in environment variables
- Check firewall settings on deployment server
- Ensure Docker is installed on the deployment server

### Registry Authentication Fails
- Confirm registry credentials are correct
- Check if the registry requires authentication
- Verify network connectivity to the registry

## Local Testing

Test the CI/CD pipeline locally using GitLab Runner:

```bash
# Install GitLab Runner
# Run pipeline validation
gitlab-runner exec docker build
gitlab-runner exec docker test
```

## Additional Resources

- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [GitLab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/)
- [Docker in Docker (dind) Setup](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html)