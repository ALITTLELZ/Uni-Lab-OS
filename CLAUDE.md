# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Please follow the rules defined in:

@AGENTS.md

## Additional Context

### Project Overview

Uni-Lab-OS (`unilabos`) is a laboratory automation platform that connects and controls experimental equipment via ROS2 or a simple Python backend. Entry point: `unilab` CLI → `unilabos/app/main.py:main()`.

### Environment Setup

```bash
# Requires mamba with python 3.11
mamba create -n unilab python=3.11.14
mamba activate unilab
mamba install uni-lab::unilabos-env -c robostack-staging -c conda-forge

# Developer install
pip install -e .
uv pip install -r unilabos/utils/requirements.txt
```

### Testing

Two test directories exist:
- `tests/` — main pytest suite (resources, devices, workflow, ros)
- `test/` — device integration tests (PRCXI devices) with custom conftest

```bash
pytest tests/                                    # all tests
pytest tests/resources/test_resourcetreeset.py   # single file
pytest tests/resources/test_resourcetreeset.py::TestClass::test_method  # single test
```

Custom pytest markers: `@pytest.mark.slow`, `@pytest.mark.integration`, `@pytest.mark.unit`, `@pytest.mark.prcxi_9300`, `@pytest.mark.prcxi_9320`.

Tests use `lab_registry.setup()` for registry initialization.

### CI

CI runs `python -m unilabos --check_mode --skip_env_check` on Windows (see `.github/workflows/ci-check.yml`). Use `--complete_registry` to regenerate YAML registry files from AST analysis — CI checks for uncommitted changes after this.

### Registry Decorators

Key decorators in `unilabos/registry/decorators.py`:
- `@device` / `@resource` — register device/resource classes
- `@action` — register action methods on devices
- `@topic` — configure ROS2 topic publishing
- `@is_not_action` — exclude methods from action discovery
- `@is_always_free` — mark methods as always available

Registry init flow: AST static scan of decorators → setup host_node → YAML loading → verify & import resolution.

### Additional CLI Flags (beyond AGENTS.md)

```bash
--registry_path PATH       # additional registry directories
--devices PATH             # Python dirs for AST device scanning
--working_dir PATH         # working directory
--complete_registry        # regenerate YAML registry from AST
--restart_mode             # supervisor mode with auto-restart (exit code 42 triggers restart)
--extra_resource           # load lab_ prefixed labware resources
--external_devices_only    # only load external device packages
```

### Logging

Custom `TRACE` level (level 5, below DEBUG) in `unilabos/utils/log.py`. Log levels: TRACE, DEBUG, INFO, WARNING, ERROR, CRITICAL. Set via `BasicConfig.log_level`.

### No Linting/Formatting Tools Configured

No ruff, black, flake8, mypy, or pre-commit configuration exists in this repo.
