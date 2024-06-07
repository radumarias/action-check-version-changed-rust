# Checks if the version has changed since last release.

Inputs:
- **type** (required): suported values [rust]. In future we might extend to other languages, also we could expose a `version_file` and `version_pattern` to be more extensible

Outputs:
- **changed**: if the version has changed
- **version**: the current version in version file
- **prev_version**: what was the previous version

# Example

```yaml
    - id: check_version
    uses: radumarias/actions/check-version@v1
    with:
      type: rust

    - run: |
        prev_version = "${{ steps.check_version.prev_version }}
        version = "${{ steps.check_version.version }}
        changed = ${{ steps.check_version.changed }}
        
        echo "Prev version $prev_version"
        echo "Version $version"
        echo "Version has changed ${{ needs.check_version.changed }}"
```
