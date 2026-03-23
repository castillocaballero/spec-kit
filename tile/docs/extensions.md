# Extension Management

The `specify_cli.extensions` module manages the full lifecycle of spec-kit extensions: installation, removal, updates, catalog search, hook execution, and layered configuration.

## Module

```python
from specify_cli.extensions import (
    ExtensionError, ValidationError, CompatibilityError,
    CatalogEntry, ExtensionManifest, ExtensionRegistry,
    ExtensionManager, ExtensionCatalog, ConfigManager, HookExecutor,
    CommandRegistrar,  # extension-specific wrapper
    normalize_priority, version_satisfies,
)
```

## Types

```python { .api }
from dataclasses import dataclass
from typing import Any, Dict, List, Optional
from pathlib import Path

@dataclass
class CatalogEntry:
    url: str              # Catalog URL
    name: str             # Catalog display name
    priority: int         # Sort order; lower = higher precedence
    install_allowed: bool # Whether extension installation from this catalog is permitted
    description: str = "" # Optional catalog description
```

## Capabilities

### Exceptions

```python { .api }
class ExtensionError(Exception):
    """Base exception for all extension-related errors."""

class ValidationError(ExtensionError):
    """Raised when extension manifest (extension.yml) validation fails."""

class CompatibilityError(ExtensionError):
    """Raised when extension is incompatible with the current spec-kit version."""
```

### Utility Functions

```python { .api }
def normalize_priority(value: Any, default: int = 10) -> int:
    """
    Normalize a stored priority value for sorting and display.

    Handles corrupted/missing/non-numeric registry values gracefully.

    Args:
        value: Priority value (may be int, str, None, etc.)
        default: Fallback priority for invalid values (default: 10).

    Returns:
        int: Normalized priority >= 1.
    """

def version_satisfies(current: str, required: str) -> bool:
    """
    Check if a version satisfies a version specifier.

    Args:
        current: Current version string (e.g. "0.3.2").
        required: PEP 440 version specifier (e.g. ">=0.1.0,<2.0.0").

    Returns:
        bool: True if current satisfies required.
    """
```

### ExtensionManifest

Represents and validates an extension's `extension.yml` manifest file.

#### Manifest YAML Format

```yaml
schema_version: "1.0"
extension:
  id: my-extension          # lowercase alphanumeric + hyphens
  name: My Extension
  version: 1.0.0
  description: Extension description
  author: author-name
requires:
  speckit_version: ">=0.3.0"   # PEP 440 specifier
provides:
  commands:
    - name: speckit.my-extension.cmd  # pattern: speckit.<ext>.<cmd>
      file: commands/cmd.md
```

```python { .api }
class ExtensionManifest:
    SCHEMA_VERSION: str = "1.0"
    REQUIRED_FIELDS: List[str] = ["schema_version", "extension", "requires", "provides"]

    def __init__(self, manifest_path: Path) -> None:
        """
        Load and validate extension manifest from file.

        Args:
            manifest_path: Path to extension.yml file.

        Raises:
            ValidationError: If manifest is missing required fields,
                has invalid schema version, or contains invalid data.
        """

    @property
    def id(self) -> str:
        """Extension ID (from manifest 'extension.id' field)."""

    @property
    def name(self) -> str:
        """Extension display name."""

    @property
    def version(self) -> str:
        """Extension version string."""

    @property
    def description(self) -> str:
        """Extension description."""

    @property
    def requires_speckit_version(self) -> str:
        """Required spec-kit version specifier (e.g. ">=0.3.0")."""

    @property
    def commands(self) -> List[Dict[str, Any]]:
        """
        List of command dicts provided by this extension.
        Each dict: {"name": str, "file": str, "aliases": List[str] (optional)}
        """

    @property
    def hooks(self) -> Dict[str, Any]:
        """Hook definitions dict from manifest."""

    def get_hash(self) -> str:
        """Calculate SHA256 hash of manifest file content. Returns "sha256:<hex>" string."""
```

### ExtensionRegistry

Low-level JSON-backed registry for installed extensions. State persisted in `.specify/extensions/`.

