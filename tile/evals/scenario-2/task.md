# Tool Check with specify-cli

Write a Python script that invokes the `specify` CLI's tool-verification subcommand and validates its behavior under different conditions.

## Capabilities

### Tool Check Command

- Runs the check subcommand and confirms it exits successfully (exit code 0) when all required tools are present on the system [@test](./test_check.py)
- Runs the check subcommand against an environment where a required tool is absent and confirms the CLI reports an error and exits with a non-zero code [@test](./test_check.py)
- Verifies that the check subcommand's output names the tools it inspects, allowing a caller to parse which tools passed or failed [@test](./test_check.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def run_check() -> subprocess.CompletedProcess:
    """
    Invoke the specify CLI's check subcommand.

    Returns:
        The completed subprocess result (returncode, stdout, stderr).
    """
    ...

def parse_check_output(output: str) -> dict[str, bool]:
    """
    Parse the stdout of the check subcommand into a mapping of
    tool name -> passed (True) or failed (False).
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

Provides the `specify` CLI. Use its `check` subcommand to verify that required tools are installed and to detect missing tools.

[@satisfied-by](specify-cli)
