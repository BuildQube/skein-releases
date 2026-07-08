---
name: skein-workspace
description: Use when the user wants to spin up a new Skein workspace — optionally linked to a GitHub issue — with a starting prompt pre-loaded into its terminal, or to survey existing workspaces before acting. Triggers on phrases like "start a Skein workspace for issue #123", "spin up a workspace and ask it where the milestone stands", "create a Skein workspace prompted with X", "list my Skein workspaces", "what's the status of workspace X". Requires the Skein desktop app to be running and the `skein` CLI on PATH.
---

# Creating a Skein workspace from a session

Use the `skein` CLI to create a workspace that materializes **live in the running Skein desktop app**, with a starting prompt pre-typed (but **unsent**) into its terminal running `claude`.

## Preconditions — check these first

1. **The Skein desktop app must be running.** If any `skein` command fails with "Skein isn't running", tell the user to open the Skein app, then retry. Do not try to launch it yourself.
2. **The `skein` binary must be on PATH.** Check with `command -v skein`. If it's missing, tell the user to open Skein → Settings and run the **"Install `skein` CLI"** action, then retry. Do **not** attempt to install or locate the binary yourself.

## Steps

1. **Discover and resolve the project.** Run:
   ```
   skein projects --json
   ```
   This prints `[{ "id", "name", "domain" }, …]`. Pick the project matching the user's intent. If the intended project name is ambiguous (same name in two domains) or you can't tell which they mean, ask the user to choose by `domain/name`.

2. **Create the workspace.** Run:
   ```
   skein new --project <id> [--issue <owner/repo#N>] --prompt "<the starting prompt>"
   ```
   - `--project` accepts the project **id** (preferred — unambiguous) or its name.
   - `--issue` is **optional**. Include it (e.g. `--issue BuildQube/Skein#123`) only when the workspace should be linked to a specific GitHub issue; the app fetches the issue to link it. Omit it for a prompt-only workspace.
   - `--prompt` is the text pre-typed into the workspace's `claude` terminal. It is staged **unsent** — the user reviews it and presses Enter. Write the prompt as the message you'd want the workspace's agent to start from (e.g. "Where does milestone X stand, and what should come next?").
   - `--branch <name>` is optional; omit it to let Skein derive a branch name.

3. **Report the result.** On success `skein new` prints the created workspace id — relay it. The workspace is now open in the app, its terminal booting `claude` with the prompt pre-loaded and unsent.

## Surveying existing workspaces

Before creating — or when the user asks where something stands — read the
current workspace state instead of guessing:

- **List them all.** `skein list [--project <id|name>] --json` prints every
  active/pending workspace across all projects (archived excluded), each with
  its `project`, `domain`, `branch`, `status`, and `linked_issue` (the
  milestone). Filter to one project with `--project`.
- **Inspect one.** `skein status <ws> --json` prints one workspace's full state
  — branch/base, worktree path, origin, the linked issue (title/state/url), and
  any setup error. `<ws>` is a workspace **id** (preferred) or a **name** that
  is unique across all projects; an ambiguous name exits non-zero and lists the
  matching ids — retry with one of them.

Use these to answer "where does milestone X stand?" before deciding whether to
create a new workspace or point the user at an existing one. Both are read-only.

## Important notes

- This **stages** the prompt; it does **not** run it. The user reviews and sends it. Never tell the user the prompt has been submitted.
- On a worktree collision, an unknown/ambiguous project, or the app not running, `skein` exits non-zero with a clear message — **relay that message and stop**; don't retry blindly or guess a different project.
- The workspace linkage and prefill use the same machinery as Skein's in-app "start from issue" flow, so the result is identical to what the user would get clicking through the UI.