```python { .api }
class ExtensionRegistry:
    def __init__(self, extensions_dir: Path) -> None:
        """
        Args:
            extensions_dir: Path to .specify/extensions/ directory.
        """

    def add(self, extension_id: str, metadata: dict) -> None:
        """Add a new extension entry. Raises if already exists."""

    def update(self, extension_id: str, metadata: dict) -> None:
        """
        Update extension metadata. Merges with existing; preserves 'installed_at'.
        Raises if extension not found.
        """

    def restore(self, extension_id: str, metadata: dict) -> None:
        """
        Restore exact metadata (replaces entirely). Used for rollback.
        """

    def remove(self, extension_id: str) -> None:
        """Remove extension from registry. Raises if not found."""

    def get(self, extension_id: str) -> Optional[dict]:
        """
        Get extension metadata as a deep copy.
        Returns None if not found.
        """

    def list(self) -> Dict[str, dict]:
        """
        Get all extensions with valid metadata as deep copies.
        Skips corrupted entries.
        """

    def keys(self) -> set:
        """Get all extension IDs, including entries with corrupted metadata."""

    def is_installed(self, extension_id: str) -> bool:
        """Return True if extension_id is in the registry."""

    def list_by_priority(self, include_disabled: bool = False) -> List[tuple]:
        """
        Get extensions sorted by priority (ascending; lower = higher precedence).

        Args:
            include_disabled: If False (default), exclude disabled extensions.
                An extension is disabled when its metadata has `enabled: False`
                (checked via meta.get("enabled", True)).

        Returns:
            List[tuple]: [(extension_id, metadata_dict), ...]
        """
```

### ExtensionManager

High-level API for extension lifecycle management. Handles installation from ZIP or directory, removal, and listing.

```python { .api }
class ExtensionManager:
    def __init__(self, project_root: Path) -> None:
        """
        Args:
            project_root: Root directory of the spec-kit project.
        """

    def check_compatibility(
        self,
        manifest: ExtensionManifest,
        speckit_version: str
    ) -> bool:
        """
        Check if extension is compatible with the current spec-kit version.

        Args:
            manifest: Extension manifest to check.
            speckit_version: Current spec-kit version string.

        Returns:
            bool: True if compatible.

        Raises:
            CompatibilityError: If extension requires a different spec-kit version.
        """

    def install_from_directory(
        self,
        source_dir: Path,
        speckit_version: str,
        register_commands: bool = True,
        priority: int = 10
    ) -> ExtensionManifest:
        """
        Install extension from a directory containing extension.yml.

        Args:
            source_dir: Directory containing extension.yml and command files.
            speckit_version: Current spec-kit version (for compatibility check).
            register_commands: If True, register commands with detected agents.
            priority: Sort priority (lower = higher precedence). Default: 10.

        Returns:
            ExtensionManifest: Installed extension manifest.

        Raises:
            ValidationError: If manifest is invalid.
            CompatibilityError: If version incompatible.
        """

    def install_from_zip(
        self,
        zip_path: Path,
        speckit_version: str,
        priority: int = 10
    ) -> ExtensionManifest:
        """
        Install extension from a ZIP archive.

        Args:
            zip_path: Path to extension ZIP file.
            speckit_version: Current spec-kit version.
            priority: Sort priority. Default: 10.

        Returns:
            ExtensionManifest: Installed extension manifest.

        Raises:
            ValidationError: If manifest is invalid.
            CompatibilityError: If version incompatible.
        """

    def remove(self, extension_id: str, keep_config: bool = False) -> bool:
        """
        Remove an installed extension.

        Args:
            extension_id: Extension ID to remove.
            keep_config: If True, preserve extension configuration files.

        Returns:
            bool: True if removed, False if extension was not found.
        """

    def list_installed(self) -> List[Dict[str, Any]]:
        """
        List all installed extensions with their metadata.

        Returns:
            List[Dict[str, Any]]: List of metadata dicts, each containing
                id, name, version, description, priority, installed_at, enabled, etc.
        """

    def get_extension(self, extension_id: str) -> Optional[ExtensionManifest]:
        """
        Get manifest for an installed extension.

        Args:
            extension_id: Extension ID.

        Returns:
            ExtensionManifest or None if not found.
        """
```

### ExtensionCatalog

