# About

This repository demonstrates how to use Bytebase and GitHub actions to do database release CI/CD with a code base following [GitLab flow](https://about.gitlab.com/topics/version-control/what-is-gitlab-flow/).

For GitLab flow, there is one production branch and as many as pre-production branches you want. Each of these branches correspond to an environment and merging into a branch triggers the deployment.

For this example, there are "test" and "prod" branches corresponding to the "test" and "prod" environment.

[sql-review.yml](/.github/workflows/sql-review.yml) checks the SQL migration files against the databases when pull requests are created targeting "test" or "prod" branch.

[release.yml](/.github/workflows/release.yml) builds the code, migrate the databases and deploy the code when new commits are pushed to "test" or "prod" branch. Use [environments with protection rules](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment#required-reviewers) to control whether the deployment to an environment requires approval.

## How to configure sql-review.yml

Copy [sql-review.yml](/.github/workflows/sql-review.yml) to your repository.

Modify the environment variables to match your setup.

```yml
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # set GITHUB_TOKEN because the 'Check release' step needs it to comment the pull request with check results.
      BYTEBASE_URL: https://demo.bytebase.com
      BYTEBASE_SERVICE_ACCOUNT: ci@service.bytebase.com
      BYTEBASE_PROJECT: "projects/project-sample"
      BYTEBASE_TARGETS: "instances/test-sample-instance/databases/hr_test" # the database targets to check against.
      FILE_PATTERN: "migrations/*.sql" # the glob pattern matching the migration files.
```

Set your service account password in the repository secrets setting with the name `BYTEBASE_SERVICE_ACCOUNT_SECRET`.

> [!IMPORTANT]
> The migration filename SHOULD comply to the naming scheme described in [bytebase/create-release-action](https://github.com/bytebase/create-release-action/tree/main).

## How to configure release.yml

Copy [release.yml](/.github/workflows/release.yml) to your repository.

Modify the environment variables to match your setup.

```yml
    env:
      BYTEBASE_URL: https://demo.bytebase.com
      BYTEBASE_SERVICE_ACCOUNT: ci@service.bytebase.com
      BYTEBASE_PROJECT: "projects/project-sample"
      BYTEBASE_TARGETS: "instances/test-sample-instance/databases/hr_test" # deploy targets
      FILE_PATTERN: "migrations/*.sql" # the glob pattern matching the migration files.
```

In the repository environments setting, create two environments: "test" and "prod". In the "prod" environment setting, configure "Deployment protection rules", check "Required reviewers" and add reviewers in order to rollout the "prod" environment after approval.

Set your service account password in the repository secrets setting with the name `BYTEBASE_SERVICE_ACCOUNT_SECRET`.

> [!IMPORTANT]
> The migration filename SHOULD comply to the naming scheme described in [bytebase/create-release-action](https://github.com/bytebase/create-release-action/tree/main).

## Preview

Branches:

- [Test branch](https://github.com/bytebase/example-gitops-gitlab-flow/tree/test)
- [Prod branch](https://github.com/bytebase/example-gitops-gitlab-flow/tree/prod)

Pull requests to the test and prod branch:

- https://github.com/bytebase/example-gitops-gitlab-flow/pull/1
- https://github.com/bytebase/example-gitops-gitlab-flow/pull/4

GitHub actions that deploy to the test and prod:

- https://github.com/bytebase/example-gitops-gitlab-flow/actions/runs/13831088906
- https://github.com/bytebase/example-gitops-gitlab-flow/actions/runs/13831106877