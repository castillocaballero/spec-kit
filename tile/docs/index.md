# specify-cli

A Python CLI tool and library for bootstrapping Spec-Driven Development (SDD) projects. It provides project initialization with AI agent integration, extension management, preset-based template resolution, and command registration for 25+ AI coding agents (Claude Code, GitHub Copilot, Gemini CLI, Cursor, Windsurf, Kiro, and many more).

## Package Information

- **Package Name**: specify-cli
- **Package Type**: pypi
- **Language**: Python (>=3.11)
- **Installation**: `pip install specify-cli`
- **CLI Command**: `specify`
- **Entry Point**: `specify_cli.main`

## Core Imports

```python
# Main module - CLI app, utilities, constants
from specify_cli import (
    app, main,
    AGENT_CONFIG, AI_ASSISTANT_ALIASES, SCRIPT_TYPE_CHOICES,
    AGENT_SKILLS_DIR_OVERRIDES, SKILL_DESCRIPTIONS,
    StepTracker, BannerGroup,
    get_speckit_version, run_command, check_tool,
    is_git_repo, init_git_repo,
    merge_json_files, save_init_options, load_init_options,
    install_ai_skills, get_key, select_with_arrows,
    download_template_from_github, download_and_extract_template,
)

# Agent command registration
from specify_cli.agents import CommandRegistrar

# Extension management
from specify_cli.extensions import (
    ExtensionError, ValidationError, CompatibilityError,
    CatalogEntry, ExtensionManifest, ExtensionRegistry,
    ExtensionManager, ExtensionCatalog, ConfigManager, HookExecutor,
    normalize_priority, version_satisfies,
)

# Preset management
from specify_cli.presets import (
    PresetError, PresetValidationError, PresetCompatibilityError,
    PresetCatalogEntry, PresetManifest, PresetRegistry,
    PresetManager, PresetCatalog, PresetResolver,
    VALID_PRESET_TEMPLATE_TYPES,
)
```

## Basic Usage

```python
# Run as CLI
from specify_cli import main
main()  # invokes the `specify` Typer app

# Programmatically check and install an extension
from pathlib import Path
from specify_cli.extensions import ExtensionManager, ExtensionCatalog
from specify_cli import get_speckit_version

project_root = Path(".")
catalog = ExtensionCatalog(project_root)
manager = ExtensionManager(project_root)

# Search and install an extension
results = catalog.search(query="my-extension")
if results:
    zip_path = catalog.download_extension(results[0]["id"])
    manifest = manager.install_from_zip(zip_path, get_speckit_version())

# Resolve a template
from specify_cli.presets import PresetResolver
resolver = PresetResolver(project_root)
template_path = resolver.resolve("spec-template", template_type="template")
```

## Architecture

`specify-cli` is organized into four modules:

- **`specify_cli`** (`__init__.py`): Core CLI application (Typer), project initialization, utility functions, constants
- **`specify_cli.agents`**: `CommandRegistrar` for writing commands to agent-specific directories
- **`specify_cli.extensions`**: Extension lifecycle management (install/remove/search/hooks/config)
- **`specify_cli.presets`**: Preset lifecycle management (install/remove/search/template resolution)

The package uses a `.specify/` directory in the project root to store state:

```
.specify/
├── init-options.json     # Saved init options
├── extensions/           # Installed extensions + registry
├── presets/              # Installed presets + registry
├── templates/            # Core + override templates
│   └── overrides/        # Project-local template overrides
└── scripts/              # Helper scripts
```

## Capabilities

### Project Initialization & CLI

Initialize SDD projects, check tools, manage versions, and access UI utilities.

```python { .api }
def main() -> None: ...
app: typer.Typer  # Root CLI app (BannerGroup)

def get_speckit_version() -> str: ...
def show_banner() -> None: ...
def get_key() -> str: ...
def select_with_arrows(options: dict, prompt_text: str = "Select an option",
                       default_key: str = None) -> str: ...
```

[Project Initialization & CLI](./init-cli.md)

### Agent Command Registration

Register command files to AI agent-specific directories in Markdown or TOML format. Supports 22 agents.

```python { .api }
class CommandRegistrar:
    AGENT_CONFIGS: dict  # 22 agent configurations

    def register_commands(self, agent_name: str, commands: List[Dict[str, Any]],
                          source_id: str, source_dir: Path,
                          project_root: Path, context_note: str = None) -> List[str]: ...

    def register_commands_for_all_agents(self, commands: List[Dict[str, Any]],
                                          source_id: str, source_dir: Path,
                                          project_root: Path,
                                          context_note: str = None) -> Dict[str, List[str]]: ...

    def unregister_commands(self, registered_commands: Dict[str, List[str]],
                             project_root: Path) -> None: ...
```

[Agent Command Registration](./agents.md)

### Extension Management

Install, remove, search, and manage extensions with catalog fetching, hook execution, and layered configuration.

```python { .api }
class ExtensionManager:
    def install_from_zip(self, zip_path: Path, speckit_version: str,
                          priority: int = 10) -> ExtensionManifest: ...
    def remove(self, extension_id: str, keep_config: bool = False) -> bool: ...
    def list_installed(self) -> List[Dict[str, Any]]: ...

class ExtensionCatalog:
    def search(self, query: str = None, tag: str = None,
               author: str = None, verified_only: bool = False) -> List[Dict[str, Any]]: ...
    def download_extension(self, extension_id: str,
                            target_dir: Optional[Path] = None) -> Path: ...
```

[Extension Management](./extensions.md)

### Preset Management & Template Resolution

Install, remove, and search presets; resolve template names to file paths using a priority-based stack.

```python { .api }
class PresetManager:
    def install_from_zip(self, zip_path: Path, speckit_version: str,
                          priority: int = 10) -> PresetManifest: ...
    def remove(self, pack_id: str) -> bool: ...

class PresetResolver:
    def resolve(self, template_name: str,
                template_type: str = "template") -> Optional[Path]: ...
    def resolve_with_source(self, template_name: str,
                             template_type: str = "template") -> Optional[Dict[str, str]]: ...
```

[Preset Management & Template Resolution](./presets.md)