Manages extension catalog fetching from GitHub, caching, and searching.

```python { .api }
class ExtensionCatalog:
    DEFAULT_CATALOG_URL: str
    # "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.json"

    COMMUNITY_CATALOG_URL: str
    # "https://raw.githubusercontent.com/github/spec-kit/main/extensions/catalog.community.json"

    CACHE_DURATION: int  # 3600 seconds (1 hour)

    def __init__(self, project_root: Path) -> None

    def get_active_catalogs(self) -> List[CatalogEntry]:
        """Get ordered list of active catalogs (sorted by priority ascending)."""

    def get_catalog_url(self) -> str:
        """Get primary catalog URL (backward compatibility helper)."""

    def is_cache_valid(self) -> bool:
        """True if cached catalog data is within CACHE_DURATION."""

    def fetch_catalog(self, force_refresh: bool = False) -> Dict[str, Any]:
        """
        Fetch and merge extensions from all active catalogs.

        Args:
            force_refresh: If True, bypass cache and fetch fresh data.

        Returns:
            Dict[str, Any]: Merged catalog data dict.
        """

    def search(
        self,
        query: str = None,
        tag: str = None,
        author: str = None,
        verified_only: bool = False
    ) -> List[Dict[str, Any]]:
        """
        Search extensions in the catalog.

        Args:
            query: Text to match against name, description, or ID.
            tag: Filter by tag (exact match).
            author: Filter by author name.
            verified_only: If True, return only verified extensions.

        Returns:
            List[Dict[str, Any]]: Matching extension info dicts.
        """

    def get_extension_info(self, extension_id: str) -> Optional[Dict[str, Any]]:
        """
        Get complete extension info from catalog.

        Args:
            extension_id: Extension ID to look up.

        Returns:
            Dict[str, Any] or None if not found.
        """

    def download_extension(
        self,
        extension_id: str,
        target_dir: Optional[Path] = None
    ) -> Path:
        """
        Download extension ZIP from catalog.

        Args:
            extension_id: Extension ID to download.
            target_dir: Directory to save ZIP (uses temp dir if None).

        Returns:
            Path: Path to the downloaded ZIP file.
        """

    def clear_cache(self) -> None:
        """Clear all cached catalog files."""
```

### CommandRegistrar (extensions module)

Extension-specific wrapper around `agents.CommandRegistrar`. Used internally by `ExtensionManager`.

```python { .api }
class CommandRegistrar:
    def register_commands_for_agent(
        self,
        agent_name: str,
        manifest: ExtensionManifest,
        extension_dir: Path,
        project_root: Path
    ) -> List[str]:
        """Register extension commands for a specific agent."""

    def register_commands_for_all_agents(
        self,
        manifest: ExtensionManifest,
        extension_dir: Path,
        project_root: Path
    ) -> Dict[str, List[str]]:
        """Register extension commands for all detected agents."""

    def unregister_commands(
        self,
        registered_commands: Dict[str, List[str]],
        project_root: Path
    ) -> None:
        """Remove previously registered command files."""

    def register_commands_for_claude(
        self,
        manifest: ExtensionManifest,
        extension_dir: Path,
        project_root: Path
    ) -> List[str]:
        """Register commands specifically for Claude Code."""
```

### ConfigManager

Manages layered extension configuration with dotted-key access.

```python { .api }
class ConfigManager:
    def __init__(self, project_root: Path, extension_id: str) -> None:
        """
        Args:
            project_root: Project root directory.
            extension_id: Extension ID for which to manage config.
        """

    def get_config(self) -> Dict[str, Any]:
        """
        Get merged configuration from all config layers.

        Returns:
            Dict[str, Any]: Merged config dict.
        """

    def get_value(self, key_path: str, default: Any = None) -> Any:
        """
        Get a config value by dotted key path.

        Args:
            key_path: Dotted path (e.g. "section.subsection.key").
            default: Value to return if key not found.

        Returns:
            Value at key_path or default.
        """

    def has_value(self, key_path: str) -> bool:
        """
        Check if a value exists at the given dotted key path.

        Args:
            key_path: Dotted path (e.g. "section.key").

        Returns:
            bool: True if value exists.
        """
```

### HookExecutor

