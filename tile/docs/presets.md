# Preset Management & Template Resolution

The `specify_cli.presets` module manages the full lifecycle of spec-kit presets: installation, removal, catalog search, and priority-based template resolution.

## Module

```python
from specify_cli.presets import (
    PresetError, PresetValidationError, PresetCompatibilityError,
    PresetCatalogEntry, PresetManifest, PresetRegistry,
    PresetManager, PresetCatalog, PresetResolver,
    VALID_PRESET_TEMPLATE_TYPES,
)
```

## Types

```python { .api }
from dataclasses import dataclass
from typing import Any, Dict, List, Optional
from pathlib import Path

@dataclass
class PresetCatalogEntry:
    url: str              # Catalog URL
    name: str             # Catalog display name
    priority: int         # Sort order; lower = higher precedence
    install_allowed: bool # Whether preset installation from this catalog is permitted
    description: str = "" # Optional catalog description

VALID_PRESET_TEMPLATE_TYPES: set = {"template", "command", "script"}
```

## Capabilities

### Exceptions

```python { .api }
class PresetError(Exception):
    """Base exception for all preset-related errors."""

class PresetValidationError(PresetError):
    """Raised when preset manifest (preset.yml) validation fails."""

class PresetCompatibilityError(PresetError):
    """Raised when preset is incompatible with the current spec-kit version."""
```

### PresetManifest

Represents and validates a preset's `preset.yml` manifest file.

#### Manifest YAML Format

```yaml
schema_version: "1.0"
preset:
  id: my-preset           # lowercase alphanumeric + hyphens
  name: My Preset
  version: 1.0.0
  description: Preset description
  author: author-name
requires:
  speckit_version: ">=0.3.0"   # PEP 440 specifier
provides:
  templates:
    - type: template            # "template", "command", or "script"
      name: spec-template       # lowercase alphanumeric + hyphens (or dots for commands)
      file: templates/spec-template.md
tags:                           # Top-level field (not inside preset section)
  - tag1
  - tag2
```

```python { .api }
class PresetManifest:
    SCHEMA_VERSION: str = "1.0"
    REQUIRED_FIELDS: List[str] = ["schema_version", "preset", "requires", "provides"]

    def __init__(self, manifest_path: Path) -> None:
        """
        Load and validate preset manifest from file.

        Args:
            manifest_path: Path to preset.yml file.

        Raises:
            PresetValidationError: If manifest is missing required fields,
                has invalid schema version, or contains invalid data.
        """

    @property
    def id(self) -> str:
        """Preset ID (from manifest 'preset.id' field)."""

    @property
    def name(self) -> str:
        """Preset display name."""

    @property
    def version(self) -> str:
        """Preset version string."""

    @property
    def description(self) -> str:
        """Preset description."""

    @property
    def author(self) -> str:
        """Preset author name."""

    @property
    def requires_speckit_version(self) -> str:
        """Required spec-kit version specifier (e.g. ">=0.3.0")."""

    @property
    def templates(self) -> List[Dict[str, Any]]:
        """
        List of template dicts provided by this preset.
        Each dict: {"name": str, "file": str, "type": "template"|"command"|"script"}
        """

    @property
    def tags(self) -> List[str]:
        """List of tag strings for catalog search."""

    def get_hash(self) -> str:
        """Calculate SHA256 hash of manifest file content. Returns "sha256:<hex>" string."""
```

### PresetRegistry

Low-level JSON-backed registry for installed presets. State persisted in `.specify/presets/`.

```python { .api }
class PresetRegistry:
    def __init__(self, packs_dir: Path) -> None:
        """
        Args:
            packs_dir: Path to .specify/presets/ directory.
        """

    def add(self, pack_id: str, metadata: dict) -> None:
        """Add a new preset entry."""

    def update(self, pack_id: str, updates: dict) -> None:
        """Update preset metadata (merges with existing)."""

    def restore(self, pack_id: str, metadata: dict) -> None:
        """Restore exact metadata (replaces entirely). Used for rollback."""

    def remove(self, pack_id: str) -> None:
        """Remove preset from registry."""

    def get(self, pack_id: str) -> Optional[dict]:
        """Get preset metadata as deep copy. Returns None if not found."""

    def list(self) -> Dict[str, dict]:
        """Get all presets with valid metadata as deep copies."""

    def keys(self) -> set:
        """Get all preset IDs including corrupted entries."""

    def is_installed(self, pack_id: str) -> bool:
        """Return True if pack_id is in the registry."""

    def list_by_priority(self, include_disabled: bool = False) -> List[tuple]:
        """
        Get presets sorted by priority (ascending; lower = higher precedence).

        Args:
            include_disabled: If False (default), exclude disabled presets.
                A preset is disabled when its metadata has `enabled: False`
                (checked via meta.get("enabled", True)).

        Returns:
            List[tuple]: [(pack_id, metadata_dict), ...]
        """
```

