---
name: test-coverage-check
description: Audit a repository's existing automated test coverage without executing tests or implementing fixes, then write an evidence-based, timestamped Markdown handoff report. Use for test coverage checks, coverage gap reviews, or requests to identify important missing tests; do not use when the user wants tests implemented or run.
---

# Test Coverage Check

Inspect the repository's existing tests and important implemented behavior, then produce a prioritized report that another agent can use to implement improvements.

## Hard boundary

This is a static inspection workflow.

- Do not execute unit, integration, end-to-end, smoke, snapshot, or coverage tests.
- Do not add, edit, delete, or regenerate tests, fixtures, snapshots, product code, dependencies, lockfiles, build configuration, test configuration, or product documentation.
- Do not fix defects found during the audit.
- Do not install packages or start applications, databases, emulators, or servers.
- The only permitted repository mutation is creating the report described below. Preserve all existing user changes.
- Read-only discovery commands are allowed. Prefer `rg`, `rg --files -uu` with dependency/build exclusions, package-manifest inspection, and read-only Git status/diff commands.

If the request also asks for implementation or test execution, record those items as handoff work and stop after writing the report. Do not broaden this skill into implementation.

## Audit method

1. Find the project root and read its repository instructions. Read the product authority, status/TODO files, relevant READMEs, manifests, and test configuration when present.
2. Inventory tests across every repository in scope, including normally ignored or hidden paths while excluding dependency, generated-client, build, and coverage-output directories.
3. Inspect the implemented source paths that own important behavior. Prioritize authentication, authorization, validation, lifecycle transitions, sensitive-data exclusion, persistence, API contracts, error states, currencies, localization, and cross-project boundaries when applicable.
4. Map implemented behavior to specific existing tests. Distinguish:
   - **Covered:** a test statically demonstrates the important success and failure behavior.
   - **Partial:** some meaningful behavior is tested, but a material branch, invariant, or integration boundary is absent.
   - **Missing:** no relevant repository-owned test was found.
   - **Unknown:** evidence is insufficient without execution or generated coverage data.
5. Inspect test quality, isolation, determinism, assertions, and whether the tests exercise the owning abstraction. Do not equate test count with meaningful coverage.
6. Record product defects noticed during inspection as findings, but do not change them. Separate current implemented scope from future roadmap features so unimplemented product work is not mislabeled as a test gap.
7. Recommend the smallest high-value tests. Give each handoff item a priority, owning project, behavior to prove, and evidence path. Avoid prescribing tests for every trivial unit.

Never report a numeric line, branch, function, or statement percentage unless an existing coverage artifact contains that number and its age/source are stated. The presence of tests does not prove that they pass. Label runtime status **not verified** unless trustworthy existing evidence says otherwise.

## Report artifact

Create `<project-root>/test-coverage-check-reports/` when absent. Write one new file per audit using the user's local timezone:

`YYYY-MM-DD_HHmmss-test-coverage-check.md`

Use a Windows-safe timestamp and never overwrite an earlier report. The report must be self-contained and include:

1. Audit timestamp, scope, and the explicit statement that no tests were run.
2. Executive assessment and runtime-verification status.
3. Test/tooling inventory by project.
4. Coverage map for important implemented behavior using Covered/Partial/Missing/Unknown.
5. Prioritized findings with evidence paths and concise impact.
6. An implementation handoff queue identifying the owning project and suggested proof, without making the changes.
7. Limitations, including unavailable numeric coverage or stale artifacts.

Use repository-relative paths in the report so it remains portable. Keep findings factual and traceable to inspected files. In the final response, link the new report and summarize only its most consequential conclusions.
