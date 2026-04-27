# Set Commit Status

A GitHub Action that sets a [commit status](https://docs.github.com/en/rest/commits/statuses) on any commit in any repository.

## Usage

```yaml
- uses: 8bitdogs/set-commit-status@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    state: success
    description: All checks passed
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | GitHub token with `repo:status` permission | **Yes** | — |
| `repository` | Repository in `owner/repo` format | No | _(current repository)_ |
| `sha` | The commit SHA to set the status on | No | _(current commit SHA)_ |
| `state` | Status state: `error`, `failure`, `pending`, `success` | No | `pending` |
| `context` | Label used to differentiate this status from others | No | `set-commit-status` |
| `description` | Short description of the status | No | _(empty)_ |

## Examples

### Mark a commit as pending at the start of a job

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Set pending status
        uses: 8bitdogs/set-commit-status@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          state: pending
          context: ci/build
          description: Build in progress…

      - name: Build
        run: make build
```

### Set status on a commit in another repository

```yaml
- uses: 8bitdogs/set-commit-status@v1
  with:
    token: ${{ secrets.CROSS_REPO_TOKEN }}
    repository: my-org/other-repo
    sha: ${{ steps.get-sha.outputs.sha }}
    state: success
    context: ci/integration
    description: Integration tests passed
```

### Use in a matrix to report per-job status

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        suite: [unit, integration, e2e]
    steps:
      - uses: actions/checkout@v4

      - name: Pending
        uses: 8bitdogs/set-commit-status@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          state: pending
          context: ci/${{ matrix.suite }}

      - name: Run tests
        id: tests
        run: make test-${{ matrix.suite }}

      - name: Report result
        if: always()
        uses: 8bitdogs/set-commit-status@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          state: ${{ steps.tests.outcome == 'success' && 'success' || 'failure' }}
          context: ci/${{ matrix.suite }}
          description: ${{ steps.tests.outcome }}
```

## Permissions

The `token` must have the `repo:status` scope (or `statuses: write` for fine-grained tokens).
When using the built-in `GITHUB_TOKEN`, ensure the workflow has the correct permission:

```yaml
permissions:
  statuses: write
```

## License

[MIT](LICENSE)