### PresetManager

High-level API for preset lifecycle management.

```python { .api }
class PresetManager:
    def __init__(self, project_root: Path) -> None:
        """
        Args:
            project_root: Root directory of the spec-kit project.
        """

    def check_compatibility(
        self,
        manifest: PresetManifest,
        speckit_version: str
    ) -> bool:
        """
        Check if preset is compatible with the current spec-kit version.

        Args:
            manifest: Preset manifest to check.
            speckit_version: Current spec-kit version string.

        Returns:
            bool: True if compatible.

        Raises:
            PresetCompatibilityError: If preset requires a different version.
        """

    def install_from_directory(
        self,
        source_dir: Path,
        speckit_version: str,
        priority: int = 10
    ) -> PresetManifest:
        """
        Install preset from a directory containing preset.yml.

        Args:
            source_dir: Directory containing preset.yml and template files.
            speckit_version: Current spec-kit version (for compatibility check).
            priority: Sort priority (lower = higher precedence). Default: 10.

        Returns:
            PresetManifest: Installed preset manifest.

        Raises:
            PresetValidationError: If manifest is invalid.
            PresetCompatibilityError: If version incompatible.
        """

    def install_from_zip(
        self,
        zip_path: Path,
        speckit_version: str,
        priority: int = 10
    ) -> PresetManifest:
        """
        Install preset from a ZIP archive.

        Args:
            zip_path: Path to preset ZIP file.
            speckit_version: Current spec-kit version.
            priority: Sort priority. Default: 10.

        Returns:
            PresetManifest: Installed preset manifest.

        Raises:
            PresetValidationError: If manifest is invalid.
            PresetCompatibilityError: If version incompatible.
        """

    def remove(self, pack_id: str) -> bool:
        """
        Remove an installed preset.

        Args:
            pack_id: Preset ID to remove.

        Returns:
            bool: True if removed, False if preset was not found.
        """

    def list_installed(self) -> List[Dict[str, Any]]:
        """
        List all installed presets with their metadata.

        Returns:
            List[Dict[str, Any]]: List of metadata dicts, each containing
                id, name, version, description, priority, installed_at, enabled, author, tags, etc.
        """

    def get_pack(self, pack_id: str) -> Optional[PresetManifest]:
        """
        Get manifest for an installed preset.

        Args:
            pack_id: Preset ID.

        Returns:
            PresetManifest or None if not found.
        """
```

### PresetCatalog

Manages preset catalog fetching from GitHub, caching, and searching.

```python { .api }
class PresetCatalog:
    DEFAULT_CATALOG_URL: str
    # "https://raw.githubusercontent.com/github/spec-kit/main/presets/catalog.json"

    COMMUNITY_CATALOG_URL: str
    # "https://raw.githubusercontent.com/github/spec-kit/main/presets/catalog.community.json"

    CACHE_DURATION: int  # 3600 seconds (1 hour)

    def __init__(self, project_root: Path) -> None

    def get_active_catalogs(self) -> List[PresetCatalogEntry]:
        """Get ordered list of active catalogs (sorted by priority ascending)."""

    def get_catalog_url(self) -> str:
        """Get primary catalog URL (backward compatibility helper)."""

    def is_cache_valid(self) -> bool:
        """True if cached catalog data is within CACHE_DURATION."""

    def fetch_catalog(self, force_refresh: bool = False) -> Dict[str, Any]:
        """
        Fetch and merge presets from all active catalogs.

        Args:
            force_refresh: If True, bypass cache and fetch fresh data.

        Returns:
            Dict[str, Any]: Merged catalog data dict.
        """

    def search(
        self,
        query: str = None,
        tag: str = None,
        author: str = None
    ) -> List[Dict[str, Any]]:
        """
        Search presets in the catalog.

        Args:
            query: Text to match against name, description, or ID.
            tag: Filter by tag (exact match).
            author: Filter by author name.

        Returns:
            List[Dict[str, Any]]: Matching preset info dicts.
        """

    def get_pack_info(self, pack_id: str) -> Optional[Dict[str, Any]]:
        """
        Get complete preset info from catalog.

        Args:
            pack_id: Preset ID to look up.

        Returns:
            Dict[str, Any] or None if not found.
        """

    def download_pack(
        self,
        pack_id: str,
        target_dir: Optional[Path] = None
    ) -> Path:
        """
        Download preset ZIP from catalog.

        Args:
            pack_id: Preset ID to download.
            target_dir: Directory to save ZIP file (defaults to cache directory).

        Returns:
            Path: Path to the downloaded ZIP file.

        Raises:
            PresetError: If pack not found or download fails.
        """

    def clear_cache(self) -> None:
        """Clear all cached catalog files."""
```

