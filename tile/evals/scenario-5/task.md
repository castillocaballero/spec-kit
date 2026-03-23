# Multi-agent Command Registration

specify-cli supports registering spec-kit commands for multiple AI agent environments. Each agent stores its commands in a distinct directory and format: Claude uses `.claude/commands/` (Markdown files), Gemini uses `.gemini/commands/` (TOML files), and GitHub Copilot uses `.github/agents/` (YAML files). Your task is to write tests that verify a command registered for one agent is written to the correct directory in the correct format, and that registering the same command for different agents produces separate, correctly-placed files.

## Capabilities

### Multi-agent Command Registration

- Registering a command for the `claude` agent creates a Markdown file under `.claude/commands/` [@test](./test_command_registration.py)
- Registering a command for the `gemini` agent creates a TOML file under `.gemini/commands/` [@test](./test_command_registration.py)
- Registering a command for the `copilot` agent creates a YAML file under `.github/agents/` [@test](./test_command_registration.py)
- Registering commands for all supported agents in one call produces all three files in their respective directories [@test](./test_command_registration.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def register_command_for_agent(
    command_name: str,
    command_content: str,
    agent: str,
    base_dir: str,
) -> str:
    """
    Register a spec-kit command for a specific AI agent.

    Args:
        command_name: Name of the command to register (without file extension).
        command_content: The body/content of the command.
        agent: Target agent identifier. One of: "claude", "gemini", "copilot".
        base_dir: Root directory of the project where agent command dirs are created.

    Returns:
        Absolute path to the file that was created.
    """
    ...

def register_command_for_all_agents(
    command_name: str,
    command_content: str,
    base_dir: str,
) -> list[str]:
    """
    Register a spec-kit command for all supported AI agents.

    Args:
        command_name: Name of the command to register.
        command_content: The body/content of the command.
        base_dir: Root directory of the project.

    Returns:
        List of absolute paths to all files created.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

specify-cli is a CLI toolkit for managing AI agent spec-kit commands and extensions. It provides programmatic APIs for registering commands into agent-specific directories with the appropriate file format for each supported agent environment.

[@satisfied-by](specify-cli)
