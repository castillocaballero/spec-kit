# Extension Catalog Management

specify-cli allows users to manage extension catalogs — remote registries from which extensions can be discovered and installed. The `specify extension catalog add` command registers a new catalog URL with optional priority and install-permission flags. The `specify extension catalog list` command shows all configured catalogs. The `specify extension catalog remove` command deletes a catalog by name. Your task is to write tests that invoke these commands and verify that catalogs are correctly added with their configured properties, appear in the listing, and can be removed.

## Capabilities

### Extension Catalog Management

- `specify extension catalog add <name> <url>` registers a catalog that subsequently appears in `specify extension catalog list` output [@test](./test_catalog_management.py)
- `specify extension catalog add` with `--priority` and `--allow-install` flags stores those values, visible in the catalog listing [@test](./test_catalog_management.py)
- `specify extension catalog remove <name>` removes the catalog so it no longer appears in the listing [@test](./test_catalog_management.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def add_catalog(
    name: str,
    url: str,
    priority: int = 50,
    allow_install: bool = True,
    config_dir: str = ".",
) -> None:
    """
    Register a new extension catalog.

    Args:
        name: Unique identifier for the catalog.
        url: URL of the catalog registry.
        priority: Numeric priority; lower value = higher precedence. Default 50.
        allow_install: Whether extensions from this catalog may be installed.
        config_dir: Path to the specify-cli configuration directory.
    """
    ...

def list_catalogs(config_dir: str) -> list[dict]:
    """
    Return all configured extension catalogs.

    Each dict contains: {"name": str, "url": str, "priority": int, "allow_install": bool}.

    Args:
        config_dir: Path to the specify-cli configuration directory.

    Returns:
        List of catalog dicts.
    """
    ...

def remove_catalog(name: str, config_dir: str) -> None:
    """
    Remove a catalog by name.

    Args:
        name: Identifier of the catalog to remove.
        config_dir: Path to the specify-cli configuration directory.

    Raises:
        CatalogNotFoundError: If no catalog with that name exists.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

specify-cli provides `specify extension catalog add`, `specify extension catalog list`, and `specify extension catalog remove` subcommands for managing extension registries. It also exposes equivalent Python APIs for use in automated tests.

[@satisfied-by](specify-cli)
