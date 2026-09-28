# Contributing to Ela

Ela is pre-alpha. Until Phase 1 lands, the most useful contributions are documentation feedback and design review.

## Ground rules

- Read [docs/SECURITY_AND_PRIVACY.md](docs/SECURITY_AND_PRIVACY.md) first. Changes that weaken the approval gate, send data off the machine or expose credentials will not be merged.
- Every new tool needs a risk level (R0 to R3, see [docs/TOOLS_SPEC.md](docs/TOOLS_SPEC.md)) and tests.
- No secrets, personal data or real credentials in commits, tests or screenshots.
- Keep new dependencies minimal and open-source licensed. Record notable choices in [docs/DECISIONS.md](docs/DECISIONS.md).

## Workflow

1. Open an issue describing the change.
2. Branch from `main`: `feat/<topic>` or `fix/<topic>`.
3. Keep pull requests small and focused.
4. Update the relevant document in `docs/` when behavior changes.

## Code style (proposed)

- Python: Black, Ruff, type hints, pytest.
- Dart/Flutter: `dart format`, `flutter analyze`.
- JS/TS: Prettier, ESLint.
- Commit messages: imperative mood ("Add file search tool").
