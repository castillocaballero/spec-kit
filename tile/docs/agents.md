# Agent Command Registration

The `specify_cli.agents` module provides `CommandRegistrar` for writing command files to AI agent-specific directories in the correct format (Markdown or TOML).

## Module

```python
from specify_cli.agents import CommandRegistrar
```

## Capabilities

### CommandRegistrar

Handles registration of commands with AI agents. Supports writing command files in Markdown or TOML format to the appropriate agent directory, with correct argument placeholders and companion files (e.g. Copilot `.prompt.md`).

```python { .api }
class CommandRegistrar:
    AGENT_CONFIGS: dict
    # Keys: claude, gemini, copilot, cursor, qwen, opencode, codex, windsurf,
    #       kilocode, auggie, roo, codebuddy, qodercli, kiro-cli, pi, amp,
    #       shai, tabnine, bob, kimi, trae, iflow
    # Each entry:
    # {
    #   "dir": str,         # Command dir relative to project root (e.g. ".claude/commands")
    #   "format": str,      # "markdown" or "toml"
    #   "args": str,        # Argument placeholder (e.g. "$ARGUMENTS" or "{{args}}")
    #   "extension": str    # File extension (e.g. ".md", ".toml", "/SKILL.md")
    # }

    @staticmethod
    def parse_frontmatter(content: str) -> tuple[dict, str]:
        """
        Parse YAML frontmatter from Markdown content.

        Args:
            content: Markdown string, optionally starting with --- frontmatter block.

        Returns:
            tuple[dict, str]: (frontmatter_dict, body_content).
                frontmatter_dict is {} if no valid frontmatter found.
        """

    @staticmethod
    def render_frontmatter(fm: dict) -> str:
        """
        Render frontmatter dict as YAML with --- delimiters.

        Args:
            fm: Frontmatter dictionary.

        Returns:
            str: "---\\n<yaml>\\n---\\n" or "" if fm is empty.
        """

    def render_markdown_command(
        self,
        frontmatter: dict,
        body: str,
        source_id: str,
        context_note: str = None
    ) -> str:
        """
        Render command in Markdown format.

        Args:
            frontmatter: Command frontmatter dict.
            body: Command body content (Markdown).
            source_id: Source identifier (extension or preset ID).
            context_note: Custom HTML comment for source attribution.
                Defaults to "\\n<!-- Source: {source_id} -->\\n".

        Returns:
            str: Full Markdown command file content.
        """

    def render_toml_command(
        self,
        frontmatter: dict,
        body: str,
        source_id: str
    ) -> str:
        """
        Render command in TOML format (for Gemini, Tabnine).

        Args:
            frontmatter: Command frontmatter dict (uses "description" key if present).
            body: Command body content.
            source_id: Source identifier.

        Returns:
            str: TOML file with description, source comment, and prompt = \"\"\"...\"\"\".
        """

    def register_commands(
        self,
        agent_name: str,
        commands: List[Dict[str, Any]],
        source_id: str,
        source_dir: Path,
        project_root: Path,
        context_note: str = None
    ) -> List[str]:
        """
        Register commands for a specific agent.

        Creates the agent command directory if it doesn't exist.
        Reads command files from source_dir, converts argument placeholders,
        and writes formatted output to the agent's command directory.
        For Copilot, also writes companion .prompt.md files.

        Args:
            agent_name: Agent key (must be in AGENT_CONFIGS).
            commands: List of command info dicts:
                [{"name": str, "file": str, "aliases": List[str] (optional)}, ...]
            source_id: Identifier of the source (extension or preset ID).
            source_dir: Directory containing the command source .md files.
            project_root: Path to project root.
            context_note: Custom context comment for markdown output.

        Returns:
            List[str]: Registered command names (includes aliases).

        Raises:
            ValueError: If agent_name is not in AGENT_CONFIGS.
        """

    def register_commands_for_all_agents(
        self,
        commands: List[Dict[str, Any]],
        source_id: str,
        source_dir: Path,
        project_root: Path,
        context_note: str = None
    ) -> Dict[str, List[str]]:
        """
        Register commands for all detected agents in the project.

        Only registers for agents whose top-level directory exists
        in project_root (e.g. .claude/, .github/, etc.).

        Args:
            commands: List of command info dicts.
            source_id: Source identifier.
            source_dir: Directory containing command source files.
            project_root: Path to project root.
            context_note: Custom context comment for markdown output.

        Returns:
            Dict[str, List[str]]: {agent_name: [registered_cmd_names]}.
        """

    def unregister_commands(
        self,
        registered_commands: Dict[str, List[str]],
        project_root: Path
    ) -> None:
        """
        Remove previously registered command files from agent directories.

        Args:
            registered_commands: Dict mapping agent names to command name lists.
                Typically the return value of register_commands_for_all_agents.
            project_root: Path to project root.
        """

    @staticmethod
    def write_copilot_prompt(project_root: Path, cmd_name: str) -> None:
        """
        Generate companion .prompt.md file for a Copilot agent command.

        Creates .github/prompts/<cmd_name>.prompt.md with minimal frontmatter.

        Args:
            project_root: Path to project root.
            cmd_name: Command name (e.g. 'speckit.my-ext.example').
        """
```

