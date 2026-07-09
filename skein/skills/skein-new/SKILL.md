---
name: skein-new
description: One-shot quick-create of a prompt-loaded Skein workspace when the target is already explicit — an issue ref (owner/repo#123, ENG-42, an Airtable issue id, or a provider URL) or a workspace name, plus a starting prompt. Triggers on explicit invocation ("/skein-new", "skein new …") and terse quick-create phrasing like "new skein workspace for ENG-42: <prompt>" or "skein workspace named X prompted with Y". For natural-language requests needing judgment — which project, whether a workspace already exists, survey/status questions — use the skein-workspace skill instead.
---

# skein-new — one-shot workspace create

Shared ground rules — preconditions (app running, `skein` on PATH), error handling (**relay the CLI's message and stop**, never retry blindly), and the survey commands — live in [`../skein-workspace/SKILL.md`](../skein-workspace/SKILL.md). Read that file when anything here fails or needs a judgment call. This shorthand assumes the target is already explicit.

1. **Parse the request** into: a project (if stated), an issue ref **or** a workspace name, and the starting prompt.
2. **Resolve the project id**: run `skein projects --json` and pick the matching project. If no project was stated and exactly one exists, use it; otherwise ask the user to pick by `domain/name`.
3. **Create** (one command, no survey first):
   - From an issue: `skein new --project <id> --issue <ref> --prompt "<prompt>"`
   - From a name: `skein new --project <id> --name "<name>" --prompt "<prompt>"`
4. **Report** the created workspace id that `skein new` prints. The prompt is staged **unsent** in the workspace's terminal — say so; never claim it was submitted.
