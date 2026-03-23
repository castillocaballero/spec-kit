# Extension Manifest Validation

specify-cli validates `extension.yml` manifest files before installing or publishing an extension. A valid manifest must include required fields (`name`, `version`, `description`), declare a supported `schema_version`, and specify a `cli_version` constraint that is compatible with the running specify-cli version. Your task is to write tests that create manifest files with various valid and invalid configurations and verify that the validation logic accepts correct manifests and rejects malformed ones with descriptive errors.

## Capabilities

### Extension Manifest Validation

- A manifest with all required fields and a compatible `cli_version` constraint passes validation [@test](./test_manifest_validation.py)
- A manifest missing a required field (e.g. `description`) fails validation with an error identifying the missing field [@test](./test_manifest_validation.py)
- A manifest with an unrecognised `schema_version` fails validation [@test](./test_manifest_validation.py)
- A manifest whose `cli_version` constraint excludes the running specify-cli version fails validation with a version-incompatibility error [@test](./test_manifest_validation.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def validate_manifest(manifest_path: str) -> None:
    """
    Validate an extension manifest file (extension.yml).

    A valid manifest must contain:
      - name (str, required)
      - version (str, required, semver)
      - description (str, required)
      - schema_version (str, required, must be a supported value)
      - cli_version (str, required, semver range compatible with installed specify-cli)

    Args:
        manifest_path: Absolute or relative path to the extension.yml file.

    Raises:
        ManifestValidationError: If any required field is missing, the
            schema_version is unsupported, or the cli_version constraint
            is incompatible with the installed specify-cli version.
            The exception message identifies which constraint failed.
    """
    ...

def is_manifest_valid(manifest_path: str) -> tuple[bool, str | None]:
    """
    Check whether a manifest is valid without raising.

    Args:
        manifest_path: Path to the extension.yml file.

    Returns:
        (True, None) if valid; (False, reason_string) if invalid.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

specify-cli validates `extension.yml` manifest files for extensions before installation or publication. The package enforces required field presence, schema version compatibility, and CLI version range constraints, and exposes both a raising validator and a non-raising check function.

[@satisfied-by](specify-cli)
