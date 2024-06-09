# Checks if the version in `Cargo.toml` has changed since last time the job runned for a Rust project

[![Tests](https://github.com/radumarias/action-check-version-changed-rust/actions/workflows/tests.yml/badge.svg)](https://github.com/radumarias/action-check-version-changed-rust/actions/workflows/tests.yml)

Useful in cases when you you want to automatically perform additional steps like creating a release and deploying/publishing the app if version is changed.  
Not useful when you create releases manualy and trigger deploy/publish from the release or manually.

When the step runs, at the end, it saves in the cache the last commit in repo as `since_commit` and the next time it runs it checks for version change since that commit.

> [!WARNING]  
> **If version is changed and you have 2 workflows that triger on push that inside are using this action, only first one that runs the check version step will see the version change, the second one will NOT, please keep this in mind when you design your workflows.**

When running the step for the first time, as we dont' have `since_commit` saved, it will compare the verison with latest release tag.

An example of such a workflow could be this:
- on `push`
    - run tests
        - build `AUR` image (cargo aur)
            - if version changed 
                - create `release`, attach binaries as `artifact`, url used in `PKGBUILD` to distribute the binaries
                    - publish to `AUR`
                    - publish to `crates.io`
        - build docker image
            - if version changed push it to `Docker Hub`

![workflow](https://github.com/radumarias/action-check-version-changed-rust/blob/main/workflow.jpeg?raw=true)

> [!WARNING]
> **`GITHUB_TOKEN` need to have `Workflow Write permission`. For that got here (change `<OWNER>` and `<REPO>`) [https://github.com/<OWNER>/<REPO>/settings/actions](https://github.com/<OWNER>/<REPO>/settings/actions) and at the bottom in `Workflow permissions` section check `Read and write permissions`.**  
> This is needed because we save the `since_commit` in the cache and it needs this permission to save the value between runs.

<!--
# Inputs

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| type | string | true | Suported values [rust]. In future we might extend to other languages, also we could expose a `version_file` and `version_pattern` to be more extensible
-->

# Usage

## Outputs

| Name | Type | Description |
| ---- | ---- | ----------- |
| changed | bool | If the version has changed
| version | string | The current version in version file
| prev_version | string | Last release version

## Example

```yaml
    - id: check_version
      uses: radumarias/action-check-version-changed-rust@v1

    - run: |
        echo "Prev version ${{ steps.check_version.outputs.prev_version }}"
        echo "Version ${{ steps.check_version.outputs.version }}"
        echo "Version has changed ${{ steps.check_version.outputs.changed }}"
```

## Conditional step

```yaml
    - id: check_version
      uses: radumarias/action-check-version-changed-rust@v1

    - name: Execute if version has changed
      if: ${{ steps.check_version.outputs.changed }}
      run: echo "Version has changed from ${{ steps.check_version.outputs.prev_version }} to ${{ steps.check_version.outputs.version }}"
```

## Conditional job

```yaml
jobs:
  check_version:
    name: Check version
    runs-on: ubuntu-latest
    outputs:
      changed: ${{ steps.check_version.outputs.changed }}
      version: ${{ steps.check_version.outputs.version }}
      prev_version: ${{ steps.check_version.outputs.prev_version }}
      
    steps:
      - uses: actions/checkout@v4

      - id: check_version
        uses: radumarias/action-check-version-changed-rust@v1

  conditional_job:
    name: Conditonal
    needs: [check_version]
    if: ${{ needs.check_version.outputs.changed }}
    runs-on: ubuntu-latest

    steps:
      - name: Version changed
        run: echo "Version has changed from ${{ steps.check_version.outputs.prev_version }} to ${{ steps.check_version.outputs.version }}"
```
