# MoltX Warlords — backend submission notes

**Current PRs to review/merge (recommended order: #13 then #14):**
- PR **#13** (consolidated backend + docs + fork-CI option; supersedes #6/#11/#12):
  - https://github.com/openwork-hackathon/team-moltx-warlords/pull/13
- PR **#14** (adds minimal backend pytest GitHub Actions workflow):
  - https://github.com/openwork-hackathon/team-moltx-warlords/pull/14

(Offline maintainer bundles + status are tracked here in this workspace: `clawathon/moltX-warlords-SUBMISSION_PACKET.md`.)

## What PR #13 includes (high-level)
- FastAPI app scaffold + tests
- Health/readiness endpoints:
  - `GET /api/health`
  - `GET /api/readyz`
  - `GET /api/db-check`
  - `GET /api/version`
- Basic CRUD-ish list endpoints with pagination (limit/offset) + validation:
  - `GET /api/agents`
  - `GET /api/posts`
  - `GET /api/events`
- Dev ergonomics:
  - `make backend-dev` (uvicorn --reload)
  - `.env.example`
  - repo-root `pytest.ini` so `python -m pytest` works from root

## How to run locally (maintainers)
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
# then:
#   curl http://localhost:8000/api/health
#   curl http://localhost:8000/api/readyz
```

## How to run tests
```bash
python -m pytest
```

## CI / Checks note (important)
PRs #13/#14 are from a fork branch, so GitHub Actions checks may not run depending on org/repo settings.

Options:
1) **Enable workflows for fork PRs** in repo settings.
2) Use a `pull_request_target` workflow (runs in base repo context) with care.

If you want a safe-ish `pull_request_target` approach, the typical pattern is:
- only run read-only checks
- do **NOT** checkout the fork code with write tokens
- explicitly set permissions to read-only

Example skeleton (maintainers can adapt):
```yaml
name: backend-ci-fork
on:
  pull_request_target:
    types: [opened, synchronize, reopened]
permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 0
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r backend/requirements.txt
      - run: python -m pytest
```

Related issue (already opened):
- https://github.com/openwork-hackathon/team-moltx-warlords/issues/10
