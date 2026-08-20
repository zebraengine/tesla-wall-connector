# Python Tesla Wall Connector API <!-- omit in TOC -->

Python Tesla Wall Connector API for local consumption. This package allows you to monitor your 3rd generation Tesla Wall Connector programmatically. It is mainly created to enable integration with Home Assistant and therefore exposes an asynchronous API.

> **Looking for wallmonitor?** The self-hosted recording & review UI, thermal
> derate forecaster, and automatic derate prevention that grew up in this
> fork's `monitor/` directory now has its own repository (full git history
> and open issues included): **<https://github.com/zebraengine/wallmonitor>**.
> This fork now tracks the upstream library only.

## Library usage

```python
import asyncio
from tesla_wall_connector import WallConnector
async def main():
    async with WallConnector('TeslaWallConnector_ABC123.localdomain') as wall_connector:
        lifetime = await wall_connector.async_get_lifetime()
        print("energy_wh: {}Wh".format(lifetime.energy_wh))

asyncio.run(main())
```

The library exposes four read-only endpoints — `async_get_vitals()`,
`async_get_lifetime()`, `async_get_version()`, and `async_get_wifi_status()` —
and includes workarounds for the charger's JSON quirks (literal `nan` values,
occasionally truncated responses). See `examples/` for a full walkthrough.

## Setting up development environment

This Python project is managed using [uv][uv], with project metadata defined in `pyproject.toml`.

You need at least:

- Python 3.11+
- [uv][uv-install]

To install all packages, including all development requirements:

```bash
uv sync
```

Then install the Git hook once for this clone:

```bash
uv run pre-commit install --install-hooks
```

As this repository uses the [pre-commit][pre-commit] framework, all changes
are linted and tested with each commit. You can run all checks and tests
manually, using the following command:

```bash
uv run pre-commit run --all-files
```

To run Ruff directly:

```bash
uv run ruff check .
uv run ruff format --check .
```

To run Pyright directly:

```bash
uv run pyright
```

To run the Python tests:

```bash
uv run pytest
```

The monitor app has its own project and tests:

```bash
cd monitor
uv sync
uv run pytest
```

[uv]: https://docs.astral.sh/uv/
[uv-install]: https://docs.astral.sh/uv/getting-started/installation/
[pre-commit]: https://pre-commit.com/
