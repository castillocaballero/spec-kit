# AI Agent Selection with specify-cli

Write a Python script that drives the `specify` CLI to initialize projects with different AI agent configurations and verifies that each agent type is correctly recorded in the resulting project scaffold.

## Capabilities

### AI Agent Selection and Configuration

- Initializes a project selecting the `claude` agent and confirms the agent is recorded in the project configuration [@test](./test_agent.py)
- Initializes a project selecting the `copilot` agent and confirms the agent is recorded in the project configuration [@test](./test_agent.py)
- Initializes a project selecting the `gemini` agent and confirms the agent is recorded in the project configuration [@test](./test_agent.py)
- Initializes a project with the `generic` agent, supplying a custom commands directory path, and verifies that the path is stored in the project configuration [@test](./test_agent.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def run_init_with_agent(
    target: str,
    agent: str,
    commands_dir: str | None = None,
) -> subprocess.CompletedProcess:
    """
    Invoke the specify CLI to initialize a project with the given agent.

    Args:
        target:       Directory to initialize the project in.
        agent:        Agent identifier string (e.g. 'claude', 'copilot',
                      'gemini', 'generic').
        commands_dir: For the generic agent, the path to the custom commands
                      directory. Passed via the appropriate CLI option.

    Returns:
        The completed subprocess result.
    """
    ...

def read_project_agent(project_dir: str) -> dict:
    """
    Parse the specify project configuration in project_dir and return the
    agent-related settings as a dictionary.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

Provides the `specify` CLI. Use it as a subprocess to test agent selection via its `--agent` option (and `--commands-dir` for the generic agent) during project initialization.

[@satisfied-by](specify-cli)