### Usage Examples

#### Register commands for a specific agent

```python
from pathlib import Path
from specify_cli.agents import CommandRegistrar

registrar = CommandRegistrar()
project_root = Path(".")
source_dir = Path(".specify/extensions/my-ext/commands")

commands = [
    {"name": "speckit.my-ext.analyze", "file": "analyze.md"},
    {"name": "speckit.my-ext.generate", "file": "generate.md", "aliases": ["speckit.my-ext.gen"]},
]

# Register for a single agent
registered = registrar.register_commands(
    agent_name="claude",
    commands=commands,
    source_id="my-ext@1.0.0",
    source_dir=source_dir,
    project_root=project_root,
)
# registered == ["speckit.my-ext.analyze", "speckit.my-ext.generate", "speckit.my-ext.gen"]
```

#### Register commands for all detected agents

```python
from pathlib import Path
from specify_cli.agents import CommandRegistrar

registrar = CommandRegistrar()
project_root = Path(".")

results = registrar.register_commands_for_all_agents(
    commands=[{"name": "speckit.my-ext.cmd", "file": "cmd.md"}],
    source_id="my-ext@1.0.0",
    source_dir=Path(".specify/extensions/my-ext/commands"),
    project_root=project_root,
)
# results == {"claude": ["speckit.my-ext.cmd"], "copilot": ["speckit.my-ext.cmd"]}
```

#### Parse and render frontmatter

```python
from specify_cli.agents import CommandRegistrar

content = """---
description: My command description
---
This is the command body with $ARGUMENTS placeholder.
"""
frontmatter, body = CommandRegistrar.parse_frontmatter(content)
# frontmatter == {"description": "My command description"}
# body == "This is the command body with $ARGUMENTS placeholder."

yaml_fm = CommandRegistrar.render_frontmatter(frontmatter)
# "---\ndescription: My command description\n---\n"
```

#### Unregister commands

```python
from pathlib import Path
from specify_cli.agents import CommandRegistrar

registrar = CommandRegistrar()

# previously_registered was returned by register_commands_for_all_agents
previously_registered = {"claude": ["speckit.my-ext.cmd"], "copilot": ["speckit.my-ext.cmd"]}
registrar.unregister_commands(previously_registered, project_root=Path("."))
```

## Agent Configuration Details

Each entry in `AGENT_CONFIGS` has:

| Field | Description |
|-------|-------------|
| `dir` | Command directory relative to project root |
| `format` | `"markdown"` or `"toml"` |
| `args` | Argument placeholder: `"$ARGUMENTS"` (markdown) or `"{{args}}"` (toml) |
| `extension` | File extension for command files |

Selected agent details:

| Agent Key | Directory | Format | Extension |
|-----------|-----------|--------|-----------|
| `claude` | `.claude/commands` | markdown | `.md` |
| `gemini` | `.gemini/commands` | toml | `.toml` |
| `copilot` | `.github/agents` | markdown | `.agent.md` |
| `cursor` | `.cursor/commands` | markdown | `.md` |
| `windsurf` | `.windsurf/workflows` | markdown | `.md` |
| `kiro-cli` | `.kiro/prompts` | markdown | `.md` |
| `codex` | `.codex/prompts` | markdown | `.md` |
| `tabnine` | `.tabnine/agent/commands` | toml | `.toml` |
| `kimi` | `.kimi/skills` | markdown | `/SKILL.md` |
| `opencode` | `.opencode/command` | markdown | `.md` |
| `amp` | `.agents/commands` | markdown | `.md` |
