# Extension Listing and Search

specify-cli provides CLI subcommands for discovering and inspecting installed extensions. The `specify extension list` command shows all currently installed extensions, `specify extension search` finds extensions available in configured catalogs by keyword, and `specify extension info` displays detailed metadata for a named extension. Your task is to write tests that invoke these subcommands via subprocess or the public Python API and verify their output under different conditions (extensions installed vs. none installed, keyword matches vs. no matches).

## Capabilities

### Extension Listing and Search

- `specify extension list` outputs an empty result when no extensions are installed [@test](./test_extension_listing.py)
- `specify extension list` outputs the names of all installed extensions when extensions are present [@test](./test_extension_listing.py)
- `specify extension search <keyword>` returns matching extension names from the catalog [@test](./test_extension_listing.py)
- `specify extension info <name>` returns metadata fields (name, version, description) for a known extension [@test](./test_extension_listing.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def list_installed_extensions(config_dir: str) -> list[dict]:
    """
    Return a list of installed extensions from the given config directory.

    Each dict contains at minimum: {"name": str, "version": str}.

    Args:
        config_dir: Path to the specify-cli configuration directory.

    Returns:
        List of extension metadata dicts, empty if none are installed.
    """
    ...

def search_extensions(keyword: str, config_dir: str) -> list[dict]:
    """
    Search available extension catalogs for extensions matching the keyword.

    Args:
        keyword: Search term to match against extension names and descriptions.
        config_dir: Path to the specify-cli configuration directory.

    Returns:
        List of matching extension metadata dicts.
    """
    ...

def get_extension_info(name: str, config_dir: str) -> dict:
    """
    Retrieve detailed metadata for a named extension.

    Args:
        name: The extension identifier.
        config_dir: Path to the specify-cli configuration directory.

    Returns:
        Dict with fields: name, version, description, author, cli_version.

    Raises:
        ExtensionNotFoundError: If no extension with that name is installed.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

specify-cli is a CLI toolkit that manages extensions via `specify extension list`, `specify extension search`, and `specify extension info` subcommands. It exposes equivalent Python APIs for programmatic use in tests.

[@satisfied-by](specify-cli)
