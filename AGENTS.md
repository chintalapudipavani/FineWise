# BuyWise — Shared Development Instructions

These rules apply to every AI coding agent and both teammates (Aman and Vivek), on every computer. Keep this file in the **repository root** and commit it so everyone works from the same instructions. This file governs development behavior; it does not replace product requirements, architecture decisions, or shared technical contracts.

## 1. Non-negotiable goals

- Preserve the approved BuyWise requirements, architecture, security boundaries, and team task split.
- Prefer the smallest complete, correct change. Reuse existing code and platform features before adding abstractions, dependencies, files, or services.
- Be lazy about code, never about understanding, validation, security, data integrity, or tests.
- Do not silently change APIs, database schemas, contracts, dependencies, environment variables, or architectural decisions.
- Do not invent missing requirements. If a conflict or important ambiguity cannot be resolved from the repository, report it and ask before making a consequential assumption.

## 2. Source of truth and documentation

Before a task, inspect the root instructions and the **relevant** documents in `docs/`; do not reread every document on every task. Use the filenames actually present in the checkout. In particular, consult the applicable versions of:

- Blockers and decisions / decision log for approved decisions, constraints, and unresolved blockers.
- Product requirements and requirement traceability for expected behavior and acceptance criteria.
- Architecture and module design, UML diagrams, and repository map to understand boundaries and existing structure.
- Shared technical contract for API shapes, schemas, naming, errors, and integration assumptions.
- The relevant Aman or Vivek development handbook and independence/conflict guidance for task ownership.
- Parallel development and integration/merge guidance before changing shared files or merging work.
- Implementation plan, progress report, and project audit report when task status or known risks matter.

**Conflict rule:** obey higher-priority system/security requirements first. For project behavior, follow the latest explicitly approved decision and shared contract; do not treat this file as permission to contradict them. If two documents disagree and no approved decision resolves it, stop the affected change, state the conflict with file names, and request a decision. Update documentation only when the task requires it and the responsible owner agrees.

## 3. Start each task with a small context budget

1. Identify the requested outcome, acceptance criteria, assigned owner, and files likely to change.
2. Check `git status` and the current branch. Never overwrite or discard another person's uncommitted work.
3. Read only the relevant instructions, the affected source files, their callers, and the smallest useful tests/configuration. Use the repository map and targeted search rather than dumping the whole repository.
4. Trace the actual execution path before designing a fix. Check whether an existing function, component, utility, dependency, or platform feature already solves the problem.
5. If a task is blocked by an unresolved contract or another teammate, do not guess or build a parallel competing implementation. Report the precise blocker and what decision/input is needed.

Keep investigations bounded: use targeted search, narrow file ranges, and focused test commands. Do not repeatedly print large files or regenerate summaries already documented. Ask the user only for information that is genuinely necessary to avoid an unsafe or incompatible assumption.

## 4. Ponytail: minimum code, not minimum safety

Use the official Ponytail skill/plugin if it is already installed and supported by the current coding agent. The repository rules in this file are the portable fallback, so development must remain consistent even when the teammates use different agents or one has no Ponytail plugin. Do not make a plugin, subscription, new runtime dependency, or machine-specific configuration a prerequisite for building BuyWise.

Follow this order before adding code:

1. Does the behavior need to exist for the stated requirement? If not, do not add it.
2. Is it already implemented or reusable in this repository? Extend/reuse it instead of duplicating it.
3. Can the standard library or native platform feature solve it?
4. Can an already-installed dependency solve it without compromising the architecture?
5. Only then implement the smallest maintainable solution.

Do not install a package, create a wrapper, introduce a design pattern, add a configuration layer, or refactor neighboring code without a concrete need. Do not golf code into cryptic one-liners. Preserve readability and the project's existing conventions. Never remove input validation, authorization, error handling, accessibility, logging needed for diagnosis, data-loss protection, or security checks merely to reduce code or tokens.

If a Ponytail installation is needed, use only the official source: <https://github.com/DietrichGebert/ponytail>. Confirm compatibility with the teammate's coding agent before installing. Do not copy untrusted installers or commit personal machine settings.

## 5. Two-teammate collaboration and independence

- Both teammates must start from the same canonical Git repository and a known shared base commit. Keep this `AGENTS.md` and approved `docs/` files version-controlled and identical across systems.
- At kickoff, inspect `git status --short`. The current screenshot appears to mark many `docs/` files as untracked (`U`); verify this in Git. Untracked files will not automatically reach the other teammate through a clone or pull. The documentation owner must review and commit the intended shared docs (and this file) before parallel work depends on them. Never use `git add .` blindly; stage only reviewed project files. If a required doc is missing or has a different version on either system, pause affected work until both teammates have the same approved copy.
- Follow the existing Aman/Vivek handbooks and task allocation. Work independently only inside the assigned ownership boundaries.
- Use separate feature branches and small commits. Never develop simultaneously in one shared working tree, force-push shared branches, or overwrite a teammate's changes. Push branches or exchange reviewable commits/patches through the agreed Git workflow; do not transfer code by replacing entire folders.
- Before parallel implementation, agree on shared contracts: API endpoints and request/response types, database schema and migration ownership, event formats, authentication/authorization behavior, environment-variable names, error conventions, and module interfaces as relevant to the task.
- Treat shared contracts and migrations as coordinated changes. One designated owner should prepare a shared schema/migration change; the other teammate reviews/consumes it. Do not create competing migrations or independently reinterpret a contract.
- A task that depends on another module may proceed against an agreed interface or small test stub. Do not duplicate the other module's implementation or change its interface without agreement.
- Keep commits scoped to the task. Do not commit secrets, local `.env` values, generated artifacts, unrelated formatting changes, or machine-specific paths.
- Before merging, fetch the latest agreed base, inspect the diff, resolve conflicts deliberately, and run the relevant validation on the integrated result. If the code or docs diverge from the contract, fix or escalate the mismatch before declaring integration complete.

## 6. Implementation and validation

- Make the narrowest complete change; avoid unrelated refactors and speculative features.
- Follow existing style, architecture, error conventions, and dependency versions. Do not change public interfaces unless the requirement and shared contract authorize it.
- Validate external input at trust boundaries and preserve authorization, privacy, and data integrity.
- Add or update the smallest relevant test for changed behavior, especially for branching logic, loops, parsers, money/calculations, persistence, authorization, and security-sensitive behavior.
- Run the focused tests first, then applicable type checks, lint/build checks, and broader tests when practical. If a command cannot be run, say exactly what was not run and why; never claim an unexecuted check passed.
- Review the final diff for unintended files, leaked secrets, contract drift, dead code, duplicated logic, and accidental edits to teammate-owned work.
- Update `docs/` only when behavior, a contract, a decision, task ownership, or progress has actually changed. Keep documentation changes focused and consistent with the code.

## 7. Completion report — required for every task

End with a brief report containing:

- **Changed:** behavior and key files changed.
- **Validation:** exact tests/checks run and their results.
- **Skipped:** relevant checks or work intentionally not done, with a reason.
- **Risks/blockers:** remaining risks, unresolved decisions, integration dependencies, or `None`.
- **Contract/docs impact:** whether any shared contract or documentation changed; name the files when applicable.

Do not declare a task complete if a required acceptance criterion is unmet or a blocking integration issue remains. State the remaining work clearly.
