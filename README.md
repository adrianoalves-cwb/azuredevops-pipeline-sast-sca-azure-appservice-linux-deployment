# Application Pipelines

> This pipeline deploys to Azure Web App (App Services) hosted on Linux.

This repository contains Azure DevOps YAML pipelines for building and deploying the application.

## Files

- `CI-azure-pipelines.yml`: Continuous Integration pipeline (build, test, code quality/security checks, and soundcheck).
- `CD-azure-pipelines.yaml`: Continuous Deployment pipeline (deploy to QA/PROD and run deployment soundcheck).
- `templates/deploy-azure-template.yml`: Shared deployment steps template used by CD stages.

## CI Pipeline (`CI-azure-pipelines.yml`)

### Triggers

Runs on:
- `main`
- `qa`
- `feature/**`

### Stages

1. **Build_Publish**
   - Restores dependencies
   - Runs tests with coverage
   - Publishes API artifacts

2. **SonarQube**
   - Runs static analysis via template

3. **Sonartype**
   - Runs SCA (Nexus IQ) scan via template

4. **SoundCheck**
   - Runs platform sound checks

### Branch-specific variable groups

- `main` -> `application-prod`
- `qa` -> `application-qa`

## CD Pipeline (`CD-azure-pipelines.yaml`)

### Trigger

- `trigger: none` (manual or pipeline resource driven)
- Consumes artifacts from upstream pipeline source `\application\CI-application-api`

### Deployment stages

1. **QA**
   - Condition: source branch is `refs/heads/qa`
   - Deploys to `application-qa` environment
   - Uses local template `templates/deploy-azure-template.yml`

2. **PROD**
   - Condition: source branch is `refs/heads/main`
   - Deploys to `application-prod` environment
   - Uses local template `templates/deploy-azure-template.yml`

3. **SoundCheck**
   - Runs post-deployment checks

## Deployment Template (`templates/deploy-azure-template.yml`)

This template centralizes reusable deployment steps:
- Downloads build artifacts from the upstream build pipeline
- Optionally transforms appsettings.json (`fileTransform`)
- Optionally runs token replacement (`replaceTokens`)
- Adds a temporary App Service access restriction for the build agent
- Deploys artifacts with `AzureRmWebAppDeployment@4`
- Removes the temporary access restriction after deployment

### Main parameters

- `buildPipeline`
- `artifactName` (default: `drop`)
- `azureSubscription`
- `appType`
- `resourceGroup`
- `appName`
- `aspnetcoreEnvironment`
- `fileTransform`
- `replaceTokens`

## External template repositories

Both pipelines reference:
- GitHub templates repository (`templates`)
- Azure DevOps templates repository (`devsecops-templates`)

In addition, the CD pipeline uses the local repository template file `templates/deploy-azure-template.yml`.

Ensure service connections and repository permissions are configured in your Azure DevOps project.

## Required variable groups and secrets

Expected variable groups include:
- `SecurityTools`
- `Sonatype` (CI)
- `application-qa`
- `application-prod`

Also ensure required secret variables (for Azure AD, monitoring service principal, App Insights, etc.) are present in variable groups or key vault integration.

## Sanitized placeholders

This repository currently uses sanitized/fake placeholders in some fields (for example org names and URLs). Replace them with real values before using in production.
