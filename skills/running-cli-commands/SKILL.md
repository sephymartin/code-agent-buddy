---
name: running-cli-commands
description: Use when running project CLI commands, debugging command-line failures, or suggesting commands to users in repositories that may define tool versions through mise config files.
---

# Running CLI Commands

## Overview
Use this skill before executing project commands or suggesting command lines to the user. The goal is to avoid version drift, misleading examples, and command failures caused by ignoring repository-defined `mise` toolchains.

## When to Use
- Running build, test, lint, format, install, migration, or code generation commands
- Debugging why a project command behaves differently across machines or shells
- Suggesting commands to the user that depend on Java, Maven, Node.js, pnpm, Python, or similar tools
- Working in monorepos, subprojects, or nested directories where tool versions may differ

Do not use this skill for shell commands that are clearly unrelated to project toolchains, such as reading files, listing directories, or other basic OS utilities.

## Core Workflow
1. Check the current project root, and any relevant subproject root, for `.mise.toml`, `.mise.yaml`, or `.mise.yml`.
2. Read the config before running toolchain-dependent commands or suggesting them to the user.
3. Identify which tools the command depends on, and whether those tools are defined in `mise`.
4. Use `mise exec -- <command>` unless you have direct evidence that the correct `mise` environment is already active.
5. When giving the user a command example, include `mise exec --` or explicitly say that the command assumes an active `mise` environment.

## Detection Rules
- Re-check `mise` config for each new project or subproject scope. Do not assume an earlier check still applies.
- In multi-project workspaces, inspect the specific directory that owns the command you are about to run.
- If both repository root and subproject config exist, prefer the config closest to the command target.

## Command Guidance
Wrap commands with `mise exec --` when they depend on configured tools.

For Maven multi-module projects, prefer explicit module targeting instead of running broad root-level builds. When the task is scoped to one or a few modules, use `-pl` to select the target module and `-am` to build required upstream dependencies.

Examples:

```bash
mise exec -- mvn -pl app-module -am test
mise exec -- pnpm build
mise exec -- python -m pytest
mise exec -- java -version
```

For Maven commands:
- Use `mise exec -- mvn -pl <module> -am <goal>` when compiling, testing, or packaging a specific module inside a multi-module repository.
- Do not default to `mvn <goal>` at repository root when the user request or changed files already identify a narrower module scope.
- If you intentionally omit `-pl` and `-am`, state why, such as a true full-repository build or a single-module project.

If you intentionally do not wrap a command, state why. Acceptable reasons are narrow, such as:
- the command is a shell builtin or basic OS utility
- the command does not use a tool managed by `mise`
- the environment is already known to be activated correctly for that exact project

## Common Mistakes
- Running `mvn`, `pnpm`, `python`, or `java` directly without checking for `mise` config first
- Running root-level `mvn test` or `mvn compile` in a multi-module repository when `-pl` and `-am` should have scoped the build
- Reading only the repository root while the real command target lives in a subproject with its own config
- Using `mise exec` for actual execution but forgetting it in the command example shown to the user
- Assuming local success proves the command is portable across machines
- Suggesting system-default tool versions when the repository has already declared its own versions

## Self-Check
- Did I check for `.mise.toml`, `.mise.yaml`, or `.mise.yml` in the relevant project scope?
- Did I read the config before running or recommending a toolchain-dependent command?
- Did I wrap the command with `mise exec --`, or clearly explain why that was unnecessary?
- If this is a Maven multi-module task, did I scope the command with `-pl` and `-am`, or explain why not?
- If I gave the user a command example, does it match the repository's `mise` expectations?
