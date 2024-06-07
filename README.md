# Checks if the version has changed since last release.

Inputs:
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| type | string | true | Suported values [rust]. In future we might extend to other languages, also we could expose a `version_file` and `version_pattern` to be more extensible

Outputs:

| Name | Type | Description |
| ---- | ---- | ----------- |
| changed | bool | If the version has changed
| version | string | The current version in version file
| prev_version | string | Last release version

# Example

```yaml
    - id: check_version
    uses: radumarias/actions/check-version@v1
    with:
      type: rust

    - run: |
        echo "Prev version ${{ steps.check_version.prev_version }}"
        echo "Version ${{ steps.check_version.version }}"
        echo "Version has changed ${{ steps.check_version.changed }}"
```
