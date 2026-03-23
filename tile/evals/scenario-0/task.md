# Project Initialization with specify-cli

Write a Python script that drives the `specify` CLI to initialize spec-driven projects under various conditions and verifies the resulting directory layout.

## Capabilities

### Project Initialization

- Initializes a new project in a freshly created empty directory, producing the expected scaffolded files [@test](./test_init.py)
- Initializes a project in the current working directory using the `--here` flag, placing scaffold files directly in that directory [@test](./test_init.py)
- Refuses to initialize when the target directory already contains a conflicting project structure, reporting an appropriate error [@test](./test_init.py)
- Initializes in a new subdirectory specified by name when that subdirectory does not yet exist [@test](./test_init.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def run_init(target: str | None = None, here: bool = False) -> subprocess.CompletedProcess:
    """
    Invoke the specify CLI to initialize a project.

    Args:
        target: Optional directory name or path to initialize into.
        here:   When True, pass --here to initialize in the current directory.

    Returns:
        The completed subprocess result (returncode, stdout, stderr).
    """
    ...

def scaffold_files_present(directory: str) -> bool:
    """
    Return True if the given directory contains the expected specify project
    scaffold (e.g. a spec config file and a specs/ folder).
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

Provides the `specify` command-line tool. Use it as a subprocess to test project initialization behavior including the `--here` flag and conflict detection.

[@satisfied-by](specify-cli)
