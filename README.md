# Python Tesla Wall Connector API <!-- omit in TOC -->

Tools for monitoring a 3rd generation Tesla Wall Connector **entirely over your
local network** — nothing here talks to any cloud service or sends data
anywhere outside your LAN.

This repository contains two parts:

| Part | What it is |
|---|---|
| [`tesla_wall_connector`](#library-usage) | The Python API library: async client for the Wall Connector's local HTTP API (vitals, lifetime, version, Wi-Fi status). Originally created to enable the Home Assistant integration. |
| [`monitor/`](#wallmonitor--recording--review-ui) | **wallmonitor** — a self-hosted recording and review web UI built on the library: it captures everything the charger reports into SQLite and gives you live dashboards, charging-session review, Wi-Fi health, and an alert/event timeline. |

## wallmonitor — recording & review UI

A local-only web app (aiohttp + SQLite + a dependency-free frontend — no CDNs,
no analytics, binds to localhost by default) that:

1. **Records at the highest device-safe fidelity** — vitals every 2 s while a
   vehicle is attached (5 s idle), Wi-Fi every 30 s, lifetime counters every
   60 s, firmware every 6 h. Requests are strictly sequential with exponential
   backoff so the charger's small embedded web server is never overloaded, and
   every response is stored with its **complete raw JSON** — nothing is discarded.
2. **Charging session review** — plug-in → unplug sessions are auto-detected
   and aggregated (energy, peak/average power, charging time), listed with
   date filters, and drillable into power / per-phase current / voltage /
   temperature charts plus the events that happened during each session.
3. **Wi-Fi health monitoring** — RSSI, SNR, and signal-strength history,
   connect/disconnect and internet-reachability events.
4. **Live session review** — a live dashboard (Server-Sent Events) with
   rolling charts, stat tiles, EVSE state, and the in-progress session.
5. **Alert & error monitoring** — device alerts are diffed into raise/clear
   records with timestamps; charger reboots, unreachability, and Wi-Fi drops
   are first-class events, with an active-alert banner on every page.
6. **One synchronized clock** — every sample, session boundary, alert, and
   event is stamped with host UTC at the moment it was observed and rendered
   in your local timezone, so you can reconstruct the exact operating
   conditions around any error.

It's also built to run unattended on an always-on box: restarts resume an
open charging session seamlessly, downtime is recorded as an explicit
"monitoring gap" event (even after hard reboots), and known firmware quirks —
invalid-sensor 255 °C temperature sentinels, malformed JSON — are handled so
they never masquerade as real data.

### Quick start

```bash
cd monitor
uv sync

# against your Wall Connector:
uv run python -m wallmonitor --host 192.168.1.50

# North American split-phase install:
uv run python -m wallmonitor --host 192.168.1.50 --split-phase

# no hardware handy? demo mode runs a built-in charger simulator:
uv run python -m wallmonitor --demo
```

Then open <http://127.0.0.1:8480>. All history lands in a single
`wallmonitor.db` SQLite file — back up that file and you have everything.

See [`monitor/README.md`](monitor/README.md) for every option (ports, bind
address, poll intervals, environment variables) and the test suite.

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
