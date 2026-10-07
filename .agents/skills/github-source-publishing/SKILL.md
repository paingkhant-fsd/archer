---
name: github-source-publishing
description: Prepare and publish a multi-project workspace as independent GitHub repositories for app, API, and mobile clients plus a root repository that tracks them as Git submodules. Use when creating, repairing, or verifying this four-repository GitHub layout.
---

# GitHub Source Publishing

Publish a workspace containing independent app, API, and mobile projects without collapsing their repository boundaries. The result is four GitHub repositories:

- `<project-name>-app`
- `<project-name>-api`
- `<project-name>-mobile`
- `<project-name>` for the root repository

The root repository tracks the three project repositories as Git submodules at their existing directory paths. It must not contain copied snapshots of their source.

## Establish the publication choices

Derive `<project-name>` from established project metadata, documentation, existing remotes, or directory names. Normalize it only as GitHub requires. If those sources disagree in a way that changes repository names, ask the user before creating remote repositories.

Use the target GitHub account or organization already established by the user's authenticated GitHub context or existing remotes. Ask when multiple plausible owners exist.

Create all four repositories as public unless the user explicitly requests private visibility for one or more of them. Confirm repository visibility in the final verification.

A software license is a legal choice. Preserve an existing valid license. If a repository has no license and the user has not already selected one, ask which license to use before publishing; do not silently assume MIT or another license. Apply the selected license consistently unless the user requests repository-specific licensing. Ensure package metadata does not contradict the license file.

## Preflight

Read the workspace `AGENTS.md`, product specification, project READMEs, package manifests, Git state, ignore files, and configured remotes before editing. Treat each child project as an independent repository with its own history, dependencies, tests, environment example, README, ignore rules, and license.

Before staging anything, search for secrets and generated or local-only material. At minimum review environment files, credentials, tokens, private keys, database files, build output, dependency directories, logs, IDE state, and platform-specific generated files. Add narrowly scoped ignore rules where needed. Never commit real secrets; provide sanitized environment examples when the project requires configuration.

Do not overwrite user changes, rewrite existing history, replace a remote, or force-push unless the user explicitly requests it. When a desired GitHub repository already exists, inspect it and reconcile safely rather than assuming it is empty.

## Prepare each child repository

Work on the app, API, and mobile repositories independently.

1. Verify that the directory matches its intended role and repository name.
2. Ensure its README accurately documents purpose, prerequisites, setup, environment variables, run/build/test commands, local endpoints when relevant, and project-specific troubleshooting. Do not move product requirements out of the specification into a README.
3. Add or verify the chosen root-level license file and any matching manifest license field.
4. Verify repository-specific ignore rules and sanitized environment examples.
5. Run the narrowest relevant checks followed by the checks required by the workspace instructions. Report failures instead of concealing them.
6. Initialize Git only if needed. Review the exact staged files, commit the prepared source, create or connect the correctly named GitHub repository, and push the intended default branch.
7. Record the canonical remote URL and pushed commit for root submodule setup.

Finish and publish all child repositories before converting the root paths into submodules. Use available GitHub tooling such as an authenticated GitHub integration or `gh`; if authentication is missing, stop before remote creation and tell the user exactly what is needed.

## Prepare the root repository

The root repository is an orchestration and documentation repository, not an alternate owner of child source.

- Keep cross-project files such as `AGENTS.md`, the product specification, compatible entry-point guidance, and root planning documents when they belong to the complete product.
- Add a root README that summarizes the architecture, links each GitHub repository, lists prerequisites, explains `git clone --recurse-submodules`, documents `git submodule update --init --recursive`, and points setup instructions to each child README.
- Add or verify the selected root license. Explain any intentional license difference between root and child repositories.
- Register the existing app, API, and mobile paths as Git submodules using their canonical remotes. Review `.gitmodules` and ensure each path and URL is correct.
- Ensure child working-tree content is represented only by gitlink entries in the root index. Do not delete a child working tree merely to convert it into a submodule.
- Initialize the root repository only if needed, review the staged root files and gitlinks, commit, create or connect `<project-name>`, and push the default branch.

If the root repository previously tracked child files directly, stop and explain the migration implications before removing those files from its index or rewriting any history.

## Verification and handoff

Verify locally and against GitHub:

- all four repository names, owners, visibility settings, default branches, and remotes;
- each child repository's expected source, README, license, ignore rules, clean status, and pushed commit;
- the root repository's README, license, `.gitmodules`, and three gitlink entries;
- a recursive clone into a temporary directory, confirming all submodules initialize at the committed revisions without relying on local paths;
- absence of obvious secrets and unintended generated files in the published commits.

Return the four repository links, their visibility and license, the commit or branch published for each, checks run and their results, and any follow-up required. Creating repositories and pushing commits are external mutations: perform them only when the user's current request authorizes publication, and do not broaden that authorization to deleting repositories, changing organization policy, or rewriting history.
