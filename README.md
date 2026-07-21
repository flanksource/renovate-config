# Flanksource Renovate

This repository is the central Renovate control plane for the Flanksource
GitHub organization. It discovers repositories through a GitHub App and creates
version-update pull requests only for dependencies owned by Flanksource.

Consumer repositories do not need a `renovate.json` file. The global
configuration disables onboarding and ignores repository-level Renovate
configuration.

Dependabot remains responsible for security updates. Renovate's GitHub and OSV
vulnerability-alert features are explicitly disabled here.

## How it works

The hourly workflow:

1. Exchanges the GitHub App ID and private key for a one-hour installation
   token covering every repository in the Flanksource organization installation.
2. Runs Renovate with [`renovate-config.js`](./renovate-config.js).
3. Discovers `flanksource/*` repositories accessible to the App.
4. Disables every dependency by default, then enables packages whose package name
   or source repository belongs to Flanksource.

The package-name rules currently cover GitHub Actions, Go modules, npm packages,
Docker Hub images, GHCR images, and Quay images. The source-URL rule also covers
other Renovate datasources when they resolve to a repository under
`github.com/flanksource`.

## GitHub App setup

Create a GitHub App owned by the Flanksource organization. Webhooks and user
authorization are not required. Configure these repository permissions, following
Renovate's GitHub App guidance:

| Permission        | Access         |
| ----------------- | -------------- |
| Administration    | Read           |
| Checks            | Read and write |
| Commit statuses   | Read and write |
| Contents          | Read and write |
| Dependabot alerts | Read           |
| Issues            | Read and write |
| Metadata          | Read           |
| Pull requests     | Read and write |
| Workflows         | Read and write |

Configure **Members: read** under organization permissions. Install the App once
on the Flanksource organization and grant it access to all repositories that
Renovate should monitor.

The workflow reuses these organization secrets, which must be available to this
repository:

- `FLANKSOURCE_APP_ID`: the App's numeric App ID.
- `FLANKSOURCE_APP_SECRET`: the complete PEM private key.

The App and its credentials are managed at organization level. The workflow's
`owner` input deliberately omits a repository list,
which creates a token covering all repositories granted to that installation.

## Operations

The workflow runs hourly at minute 17 and can also be started manually from the
Actions tab. GitHub App installation tokens expire after one hour, so the job has
a 55-minute timeout. If execution approaches that limit as the organization grows,
split repository discovery across multiple jobs or narrower filters so each job
gets its own token.

To add another Flanksource-owned registry or package namespace, add its pattern to
the enabling rules in [`renovate-config.js`](./renovate-config.js). Changes take
effect on the next run; no consumer-repository configuration or sync pull request
is needed.
