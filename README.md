# ci-workflows

Public home for reusable GitHub Actions workflows used across `BrandonTorres` repositories.

## Why this repository is public

GitHub Actions does not allow a reusable workflow (`workflow_call`) hosted in a **private** repository under a personal account to be called from another repository. Keeping these generic workflows in a public repository allows any repository to reference them cleanly while keeping deployment configuration, caller templates, and onboarding documentation in the private `deploy-center` repository.

## Workflows

### `build-image.yml` — Build and Publish Container Image

A reusable `workflow_call` workflow that checks out the caller repository, sets up Docker Buildx, optionally logs in to GitHub Container Registry (`ghcr.io`), extracts metadata tags, and builds (and pushes on default branches) the container image.

#### Push-gate behavior

- **Pull requests (`pull_request`, `pull_request_target`) and non-default branches:** Builds the image to validate the Dockerfile without pushing to GHCR.
- **Default branch (`main`, `master`, or `github.event.repository.default_branch`):** Logs in to `ghcr.io` with `GITHUB_TOKEN` and pushes the image.

#### Permissions

Callers must grant the job:

- `contents: read`
- `packages: write`

#### Inputs

All inputs are optional strings:

| Input | Default | Description |
| :--- | :--- | :--- |
| `image-name` | `""` | GHCR image path under `ghcr.io/` (defaults to `github.repository` lowercased). |
| `dockerfile` | `"Dockerfile"` | Path to `Dockerfile` relative to the repository root. |
| `context` | `"."` | Docker build context path. |
| `build-args` | `""` | Additional newline-separated Docker build-args (`APP_SHA` is always passed). |
| `platforms` | `"linux/amd64"` | Target platforms for `buildx`. |

#### Outputs & image tags

The workflow produces the following tags on `ghcr.io/<image-name || github.repository>`:

- `sha-<short-sha>` — short commit SHA tag
- `<branch>` — branch ref tag (`type=ref,event=branch`)

Workflow outputs:

| Output | Description |
| :--- | :--- |
| `digest` | Built image digest |
| `tags` | Image tags produced |

## Pinning a version

Callers should pin to an immutable semver release tag rather than `@main`:

```yaml
jobs:
  build:
    uses: BrandonTorres/ci-workflows/.github/workflows/build-image.yml@v1.0.0
    permissions:
      contents: read
      packages: write
```

## Releasing a new version

1. Open a pull request against `main` and merge via squash merge once reviewed.
2. After merge, a maintainer cuts a semver tag (for example, `v1.0.0`) on `main`.
3. Callers update their `uses:` reference to the new tag.

## Caller template & onboarding

For the full caller workflow template and project onboarding runbook, see the private `BrandonTorres/deploy-center` repository.
