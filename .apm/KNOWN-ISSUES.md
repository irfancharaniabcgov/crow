# Known Issues

## Copilot CLI: "Unknown tool name in the tool allowlist: create / edit"

**Symptom:** While running any `.apm` agent (e.g. Crow Testing Agent, Crow B.C. Government UX Agent, Crow
Security & Dependency Review Agent, Crow Security Remediation Agent, Crow Architecture Review Agent, Crow
Agent & Skill Authoring Agent, Crow Executive Summary Report Agent) in GitHub Copilot CLI, you may see log
lines like:

```
Unknown tool name in the tool allowlist: "create"
Unknown tool name in the tool allowlist: "edit"
```

sometimes duplicated.

**Root cause:** These agents declare `tools: [..., 'edit', ...]` in their frontmatter, using the standard,
documented GitHub Copilot custom-agent tool alias (`read`, `edit`, `search`, `execute`, `web` — see GitHub's
[custom agent configuration reference](https://docs.github.com/copilot/reference/custom-agents-configuration)).
Copilot CLI expands the `edit` alias into its own local file-editing tools, which in current CLI versions are
split into two distinct tools, `create` (new files) and `edit` (existing files). The CLI's session
tool-allowlist validator does not recognize these expanded names as grantable, and logs the error above. Per
GitHub's own docs, unrecognized tool names in an agent's `tools` list should be silently ignored, not raise an
error — so this is a **Copilot CLI-side bug** (a mismatch between its alias-expansion layer and its
allowlist-validation layer, most likely introduced when the CLI split a single "edit" tool into separate
`create`/`edit` tools), not a misconfiguration in any `.apm` agent file.

**Impact:** Cosmetic / logging noise only. File create and edit actions performed by the affected agents
still succeed; the engagement is not blocked.

**Workaround status:** Omitting the `tools:` frontmatter property (or using `tools: ["*"]`) is a documented,
supported way to grant an agent all tools, and may avoid whichever CLI code path emits this error. This has
**not been empirically confirmed** to suppress the log lines, because the error is emitted to the CLI's own
runtime/session log and is not observable from within an agent's own tool output. If you want to try it,
temporarily replace an affected agent's `tools:` line with `tools: ['*']` and watch the CLI's log/console
during a real engagement for the error lines. Because it's unverified and would drop each agent's intentional
tool-list documentation value, this repo has not applied it by default.

**If the errors persist:** File an issue with the GitHub Copilot CLI team, referencing this note and the
`edit` → `create`/`edit` alias-expansion mismatch described above.
