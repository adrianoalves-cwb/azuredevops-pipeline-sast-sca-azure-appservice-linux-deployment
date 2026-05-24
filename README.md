# Application Pipelines

> This pipeline deploys to Azure Web App (App Services) hosted on Linux.

This repository contains Azure DevOps YAML pipelines for building and deploying the application.

<<<<<<< HEAD
## Architecture

The delivery architecture is split into CI and CD pipelines. The CI pipeline builds, tests, scans, and publishes artifacts. The CD pipeline consumes those artifacts and deploys them to Azure Web App (Linux) environments for QA and PROD using a shared deployment template.

## Business Problem

Manual deployments and inconsistent release steps increase lead time and create avoidable production risk. This repository standardizes delivery so every change follows the same quality, security, and deployment gates.

## Operational Impact

The pipelines improve deployment reliability, reduce manual intervention, and provide repeatable release behavior across environments. Built-in checks (tests, quality scans, and security scans) reduce late-stage defects and increase confidence in releases.

## Scalability

The template-based approach allows reuse across multiple services and environments. Parameterized deployment steps and branch-based variable groups make it easier to scale this pattern to additional apps, teams, and release tracks.

## Governance

Governance is enforced through codified pipeline definitions, branch-driven release rules, variable groups, and quality/security scanning stages. This creates an auditable and versioned delivery process aligned with enterprise controls.

## Workflow

1. Developer pushes code to `feature/*`, `qa`, or `main`.
2. CI runs build, tests, and analysis/scanning stages.
3. CI publishes build artifacts.
4. CD consumes artifacts and deploys to QA or PROD based on branch conditions.
5. SoundCheck runs post-deployment validation.

=======
>>>>>>> 9de43953284848f5682270eb382487ef19f16c03
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
