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
            - on new version push it to DOcker Hub

![workflow](https://github.com/radumarias/action-check-version-changed-rust/blob/ec5403ac571af1979586a79152190aeadf237586/workflow.jpeg?raw=true)

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
    uses: radumarias/actions/check-version@v1

    - run: |
        echo "Prev version ${{ steps.check_version.prev_version }}"
        echo "Version ${{ steps.check_version.version }}"
        echo "Version has changed ${{ steps.check_version.changed }}"
```