### PresetResolver

Resolves template names to file paths using a priority-based stack. This is the primary API for consuming installed presets in spec-kit workflows.

```python { .api }
class PresetResolver:
    def __init__(self, project_root: Path) -> None:
        """
        Args:
            project_root: Root directory of the spec-kit project.
        """

    # Resolution order (highest to lowest priority):
    # 1. .specify/templates/overrides/   - Project-local overrides (always wins)
    # 2. .specify/presets/<preset-id>/   - Installed presets (sorted by priority)
    # 3. .specify/extensions/<ext-id>/templates/ - Extension-provided templates
    # 4. .specify/templates/             - Core spec-kit templates (lowest priority)

    def resolve(
        self,
        template_name: str,
        template_type: str = "template"
    ) -> Optional[Path]:
        """
        Resolve a template name to its file path.

        Args:
            template_name: Template name without extension (e.g. "spec-template").
            template_type: One of "template", "command", or "script".

        Returns:
            Path to the resolved template file, or None if not found.
        """

    def resolve_with_source(
        self,
        template_name: str,
        template_type: str = "template"
    ) -> Optional[Dict[str, str]]:
        """
        Resolve template name to file path with source attribution.

        Args:
            template_name: Template name without extension.
            template_type: One of "template", "command", or "script".

        Returns:
            Dict with keys:
                "path": str  - Absolute path to the template file
                "source": str - Source description (e.g. "preset: my-preset",
                                "extension: my-ext", "override", "core")
            Returns None if template not found.
        """
```

## Usage Examples

### Install preset from catalog

```python
from pathlib import Path
from specify_cli.presets import PresetManager, PresetCatalog
from specify_cli import get_speckit_version

project_root = Path(".")
catalog = PresetCatalog(project_root)
manager = PresetManager(project_root)

# Search for presets
results = catalog.search(query="python")
for preset in results:
    print(preset["id"], preset.get("description", ""))

# Install a preset
if results:
    zip_path = catalog.download_pack(results[0]["id"])
    manifest = manager.install_from_zip(zip_path, get_speckit_version(), priority=5)
    print(f"Installed: {manifest.id} v{manifest.version}")
```

### Resolve templates

```python
from pathlib import Path
from specify_cli.presets import PresetResolver

resolver = PresetResolver(Path("."))

# Resolve a template file
template_path = resolver.resolve("spec-template", template_type="template")
if template_path:
    content = template_path.read_text()

# Resolve with source info (useful for debugging)
result = resolver.resolve_with_source("spec-template")
if result:
    print(f"Template found at: {result['path']}")
    print(f"Provided by: {result['source']}")
```

### List and manage installed presets

```python
from pathlib import Path
from specify_cli.presets import PresetManager

manager = PresetManager(Path("."))

# List installed
for preset in manager.list_installed():
    status = "disabled" if preset.get("disabled") else "enabled"
    print(f"{preset['id']} v{preset['version']} (priority: {preset.get('priority', 10)}, {status})")

# Remove a preset
removed = manager.remove("my-preset-id")
print("Removed:", removed)
```

### Use PresetRegistry directly

```python
from pathlib import Path
from specify_cli.presets import PresetRegistry

registry = PresetRegistry(Path(".specify/presets"))

# Check if installed
if registry.is_installed("my-preset"):
    meta = registry.get("my-preset")
    print(meta)

# Get by priority order
for pack_id, metadata in registry.list_by_priority():
    print(pack_id, metadata.get("priority", 10))
```
