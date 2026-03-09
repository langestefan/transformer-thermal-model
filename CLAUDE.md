# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
poetry install --with dev

# Run tests
poetry run pytest
poetry run pytest tests/path/to/test_file.py::test_name  # single test
poetry run pytest -m "not integrationtest"               # skip integration tests

# Lint and format
poetry run ruff check --fix .
poetry run ruff format .

# Type checking
poetry run mypy .

# All pre-commit hooks
poetry run pre-commit run -a

# Mutation testing
poetry run mutmut run

# Docs
mkdocs serve
```

## Architecture

The library implements the IEC 60076-7 thermal model for transformers. The core flow is: construct a `Transformer` → provide an `InputProfile` → run `Model.calculate()` → get an `OutputProfile`.

### Key modules

**`transformer_thermal_model/model/`** — The main calculation engine. `Model` takes a `Transformer` and `InputProfile`, runs vectorized NumPy calculations, and returns an `OutputProfile` with top-oil and hot-spot temperatures over time.

**`transformer_thermal_model/transformer/`** — Transformer hierarchy:
- `base.py`: Abstract `Transformer` base class
- `power.py`: `PowerTransformer` (two-winding)
- `distribution.py`: `DistributionTransformer`
- `threewinding.py`: `ThreeWindingTransformer`
- `cooling_switch_controller.py`: ONAN/ONAF cooling mode switching logic

**`transformer_thermal_model/schemas/`** — Pydantic v2 models for all inputs/outputs:
- `specifications/`: `UserTransformerSpecifications`, `DefaultTransformerSpecifications`
- `thermal_model/`: `InputProfile`, `ThreeWindingInputProfile`, `OutputProfile`, initial state types (`ColdStart`, `InitialLoad`, `InitialTopOilTemp`)

**`transformer_thermal_model/aging/`** — Insulation aging calculations based on hot-spot temperature.

**`transformer_thermal_model/hot_spot_calibration/`** — Calibration logic for the hot-spot factor.

**`transformer_thermal_model/toolbox/`** — Utility functions for pandas DataFrame integration.

### Design conventions

- Pydantic v2 schemas for all data validation at boundaries
- Strict mypy (all functions must have type annotations)
- NumPy vectorization for thermal calculations (no Python loops over time steps)
- Google-style docstrings
- Line length: 120 characters
- SPDX license headers required on all source files (REUSE compliant)

## Commit conventions

Commits follow Angular/Conventional Commits format and require DCO sign-off:

```bash
git commit -s -m "feat: add support for XYZ"
```

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `ci`

Merge strategy: squash and merge to `main`.
