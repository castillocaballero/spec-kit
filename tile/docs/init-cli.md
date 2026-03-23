# Project Initialization & CLI

Core CLI application, project initialization utilities, constants, and UI helpers.

## Package Information

- **Module**: `specify_cli`
- **Installation**: `pip install specify-cli`
- **CLI entry point**: `specify` command

## Capabilities

### CLI Application

The root Typer application and main entry point.

```python { .api }
app: typer.Typer
# Root CLI application with custom BannerGroup (shows banner on help)

def main() -> None:
    """Entry point registered as `specify` script. Calls app()."""
```

Usage:

```python
from specify_cli import main
main()  # runs the specify CLI
```

Or as CLI:

```bash
specify --help
specify init myproject --ai claude
specify check
specify version
```

### Constants: AGENT_CONFIG

Configuration for all supported AI agents. Maps agent key to dict with `name`, `folder`, `commands_subdir`, `install_url`, `requires_cli`.

```python { .api }
AGENT_CONFIG: dict
# Keys: copilot, claude, gemini, cursor-agent, qwen, opencode, codex,
#       windsurf, kilocode, auggie, codebuddy, qodercli, roo, kiro-cli,
#       amp, shai, tabnine, agy, bob, vibe, kimi, trae, pi, iflow, generic

# Each entry:
# {
#   "name": str,          # Display name (e.g. "Claude Code")
#   "folder": str,        # Agent folder in project root (e.g. ".claude/")
#   "commands_subdir": str, # Subdir for commands (e.g. "commands")
#   "install_url": str | None,  # Install URL or None for IDE-based
#   "requires_cli": bool  # True if CLI tool check is needed
# }

AI_ASSISTANT_ALIASES: dict
# {"kiro": "kiro-cli"}

AI_ASSISTANT_HELP: str
# Auto-generated help text for --ai option listing all agents

SCRIPT_TYPE_CHOICES: dict
# {"sh": "POSIX Shell (bash/zsh)", "ps": "PowerShell"}

INIT_OPTIONS_FILE: str  # ".specify/init-options.json"
DEFAULT_SKILLS_DIR: str  # ".agents/skills"
CLAUDE_LOCAL_PATH: Path  # ~/.claude/local/claude
BANNER: str              # ASCII art banner
TAGLINE: str             # "GitHub Spec Kit - Spec-Driven Development Toolkit"

AGENT_SKILLS_DIR_OVERRIDES: dict
# Agent-specific skills directory overrides for agents whose skills directory
# doesn't follow the standard <agent_folder>/skills/ pattern.
# e.g. {"codex": ".agents/skills"}

SKILL_DESCRIPTIONS: dict
# Maps spec-kit command short names to enhanced descriptions for agent skills.
# Keys: "specify", "plan", "tasks", "implement", "analyze",
#       "clarify", "constitution", "checklist", "taskstoissues"
```

### Version Management

```python { .api }
def get_speckit_version() -> str:
    """
    Get current spec-kit version from package metadata or pyproject.toml.

    Returns:
        str: Version string (e.g. "0.3.2") or "unknown" if unavailable.
    """
```

### StepTracker

Track and render hierarchical progress steps for display with Rich.

```python { .api }
class StepTracker:
    def __init__(self, title: str) -> None:
        """
        Args:
            title: Title displayed at the top of the step tree
        """

    def attach_refresh(self, cb: Callable) -> None:
        """Attach a callable that triggers UI refresh when steps update."""

    def add(self, key: str, label: str) -> None:
        """Add a new pending step. No-op if key already exists."""

    def start(self, key: str, detail: str = "") -> None:
        """Mark step as running (in-progress)."""

    def complete(self, key: str, detail: str = "") -> None:
        """Mark step as done (success)."""

    def error(self, key: str, detail: str = "") -> None:
        """Mark step as error (failed)."""

    def skip(self, key: str, detail: str = "") -> None:
        """Mark step as skipped."""

    def render(self) -> Tree:
        """Render all steps as a Rich Tree object for display."""
```

Usage:

```python
from specify_cli import StepTracker
from rich.console import Console
from rich.live import Live

tracker = StepTracker("Installing extension")
tracker.add("download", "Download ZIP")
tracker.add("validate", "Validate manifest")
tracker.add("install", "Install files")

console = Console()
with Live(tracker.render(), console=console) as live:
    tracker.attach_refresh(live.refresh)
    tracker.start("download")
    # ... do work ...
    tracker.complete("download", "Downloaded 1.2 MB")
    tracker.start("validate")
    # ... validate ...
    tracker.complete("validate")
    tracker.start("install")
    # ... install ...
    tracker.complete("install", "Installed to .specify/extensions/my-ext/")
```

### BannerGroup

Custom Typer group that shows the ASCII art banner before help text.

```python { .api }
class BannerGroup(typer.core.TyperGroup):
    def format_help(self, ctx: Any, formatter: Any) -> None: ...
```

### UI / Input Utilities

