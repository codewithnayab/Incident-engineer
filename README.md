# Incident Case Format

Each incident in `cases/case_NNN_<slug>/` is a self-contained, reproducible debugging scenario.

## Directory Structure

```text
case_NNN_<slug>/
├── case.json              # Machine-readable metadata and verification spec
├── incident.md            # Human-readable incident report provided to the agent
├── repo/                  # Buggy, self-contained Python project
│   ├── <package>/
│   ├── tests/
│   └── README.md
├── logs/
│   └── incident.log       # Captured output from the failing run
├── ground_truth.patch     # Official minimal fix
└── build_incident.sh      # Script used to reproduce the incident
```

## Verification Specification (`case.json`)

Each case includes a `case.json` file that defines its metadata, constraints, and verification requirements.

```json
{
  "id": "case_001",
  "slug": "short-name",
  "title": "...",
  "severity": "P1",
  "service": "affected component",
  "description": "One-line incident summary",
  "tags": ["timezone", "datetime"],
  "challenging": false,
  "budget_steps": 60,
  "check": {
    "test_files_immutable": ["tests/"],
    "must_pass": ["tests/test_payments.py"],
    "must_not_contain": [
      "# noqa",
      "pytest.skip",
      "except: pass"
    ]
  }
}
```

### Verification Workflow

The verifier performs the following steps:

1. Copies `repo/` into a fresh workspace.
2. Allows the agent to modify the copied repository.
3. Runs `pytest -q` inside the workspace.
4. Requires all tests listed in `must_pass` to pass.
5. Ensures directories listed in `test_files_immutable` remain byte-identical to their original versions.
6. Rejects fixes containing any markers specified in `must_not_contain` to prevent test bypassing or other anti-cheat behavior.

## Rules for a Good Incident Case

Every incident case must meet the following requirements:

- **Self-contained:** Use only the Python standard library. No network access or external dependencies should be required during evaluation.
- **Small and focused:** Include 2–8 source files.
- **Reproducible:** Running the test suite before applying the fix must fail on the tests specified in `must_pass`.
- **Minimal ground truth:** The official patch should fix only the underlying bug, without unrelated changes.
- **Realistic incident report:** `incident.md` should read like a genuine on-call report and include:
  - The observed symptom
  - Relevant error excerpts
  - The affected service or component
  - The user-visible impact
- **No root-cause spoilers:** The incident report must not explicitly reveal the root cause. The agent should investigate and identify it independently.
- **Authentic evidence:** Logs must contain the actual captured output from the failing run. Do not fabricate or manually alter incident evidence.

The goal is to create realistic, reproducible debugging incidents that evaluate an agent's ability to investigate failures, identify root causes, and implement correct fixes without modifying or weakening the tests.