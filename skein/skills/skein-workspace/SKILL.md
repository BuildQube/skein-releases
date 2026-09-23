---
name: skein-workspace
description: Use when the user wants to spin up a new Skein workspace — optionally linked to a GitHub, Linear, or Airtable issue — with a starting prompt pre-loaded into its terminal, or to survey existing workspaces before acting. Triggers on phrases like "start a Skein workspace for issue #123", "spin up a workspace and ask it where the milestone stands", "create a Skein workspace prompted with X", "list my Skein workspaces", "what's the status of workspace X". Requires the Skein desktop app to be running and the `skein` CLI on PATH.
---

# Creating a Skein workspace from a session

Use the `skein` CLI to create a workspace that materializes **live in the running Skein desktop app**, with a starting prompt pre-typed (but **unsent**) into its terminal running `claude`.

## Preconditions — check these first

1. **The Skein desktop app must be running.** If any `skein` command fails with "Skein isn't running", tell the user to open the Skein app, then retry. Do not try to launch it yourself.
2. **The `skein` binary must be on PATH.** Check with `command -v skein`. If it's missing, tell the user to open Skein → Settings → **Companion CLI** and click **Install**, then retry. Do **not** attempt to install or locate the binary yourself.

## Steps

1. **Discover and resolve the project.** Run:
   ```
   skein projects --json
   ```
   This prints `[{ "id", "name", "domain" }, …]`. Pick the project matching the user's intent. If the intended project name is ambiguous (same name in two domains) or you can't tell which they mean, ask the user to choose by `domain/name`.

2. **Create the workspace.** Run:
   ```
   skein new --project <id> [--issue <ref>] [--name "<name>"] --prompt "<the starting prompt>"
   ```
   - `--project` accepts the project **id** (preferred — unambiguous) or its name.
   - `--issue` is **optional**. Include it only when the workspace should be linked to a specific issue; the app fetches the issue from its provider to link it and derive the branch **and workspace name** (both match what the in-app issue picker would produce; an explicit `--name` still wins). Omit it for a prompt-only workspace. Accepted ref shapes:
     - **GitHub**: `owner/repo#N` (e.g. `BuildQube/Skein#123`).
     - **Linear**: a team-key id like `ENG-42`, or a `linear.app` issue URL. Requires the project to have a Linear team connected.
     - **Airtable**: the record's user-facing issue id (e.g. `O2178`), or an `airtable.com` record URL. Requires the project to have an Airtable binding; the bare-id form additionally needs the table profile's issue-id mapping.
     - A `TEAM-123`-shaped ref goes to Linear when the project has a Linear team; otherwise it is tried as an Airtable issue id.
   - `--prompt` is the text pre-typed into the workspace's `claude` terminal. It is staged **unsent** — the user reviews it and presses Enter. Write the prompt as the message you'd want the workspace's agent to start from (e.g. "Where does milestone X stand, and what should come next?").
   - `--name <name>` is optional: it sets the workspace's display name. Without `--issue`, the derived branch comes from the kebab-cased name (e.g. `--name "Payment Retry Spike"` → `skein/payment-retry-spike`).
   - `--branch <name>` is optional; omit it to let Skein derive a branch name. An explicit `--branch` always wins over derivation.
   - There is **no `--base` flag** yet ([#1260](https://github.com/BuildQube/Skein/issues/1260)): a new branch is always cut from the project's default branch. To stack on another branch, create the branch off that base yourself (`git branch <new> <base>` in the project repo), then run `skein new --branch <new> --force`. The "branch already exists" collision is `--force`-able and attaches a workspace to it. The attached workspace records **no base branch**, so tell the user the PR base must be set explicitly, and say so in the prompt.

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
- Any **other** non-zero exit (e.g. a database or spawn error) can leave the workspace already created with its prompt undelivered. Run `skein list --project <id> --json` before suggesting anything. If the workspace is there, give the user its id and the prompt text to paste into its terminal. **Don't suggest re-running `skein new`**: against a live workspace it returns the "already live on this branch" collision, which `--force` deliberately won't touch, so the prompt is never re-delivered.
- The workspace linkage and prefill use the same machinery as Skein's in-app "start from issue" flow, so the result is identical to what the user would get clicking through the UI.