```python { .api }
def show_banner() -> None:
    """Display ASCII art banner with tagline using Rich console."""

def get_key() -> str:
    """
    Get single keypress cross-platform using readchar.

    Returns:
        str: 'up', 'down', 'enter', 'escape', or a character string.
    """

def select_with_arrows(
    options: dict,
    prompt_text: str = "Select an option",
    default_key: str = None
) -> str:
    """
    Interactive selection menu using arrow keys and Rich Live display.

    Args:
        options: Dict mapping option key to display description string.
        prompt_text: Text shown above the options list.
        default_key: Key of default pre-selected option.

    Returns:
        str: The selected option key.

    Raises:
        typer.Exit(1): On error or if user cancels (Ctrl+C / Escape).
    """
```

Usage:

```python
from specify_cli import select_with_arrows

choice = select_with_arrows(
    options={"claude": "Claude Code", "copilot": "GitHub Copilot", "gemini": "Gemini CLI"},
    prompt_text="Select AI assistant",
    default_key="claude",
)
print(f"Selected: {choice}")
```

### Shell & System Utilities

```python { .api }
def run_command(
    cmd: list[str],
    check_return: bool = True,
    capture: bool = False,
    shell: bool = False
) -> Optional[str]:
    """
    Run a shell command.

    Args:
        cmd: Command as list of strings (e.g. ["git", "status"]).
        check_return: If True, raise on non-zero exit code.
        capture: If True, capture and return stdout.
        shell: If True, run through the system shell.

    Returns:
        Captured stdout string if capture=True, else None.

    Raises:
        subprocess.CalledProcessError: If check_return=True and exit code != 0.
    """

def check_tool(tool: str, tracker: StepTracker = None) -> bool:
    """
    Check if a CLI tool is installed and accessible.

    Args:
        tool: Tool name (e.g. "git", "claude", "gh").
        tracker: Optional StepTracker to update with result.

    Returns:
        bool: True if tool is found, False otherwise.

    Note:
        Claude CLI is searched at CLAUDE_LOCAL_PATH (~/.claude/local/claude)
        first (post `claude migrate-installer`), then via PATH.
    """

def is_git_repo(path: Path = None) -> bool:
    """
    Check if path is inside a git repository.

    Args:
        path: Directory to check. Defaults to current directory.

    Returns:
        bool: True if path is inside a git repository.
    """

def init_git_repo(project_path: Path, quiet: bool = False) -> Tuple[bool, Optional[str]]:
    """
    Initialize a git repository.

    Runs: git init, git add ., git commit -m "Initial commit from Specify template"
    Note: The directory must contain at least one file for git commit to succeed.

    Args:
        project_path: Directory to initialize.
        quiet: If True, suppress output.

    Returns:
        Tuple[bool, Optional[str]]: (success, error_message).
            error_message is None on success.
    """
```

### File / JSON Utilities

```python { .api }
def merge_json_files(
    existing_path: Path,
    new_content: Any,
    verbose: bool = False
) -> Optional[dict[str, Any]]:
    """
    Deep merge new JSON content into an existing JSON file.

    Merge semantics: existing keys are preserved unless both values are dicts
    (in which case they are recursively merged).

    Args:
        existing_path: Path to existing JSON file.
        new_content: New JSON-serializable content to merge in.
        verbose: If True, print merge details.

    Returns:
        Merged dict, or None if file should be left untouched.
    """

def handle_vscode_settings(
    sub_item,
    dest_file,
    rel_path,
    verbose: bool = False,
    tracker: StepTracker = None
) -> None:
    """
    Handle merging or copying of .vscode/settings.json files.
    Preserves existing settings; normalizes comments and trailing commas.
    """

def ensure_executable_scripts(
    project_path: Path,
    tracker: StepTracker | None = None
) -> None:
    """
    Make all .sh scripts under .specify/scripts executable.
    No-op on Windows.
    """

def ensure_constitution_from_template(
    project_path: Path,
    tracker: StepTracker | None = None
) -> None:
    """Copy constitution template to memory directory if not already present."""
```

### Init Options Persistence

```python { .api }
def save_init_options(project_path: Path, options: dict[str, Any]) -> None:
    """
    Persist CLI options from `specify init` to .specify/init-options.json.

    Args:
        project_path: Project root directory.
        options: Dict of option name -> value.
    """

def load_init_options(project_path: Path) -> dict[str, Any]:
    """
    Load previously saved init options.

    Args:
        project_path: Project root directory.

    Returns:
        dict: Options dict, or {} if file missing or unparseable.
    """
```

### GitHub Template Download

