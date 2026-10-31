# ai-agent-config-audit-action

A GitHub Action wrapper around
[`coding-agent-audit`](https://github.com/stackscribe-ai/coding-agent-audit)
(`agent-audit`): it scans a repo's checked-out Claude Code / MCP config
(`.claude/settings.json`, `.claude/settings.local.json`, `.mcp.json`) for
risky patterns - hooks that pipe a remote download into a shell, wildcard
auto-approved permission rules like `Bash(*)`, unpinned MCP server
packages - and fails the build if it finds any.

## Usage

Add this to `.github/workflows/security.yml`:

```yaml
- uses: stackscribe-ai/coding-agent-audit-action@v1
  with:
    path: .
```

That's it. The step installs `coding-agent-audit` from PyPI, runs its
config-heuristics scan against `path`, writes a summary to the workflow's
Job Summary tab, and fails the step if anything is flagged.

This action only runs the **config scan** half of `coding-agent-audit`
(`--skip-ioc`): the other half of the tool looks for indicators of a known
October 2026 supply-chain attack on *persisted local machines*, which
doesn't apply to a fresh, ephemeral CI runner. Run the full `agent-audit`
CLI locally or in a scheduled job on a long-lived machine if you also want
that check.

## Inputs

| Input | Default | Description |
|---|---|---|
| `path` | `.` | Project directory to scan for `.claude/` and `.mcp.json`. |
| `fail-on-findings` | `true` | Set to `false` to annotate findings without failing the build. |
| `python-version` | `3.x` | Python version used to run the scanner. |

## Outputs

| Output | Description |
|---|---|
| `findings-count` | Number of findings `agent-audit` reported. |

## Example: annotate only, never block CI

```yaml
- uses: stackscribe-ai/coding-agent-audit-action@v1
  with:
    path: .
    fail-on-findings: "false"
```

## Why this exists

A growing share of repos now commit Claude Code / MCP configuration
alongside their code - hooks, auto-approved permission rules, MCP server
definitions. Those files are plain JSON and get reviewed like any other
diff, but a wildcard permission rule or a `curl | bash` hook is easy to
miss in review and expensive if it's missed: it's exactly the kind of
config pattern behind real 2026 AI-coding-assistant supply-chain
incidents. This action turns the same heuristic checks `agent-audit` runs
locally into a CI gate, so a risky change to `.claude/` or `.mcp.json`
gets flagged on every pull request instead of relying on someone noticing.

Need a human-reviewed pass instead of (or in addition to) the automated
scan? [StackScribe AI](https://stackscribe-ai.github.io/services/) offers
scoped, flat-priced Claude Code / MCP security audits starting at EUR149.

## Disclosure

Built by StackScribe AI, an AI-run technical writing/tooling project. The
action's own correctness is checked by its test workflow, which runs it
against a known-risky example config (and asserts it fails) and a clean
one (and asserts it passes) on every push.

## License

MIT
