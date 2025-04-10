# GitFlow Jenkins Pipeline User Manual

## Introduction

This document provides comprehensive instructions for working with the automated Jenkins CI/CD pipeline designed to support a GitFlow-based development workflow with proper hotfix handling. The pipeline ensures that fixes made to production code are properly propagated to all relevant branches, preventing fixes from being lost in future releases.

## Table of Contents

1. [Branch Naming Conventions](#branch-naming-conventions)
2. [Development Workflow](#development-workflow)
3. [Release Process](#release-process)
4. [Hotfix Process](#hotfix-process)
5. [Completion Tags](#completion-tags)
6. [Merge Conflict Resolution](#merge-conflict-resolution)
7. [Notifications](#notifications)
8. [Pipeline Configuration](#pipeline-configuration)
9. [Troubleshooting](#troubleshooting)

## Branch Naming Conventions

For the automated pipeline to work correctly, adhere to these branch naming conventions:

| Branch Type | Naming Pattern | Example | Purpose |
|-------------|----------------|---------|---------|
| Development | `d{number}` | `d1`, `d2` | Active development work |
| Release | `r{number}` | `r1`, `r2` | Release candidate branches |
| Hotfix | `hotfix/r{number}-{description}` | `hotfix/r1-login-fix` | Fixes for production issues |
| Master | `master` | `master` | Production code |

The number in branch names should correspond between development and release (e.g., `d1` → `r1`).

## Development Workflow

### Starting a New Development Cycle

1. Create a new development branch from master:
   ```bash
   git checkout master
   git pull origin master
   git checkout -b d1
   git push origin d1
   ```

2. Develop features on the development branch (or feature branches that merge to it).

3. When development is complete, tag the branch as complete:
   ```bash
   git checkout d1
   git tag -a d1-complete -m "Marking development branch as complete"
   git push origin d1-complete
   ```

4. The pipeline will automatically create a release branch (`r1`) from this development branch.

## Release Process

### Testing and Finalizing a Release

1. Perform final testing on the automatically created release branch.

2. Make any minor fixes directly to the release branch.

3. When the release is ready for production, tag it as complete:
   ```bash
   git checkout r1
   git tag -a r1-complete -m "Marking release branch as complete"
   git push origin r1-complete
   ```

4. The pipeline will automatically merge the release branch to master and deploy to production.

## Hotfix Process

### Creating and Applying a Hotfix

When an issue is discovered in production:

1. Create a hotfix branch from the appropriate release branch:
   ```bash
   git checkout r1
   git pull origin r1
   git checkout -b hotfix/r1-issue-description
   ```

2. Implement and test the fix.

3. When the hotfix is complete, tag it:
   ```bash
   git checkout hotfix/r1-issue-description
   git tag -a hotfix/r1-issue-description-complete -m "Marking hotfix as complete"
   git push origin hotfix/r1-issue-description-complete
   ```

4. The pipeline will automatically:
   - Merge the hotfix to the release branch
   - If the release branch is tagged complete, merge it to master
   - Propagate the fix to the current development branch (e.g., `d2`)

## Completion Tags

Every branch (except master) requires a completion tag before it can proceed to the next stage:

| Branch Type | Tag Format | Example |
|-------------|------------|---------|
| Development | `{branch-name}-complete` | `d1-complete` |
| Release | `{branch-name}-complete` | `r1-complete` |
| Hotfix | `{branch-name}-complete` | `hotfix/r1-login-fix-complete` |

### Adding a Completion Tag

```bash
git checkout <branch-name>
git tag -a <branch-name>-complete -m "Marking branch as complete"
git push origin <branch-name>-complete
```

If you forget to add a completion tag, the pipeline will send a notification with instructions.

## Merge Conflict Resolution

When the pipeline detects merge conflicts (typically when propagating hotfixes to development branches):

1. A conflict resolution branch is automatically created, named `merge-hotfix-to-dev-{build-number}`.

2. You'll receive an email with details about the conflicts and step-by-step resolution instructions.

3. To resolve the conflicts:
   ```bash
   git checkout merge-hotfix-to-dev-123
   # Resolve conflicts in the files mentioned in the email
   git add <resolved-files>
   git commit -m "Resolve conflicts merging hotfix to dev"
   git push origin merge-hotfix-to-dev-123
   ```

4. Create a pull request from the conflict resolution branch to the target development branch.

5. After review, merge the pull request to complete the hotfix propagation.

## Notifications

The pipeline sends email notifications for important events:

- Branch requiring a completion tag
- New release branch creation
- Successful production deployments
- Hotfix propagation status
- Merge conflicts requiring resolution

Ensure your email is correctly configured in Jenkins to receive these notifications.

## Deployment Rules

The pipeline enforces strict rules about which environments branches can deploy to:

| Branch Type | Deploys To | Approval Required |
|-------------|------------|-------------------|
| Development (`d*`) | DEV environment | Automatic |
| Release (`r*`) | STAGING/QA environment | Automatic |
| Hotfix (`hotfix/*`) | TEST environment | Automatic |
| Master (`master`) | PRODUCTION | **Manual Approval Required** |

### Production Deployment Process

When changes are merged to master, the deployment to production follows this process:

1. The pipeline processes the change and runs all tests
2. An email notification is sent to the team requesting deployment approval
3. A qualified team member must log into Jenkins and approve the deployment
4. Only after explicit approval will the deployment to production proceed

This ensures that:
- No accidental deployments to production can occur
- Changes are reviewed one final time before going to production
- There is a clear record of who approved each production deployment

## Troubleshooting

### Common Issues

#### Pipeline Not Detecting Branch Type

- Verify branch name follows the conventions
- Check regex patterns in the Jenkinsfile match your branch naming

#### Hotfix Not Propagating to Development

- Check for merge conflicts and resolve them
- Verify the correct development branch is detected
- Check Jenkins logs for specific error messages

#### Completion Tag Not Recognized

- Ensure tag is pushed to remote repository
- Verify tag name exactly matches the expected format
- Check permissions for creating tags

### Getting Help

If you encounter issues not covered in this manual:

1. Review the Jenkins build logs for specific error messages
2. Check the email notifications for detailed instructions
3. Consult with your DevOps team for assistance

---

This manual covers the standard workflows supported by the Jenkins pipeline. For advanced usage or custom modifications to the pipeline, consult your DevOps team or refer to the Jenkins and Git documentation.