```python { .api }
def download_template_from_github(
    ai_assistant: str,
    download_dir: Path,
    *,
    script_type: str = "sh",
    verbose: bool = True,
    show_progress: bool = True,
    client: httpx.Client = None,
    debug: bool = False,
    github_token: str = None
) -> Tuple[Path, dict]:
    """
    Download the latest template release from GitHub for an AI assistant.

    Args:
        ai_assistant: Agent key from AGENT_CONFIG (e.g. "claude", "copilot").
        download_dir: Directory to download ZIP file to.
        script_type: "sh" (POSIX/bash/zsh) or "ps" (PowerShell).
        verbose: Print status messages.
        show_progress: Show download progress bar.
        client: Custom httpx.Client (uses module-level client if None).
        debug: Show detailed error info.
        github_token: GitHub API token for authenticated requests.

    Returns:
        Tuple[Path, dict]: (zip_path, metadata_dict).

    Raises:
        typer.Exit(1): On network error or HTTP failure.
    """

def download_and_extract_template(
    project_path: Path,
    ai_assistant: str,
    script_type: str,
    is_current_dir: bool = False,
    *,
    verbose: bool = True,
    tracker: StepTracker | None = None,
    client: httpx.Client = None,
    debug: bool = False,
    github_token: str = None
) -> Path:
    """
    Download latest GitHub template release and extract to project.

    Args:
        project_path: Target project directory.
        ai_assistant: Agent key (e.g. "claude").
        script_type: "sh" or "ps".
        is_current_dir: True if initializing in the current directory.
        verbose: Print status messages.
        tracker: StepTracker for progress updates.
        client: Custom httpx.Client.
        debug: Show verbose diagnostic output.
        github_token: GitHub API token.

    Returns:
        Path: project_path (unchanged).
    """
```

### Agent Skills Installation

```python { .api }
def install_ai_skills(
    project_path: Path,
    selected_ai: str,
    tracker: StepTracker | None = None
) -> bool:
    """
    Install Prompt.MD files from templates/commands/ as agent skills.

    Args:
        project_path: Target project directory.
        selected_ai: Agent key from AGENT_CONFIG.
        tracker: Optional StepTracker for progress.

    Returns:
        bool: True if skills installed or already present, False otherwise.
    """
```

### GitHub Rate Limit Utilities

```python { .api }
def _github_token(cli_token: str | None = None) -> str | None:
    """
    Return sanitized GitHub token.
    Precedence: cli_token > GH_TOKEN env var > GITHUB_TOKEN env var.
    Returns None if no token found.
    """

def _github_auth_headers(cli_token: str | None = None) -> dict:
    """
    Return Authorization header dict if token exists, else {}.
    Returns: {"Authorization": "Bearer <token>"} or {}
    """

def _parse_rate_limit_headers(headers: httpx.Headers) -> dict:
    """
    Parse GitHub rate-limit response headers.

    Returns dict with zero or more keys:
      limit: str            # X-RateLimit-Limit
      remaining: str        # X-RateLimit-Remaining
      reset_epoch: int      # X-RateLimit-Reset as Unix timestamp
      reset_time: datetime  # reset_epoch as UTC datetime
      reset_local: datetime # reset_epoch as local datetime
      retry_after_seconds: int  # Retry-After as seconds (if numeric)
      retry_after: str          # Retry-After as HTTP-date (if non-numeric)
    """

def _format_rate_limit_error(
    status_code: int,
    headers: httpx.Headers,
    url: str
) -> str:
    """Format a user-friendly error message including rate limit information."""
```

### CLI Commands Reference

All commands available via the `specify` CLI:

```bash
# Top-level
specify init [PROJECT_NAME]
    --ai TEXT                  AI assistant (claude, copilot, gemini, cursor-agent, ...)
    --ai-commands-dir PATH     Custom commands directory (required with --ai=generic)
    --script [sh|ps]           Script type (default: sh)
    --ignore-agent-tools       Skip tool availability checks
    --no-git                   Skip git initialization
    --here                     Initialize in current directory
    --force                    Force merge when using --here
    --skip-tls                 Skip SSL/TLS verification
    --debug                    Show verbose diagnostic output
    --github-token TEXT        GitHub API token (or set GH_TOKEN/GITHUB_TOKEN env)
    --ai-skills / --no-ai-skills  Install agent skills (default: no)
    --preset TEXT              Install preset during init

specify check                   Check for installed required tools
specify version                 Display version and system information

# Preset subcommands
specify preset list
specify preset add <PRESET_ID>     [--force] [--github-token TEXT]
specify preset remove <PRESET_ID>  [--keep-config]
specify preset search [QUERY]      [--tag TEXT] [--author TEXT]
specify preset resolve [TEMPLATE_NAME] [--type template|command|script]
specify preset info <PRESET_ID>
specify preset set-priority <PRESET_ID> <PRIORITY>
specify preset enable <PRESET_ID>
specify preset disable <PRESET_ID>

specify preset catalog list
specify preset catalog add <URL>    [--name TEXT] [--priority INT]
specify preset catalog remove <URL>

# Extension subcommands
specify extension list
specify extension add <EXTENSION_ID>    [--force] [--github-token TEXT]
specify extension remove <EXTENSION_ID> [--keep-config]
specify extension search [QUERY]        [--tag TEXT] [--author TEXT] [--verified]
specify extension info <EXTENSION_ID>
specify extension update [EXTENSION_ID]  # updates all if no ID given
specify extension enable <EXTENSION_ID>
specify extension disable <EXTENSION_ID>
specify extension set-priority <EXTENSION_ID> <PRIORITY>

specify extension catalog list
specify extension catalog add <URL>    [--name TEXT] [--priority INT]
specify extension catalog remove <URL>
```
