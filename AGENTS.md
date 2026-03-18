# AGENTS.md

This file captures non-obvious intent for contributors and IDE agents working in this repository.

## Product and Domain Source of Truth

Product/domain semantics are defined in the docs and should not be re-specified here.
Use these as authoritative references:

- `docs/mvp_v1/specification.md` for MVP goals, scope, behavior, and roadmap constraints
- `docs/mvp_v1/schema/reference_domain.md`
- `docs/mvp_v1/schema/corporation_domain.md`
- `docs/mvp_v1/schema/market_domain.md`
- `docs/mvp_v1/schema/configuration_domain.md`
- `docs/mvp_v1/schema/operational_telemetry_domain.md` for telemetry semantics (source identity, refresh cadence, run status, logging, redaction)

## Branch Strategy

Repository branch model:

- `main`: stable branch
- `dev`: integration branch for ongoing work
- most edits are made directly to `dev`
- feature branches are optional and may be created at developer discretion

Changes are typically merged into `dev` first. Periodic PRs are created from `dev` to `main` to promote stable increments.

## Commit and PR Rules

- Do not commit directly to `main`.
- Direct commits to `dev` are allowed and are the normal path in this repository.
- Feature branches may be used at developer discretion, typically branching from `dev` and merging back to `dev`.
- Use PRs for promotion from `dev` to `main`.
- Keep branches and PRs scoped to a single purpose; avoid mixing unrelated work.
- Keep commits scoped to one logical change whenever practical.
- Avoid mixing unrelated code, tests, docs, and config changes unless required for one atomic change.
- Prefer semantic commit messages (Conventional Commits), for example: `feat:`, `fix:`, `refactor:`, `chore:`, `ci:`.
- Draft PRs are recommended by default, but this is guidance rather than a hard requirement.
- Do not add AI attribution to commit messages (AI usage is documented in README).

## Explicit Instruction Mode

- Do not edit, create, rename, or delete files unless the user explicitly asks for it.
- Do not run shell/system commands unless the user explicitly asks for them, or explicitly asks for an action that clearly requires commands.
- Do not infer execution permission from context, earlier turns, or likely next steps.
- Treat proposal-style language (for example "how about", "what if", "should we", "would it make sense") as discussion by default.
- Treat question-form requests (for example "can you", "could you", "is it possible") as discussion by default.
- Treat declarative requirement statements (for example "it should...", "this needs to...") as non-executable unless paired with a clear execution cue.
- After proposal/question discussion, require an explicit execution cue (for example "implement this", "go ahead and make this change") before making changes.
- If intent is ambiguous, ask a short clarifying question before taking action.
- Default to read-only discussion/planning until explicit execution direction is provided.

## Plan Mode Rules

When operating in plan mode:

- **Always use ExitPlanMode tool** before making any file modifications (edits, creates, deletions, renames).
- Present a complete plan to the user and wait for explicit approval.
- Do not make edits during the planning phase, even if the task seems straightforward.
- Plan mode does not bypass explicit instruction mode - both apply simultaneously.
- Making edits without using ExitPlanMode violates the explicit instruction mode rules above.
- This applies to ALL file modifications: code, documentation, configuration, tests, and any other files.
- After approval via ExitPlanMode, proceed with implementation using appropriate tools.

## Safety and Transparency Rules

- Before any change, state exactly what will be done.
- For bug/failure remediation (including CI/workflow failures), explain the proposed fix and ask for explicit confirmation before editing files.
- When changing GitHub Actions/workflow behavior, ensure this `AGENTS.md` remains aligned with the actual workflow triggers and rules; update this file in the same change when needed.
- Guardrails may be bypassed only after explicit user verification in a separate interaction beyond the original request.
- Any bypass confirmation request must include a brief overview of the specific guardrail(s) being bypassed.
- After any change, summarize exactly what changed and where.
- If a requested action conflicts with these rules, explain the conflict and request confirmation before proceeding.

## Git Commit and Push Rules

- Do not create git commits unless the user explicitly requests it.
- Do not push to remote unless the user explicitly requests it.
- Treat requests like "save this" or "update the file" as file operations only, not commit operations.
- Only commit/push when the user uses explicit git-related language (for example "commit this", "push these changes", "create a commit").
- When unsure whether the user wants commits created, ask for clarification.
