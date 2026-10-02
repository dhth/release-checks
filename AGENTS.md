# release-checks

Smoke-tests the prebuilt release binaries of @dhth's CLI tools. mise installs each tool from its GitHub release, and CI runs `<tool> --help` on Linux (x64 and arm64, Ubuntu 22.04 and 24.04) and macOS (arm64). Checks are triggered manually.

## Layout

- `mise.toml`: tools under test (`[tools]` under `# tools to check`, plus a `[tool_alias]` entry `github:dhth/<tool>`), repo tools, and lint tasks
- `mise-tasks/resolve`: resolves `all` or a tool name to the tools to check
- `mise-tasks/report`: prints a Markdown results table for a workflow run
- `.github/workflows/check.yml`: `workflow_dispatch` with a tool dropdown; `plan` -> `check` (tool x OS matrix) -> `report` (job summary)

## Adding or bumping a tool

1. Find the latest release: `gh release view -R dhth/<tool> --json tagName`. Use the tag without the leading `v`.
2. Set the version under `[tools]` (`# tools to check`). For a new tool, also add `<tool> = "github:dhth/<tool>"` under `[tool_alias]`. Keep both sorted.
3. Run `mise lock <tool>`. Confirm the tool has `url` and `checksum` entries for `linux-x64`, `linux-arm64`, and `macos-arm64`.
4. For a new tool, add it to the `tool` dropdown in `check.yml`: `all` first, then alphabetical. The options must match `mise run resolve all`.
5. If a platform asset is missing, add a matrix `exclude` in `check.yml` with a comment saying why.

## Invariants

- The binary name must match the tool name, since CI runs `<tool> --help`.
- Only `github:dhth/*` tools are checked. Repo tools (actionlint, shellcheck, yamlfmt) come from the mise registry and are ignored by `resolve`.
- `report` parses the `check` job name `"<tool> / <os>"`. Change both together.