Executes and manages extension hooks triggered by spec-kit lifecycle events.

```python { .api }
class HookExecutor:
    def __init__(self, project_root: Path) -> None

    def get_project_config(self) -> Dict[str, Any]:
        """Read extensions.yml project config. Returns {} if missing."""

    def save_project_config(self, config: Dict[str, Any]) -> None:
        """Write extensions.yml project config."""

    def register_hooks(self, manifest: ExtensionManifest) -> None:
        """Register hooks from an extension manifest into extensions.yml."""

    def unregister_hooks(self, extension_id: str) -> None:
        """Remove all hooks belonging to an extension from extensions.yml."""

    def get_hooks_for_event(self, event_name: str) -> List[Dict[str, Any]]:
        """
        Get all enabled hooks registered for an event.

        Args:
            event_name: Event identifier (e.g. "post-init", "pre-generate").

        Returns:
            List[Dict[str, Any]]: Enabled hook dicts for the event.
        """

    def should_execute_hook(self, hook: Dict[str, Any]) -> bool:
        """
        Determine if a hook should be executed.

        Returns:
            bool: True if hook is enabled and all conditions are met.
        """

    def format_hook_message(
        self,
        event_name: str,
        hooks: List[Dict[str, Any]]
    ) -> str:
        """Format a human-readable hook listing message for an event."""

    def check_hooks_for_event(self, event_name: str) -> Dict[str, Any]:
        """
        Check and gather hooks for an event without executing them.

        Returns:
            Dict[str, Any]: Results dict with hook info.
        """

    def execute_hook(self, hook: Dict[str, Any]) -> Dict[str, Any]:
        """
        Return execution information for a single hook (delegates to AI agent).

        Note: Hooks are not directly shell-executed; execution is delegated to the
        AI agent. This method returns structured info describing how to execute.

        Args:
            hook: Hook dict from extensions.yml.

        Returns:
            Dict[str, Any]: Info dict with keys:
                command: str          # Command name to run (e.g. "speckit.my-ext.cmd")
                extension: str        # Extension ID
                optional: bool        # Whether hook execution is optional
                description: str      # Hook description
                prompt: str           # Hook prompt text
        """

    def enable_hooks(self, extension_id: str) -> None:
        """Enable all hooks for an extension in extensions.yml."""

    def disable_hooks(self, extension_id: str) -> None:
        """Disable all hooks for an extension in extensions.yml."""
```

## Usage Examples

### Install extension from catalog

```python
from pathlib import Path
from specify_cli.extensions import ExtensionManager, ExtensionCatalog
from specify_cli import get_speckit_version

project_root = Path(".")
catalog = ExtensionCatalog(project_root)
manager = ExtensionManager(project_root)

# Search for an extension
results = catalog.search(query="testing")
for ext in results:
    print(ext["id"], ext.get("description", ""))

# Install an extension
if results:
    zip_path = catalog.download_extension(results[0]["id"])
    manifest = manager.install_from_zip(zip_path, get_speckit_version())
    print(f"Installed: {manifest.id} v{manifest.version}")
```

### List and remove extensions

```python
from pathlib import Path
from specify_cli.extensions import ExtensionManager

manager = ExtensionManager(Path("."))

installed = manager.list_installed()
for ext in installed:
    print(f"{ext['id']} v{ext['version']} (priority: {ext.get('priority', 10)})")

# Remove by ID
removed = manager.remove("my-extension-id")
print("Removed:", removed)
```

### Use ConfigManager

```python
from pathlib import Path
from specify_cli.extensions import ConfigManager

config = ConfigManager(Path("."), "my-extension")
api_key = config.get_value("credentials.api_key", default="")
if config.has_value("feature_flags.experimental"):
    flags = config.get_value("feature_flags")
```

### Execute hooks

```python
from pathlib import Path
from specify_cli.extensions import HookExecutor

executor = HookExecutor(Path("."))

# Get hooks for an event
hooks = executor.get_hooks_for_event("post-init")
for hook in hooks:
    result = executor.execute_hook(hook)
    if not result["success"]:
        msg = executor.format_hook_message(hook, error=result.get("error"))
        print(f"Hook failed: {msg}")
```
