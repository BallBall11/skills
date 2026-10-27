# Verification Entrypoint

The repo's single command for proving health, and the rules for who runs it when.

## The entrypoint

**`./init.sh`** at the repo root. It runs, in fast-fail order, and stops on the first failure:

- Dependency install
- Static checks (format, lint)
- Typecheck or build
- Tests

Read `init.sh` for the exact commands: they are drawn from this repo's own scripts and targets, and the file is the source of truth when the two disagree.

## Consumer rules

- **`implement` / `implement-spec`**: run `./init.sh` before claiming a feature done. A red `init.sh` means not done, whatever the diff looks like.
- **`code-review`**: run `./init.sh` before reviewing, so a diff is judged on top of a green baseline.
- **Fresh sessions**: run `./init.sh` before the first edit. A stale environment is cheapest to catch before it corrupts work.
- **`retro`**: an `init.sh` that is missing, unwired, or silently broken is a finding.

## Editing this contract

When the check set changes (new test suite, new linter, new build step), update `init.sh` first and the layer list above second, in the same commit. Skills consume both; letting them drift recreates the guessing this file exists to remove.
