---
name: pr-review
description: Template for Pull Request reviews. Use this skill when the user wants to review a PR from GitHub, and has already fetched the PR from the remote and checked out those changes locally.
argument-hint: "[flags]"
disable-model-invocation: true
allowed-tools: Read Grep Git Cd Ls
---

## Feature Flags Arguments:
- $ARGUMENTS: Optional flags

## Options

| Option | Purpose |
|--------|---------|
| (none) | Runs the skill in standard default mode |
| `--compiled` | Skip pausing for feedback between steps; collate all findings into a single report at the end |

## Pull Request

!`git diff $(git symbolic-ref --short refs/remotes/origin/HEAD)...HEAD`

## Instructions

Analyze the differences between the PR branch (`HEAD`) and the repo's default branch. Perform the following steps of analysis, 1 step at a time. If the `--compiled` flag is omitted, leave room for feedback from the user between each step. If the `--compiled` flag is included, skip asking for feedback between each step, and instead collate all findings into a single compiled report provided to the user after all steps are completed. Here are the steps to perform:

1. Provide a concise and high level description of what the PR is trying to achieve.
2. Identify the total number of files changed. Explain why the PR changed that quantity and if each changed file was necessary to achieve the PR's intended goals.
3. Determine if there's any place where the PR deviates from the normal best practices of the current repository.
4. Determine any places where the code does not conform to the established patterns of the repo, and suggest how it could be better streamlined.
5. Determine any areas where the code could be improved, optimized or made more concise. Be specific here, rather than general.
6. Launch 3 agents to do the following: each agent will review the PR and scrutinize it with specific goals: agent 1 - reviews if there are any security vulnerabilities, agent 2 - reviews if there are any anti-patterns, assumptions, or logical oversights in the suggested changes; and, agent 3 - reviews if the PR creates any technical debt which could become a problem later on should the PR get merged.

**NOTE:** In cases where any issues are duplicated and repeated throughout the PR, list all line numbers and filenames for each occurrence to make the process of flagging each occurence easier for the PR review process, but describe the problem once (e.g. don't repeat your review assessment at each occurrence).