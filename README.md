# balena versionist GitHub action

Automatic versioning of source files with [balena-versionist](https://github.com/product-os/balena-versionist).

Useful as a first step in continuous deployment workflows.

The following actions are taken sequentially:

1. Run `balena-versionist`. This will update a number of files depending on the detected project type.
2. Add a new commit to the working branch with the versioned changes.
3. Create a release tag corresponding to the new version.
4. Upload a compressed artifact of the versioned source.
5. Push changes and tags to the default branch on merge.

Read more about the opinionated versioning here:

- [versionist](https://github.com/product-os/versionist)
- [balena-versionist](https://github.com/product-os/balena-versionist)

## Getting Started

```yaml
name: Versionist
on:
  push:
    branches: ["master", "main"]
  pull_request:
    branches: ["master", "main"]

# https://docs.github.com/en/actions/using-jobs/using-concurrency
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

jobs:
  versionist:
    name: Run versionist
    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.versionist.outputs.version }}
      tag: ${{ steps.versionist.outputs.tag }}
      artifact: ${{ steps.versionist.outputs.artifact }}

    steps:
      - name: Checkout project
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.ACCESS_TOKEN }}
          fetch-depth: 0 # versionist needs all commits and tags

      - name: Run balena-versionist
        id: versionist
        uses: klutchell/balena-versionist-action
        with:
          github_token: ${{ secrets.ACCESS_TOKEN }}
          gpg_private_key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg_passphrase: ${{ secrets.GPG_PRIVATE_KEY }}

  run_tests:
    name: Run tests
    needs: versionist
    runs-on: ubuntu-latest

    steps:
      - name: Download versioned source
        uses: actions/download-artifact@v3
        with:
          name: ${{ needs.versionist.outputs.artifact }}
          path: ${{ runner.temp }}

      - name: Extract versioned source
        shell: pwsh
        run: tar -xvf ${{ runner.temp }}/source.tgz

      - name: Echo version and tag
        run: |
          echo "Version is ${{ needs.versionist.outputs.version }}"
          echo "Tag is ${{ needs.versionist.outputs.tag }}"
```

## Usage

### Inputs

#### `github_token`

Personal access token (PAT) for the GitHub service account with admin/owner permissions.

#### `gpg_private_key`

GPG private key exported with `gpg --armor --export-secret-keys ...` to sign commits.

See [Managing commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification).

#### `gpg_passphrase`

Passphrase to decrypt GPG private key.

See [Managing commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification).

#### `balena_versionist_version`

Use a specific release of [balena-versionist](https://www.npmjs.com/package/balena-versionist).

#### `versionist_version`

Use a specific release of [versionist](https://www.npmjs.com/package/versionist).

### Outputs

#### `version`

Generated version.

Read more about the opinionated versioning here:

- [versionist](https://github.com/product-os/versionist)
- [balena-versionist](https://github.com/product-os/balena-versionist)

#### `tag`

Generated tag. Same as version but with a `v` prefix.

#### `artifact`

Name of the uploaded versioned source artifact.

#### `changed`

Either `true` or `false` depending on whether any source files were updated by versionist.

#### `error`

Error message from versionist, if any.

## Contributing

Please open an issue or submit a pull request with any features, fixes, or changes.
