# Multi-Step Progress Display with specify-cli

Write a Python script that uses the progress-tracking utility provided by the `specify-cli` package to manage and render the status of a multi-step workflow.

## Capabilities

### StepTracker Progress Rendering

- Creates a tracker, adds multiple named steps, marks each step through the pending → in-progress → completed lifecycle, and verifies the rendered output reflects each state [@test](./test_tracker.py)
- Marks a step as errored with a message and confirms the rendered output includes the error indicator and message [@test](./test_tracker.py)
- Marks a step as skipped and confirms the rendered output reflects the skipped state [@test](./test_tracker.py)
- Renders the full tracker state after mixed outcomes (some completed, one errored, one skipped) and asserts the output contains the correct status symbols or labels for each step [@test](./test_tracker.py)

## Implementation

[@generates](./solution.py)

## API

```python { #api }
def build_tracker(step_names: list[str]):
    """
    Instantiate the specify-cli progress tracker and register the given steps.

    Returns the tracker object ready for use.
    """
    ...

def run_lifecycle(tracker, step_name: str) -> str:
    """
    Drive step_name through its full lifecycle (pending → in-progress →
    completed) on the given tracker, then return the rendered output string.
    """
    ...

def run_error(tracker, step_name: str, message: str) -> str:
    """
    Mark step_name as errored with the provided message, then return the
    rendered output string.
    """
    ...

def run_skip(tracker, step_name: str) -> str:
    """
    Mark step_name as skipped, then return the rendered output string.
    """
    ...
```

## Dependencies { .dependencies }

### specify-cli 0.3.2 { .dependency }

Provides the progress-tracking class used to manage and render multi-step task state. Import it directly from the `specify` package.

[@satisfied-by](specify-cli)
