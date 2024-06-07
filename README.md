# Checks if the version in `Cargo.toml` has changed since the last release for a Rust project

Useful in cases when you you want to automatically perform additional steps like creating a release and deploying/publishing the app if version is changed.  
Not useful when you create releases manualy and and trigger deploy/publish from the release or manually.

An example of such a workflow could be this:
- on push
    - run tests
        - build AUR image
            - if version changed 
                - create release, attach binaries as artifact, url used for PKGBUILD to distribute the binaries
                    - publish to AUR
                    - publish to crates.io
        - build docker image
            - if version changed push it to Docker Hub

![workflow](https://github.com/radumarias/action-check-version-changed-rust/blob/main/workflow.jpeg?raw=true)

<!--
# Inputs

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| type | string | true | Suported values [rust]. In future we might extend to other languages, also we could expose a `version_file` and `version_pattern` to be more extensible
-->

# Outputs

| Name | Type | Description |
| ---- | ---- | ----------- |
| changed | bool | If the version has changed
| version | string | The current version in version file
| prev_version | string | Last release version

# Example

```yaml
    - id: check_version
      uses: radumarias/action-check-version-changed-rust@v1

    - run: |
        echo "Prev version ${{ steps.check_version.outputs.prev_version }}"
        echo "Version ${{ steps.check_version.outputs.version }}"
        echo "Version has changed ${{ steps.check_version.outputs.changed }}"
```

# Conditional step

```yaml
    - id: check_version
      uses: radumarias/action-check-version-changed-rust@v1

    - name: Execute if version has changed
      if: ${{ steps.check_version.outputs.changed }}
      run: echo "Version has changed from ${{ steps.check_version.outputs.prev_version }} to ${{ steps.check_version.outputs.version }}"
```

# Conditional job

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
      - name: Execute if version has changed
        if: ${{ steps.check_version.outputs.changed }}
        run: echo "Version has changed from ${{ steps.check_version.outputs.prev_version }} to ${{ steps.check_version.outputs.version }}"
```
