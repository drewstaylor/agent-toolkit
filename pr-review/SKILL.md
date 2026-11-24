---
name: pr-review
description: Template for Pull Request reviews. Use this skill when the user wants to review a PR from GitHub, and has already fetched the PR from the remote and checked out those changes locally.
argument-hint: "[flags]"
disable-model-invocation: true
allowed-tools: Read Grep Git Cd Ls Bash(gh:*) Agent
---

## Feature Flags Arguments:
- $ARGUMENTS: Optional flags

## Options

| Option | Purpose |
|--------|---------|
| (none) | Runs the skill in standard default mode |
| `--compiled` | Skip pausing for feedback between steps; collate all findings into a single report at the end |
| `--docs` | Review a design or documentation PR: replaces steps 2–6 with steps D1–D7 (see Docs mode). Selected automatically when every changed file is Markdown. Combines with `--compiled` |
| `--help` | Show how to use the skill, then stop without reviewing. Add other flags to explain only those, as in `--help --docs` |

## Help (`--help`)

If `--help` is among the arguments, show help, then stop. Run no review steps, and ignore the Pull Request section below.

- **`--help` alone:** print the Options table above, the usage below, and the prerequisites.
- **`--help` with other flags**, such as `--help --docs`: explain only those flags. For each one, give its row from the Options table, then what it changes in the review, which steps it adds or replaces, what it needs before you run it, which flags it combines with, and an example command. Take the details from the flag's own section of this file, such as Docs mode for `--docs`.
- **An unknown flag:** say that it isn't an option, then print the full help.

```text
/pr-review                    Review step by step, pausing for feedback after each step
/pr-review --compiled         Run every step without pausing, and give one report at the end
/pr-review --docs             Review a design or documentation PR (steps D1–D7)
/pr-review --docs --compiled  Docs mode, with one report at the end
/pr-review --help             Show this help
/pr-review --help --docs      Explain only the flags after --help
```

Before you run it:
- Check out the PR's branch locally. The skill reviews the diff between `HEAD` and the repo's default branch.
- The default branch comes from `refs/remotes/origin/HEAD`. If it isn't set, run `git remote set-head origin --auto`.
- `gh` must be signed in, so the skill can read the PR's review threads.
- `--docs` is selected automatically when every changed file is Markdown.

## Pull Request

!`git diff origin/HEAD...HEAD`

## Instructions

Analyze the differences between the PR branch (`HEAD`) and the repo's default branch. Perform the following steps of analysis, 1 step at a time. If the `--compiled` flag is omitted, leave room for feedback from the user between each step. If the `--compiled` flag is included, skip asking for feedback between each step, and instead collate all findings into a single compiled report provided to the user after all steps are completed. Here are the steps to perform:

1. Provide a concise and high level description of what the PR is trying to achieve.
2. Identify the total number of files changed. Explain why the PR changed that quantity and if each changed file was necessary to achieve the PR's intended goals.
3. Determine if there's any place where the PR deviates from the normal best practices of the current repository.
4. Determine any places where the code does not conform to the established patterns of the repo, and suggest how it could be better streamlined.
5. Determine any areas where the code could be improved, optimized or made more concise. Be specific here, rather than general.
6. Launch 3 agents to do the following: each agent will review the PR and scrutinize it with specific goals: agent 1 - reviews if there are any security vulnerabilities, agent 2 - reviews if there are any anti-patterns, assumptions, or logical oversights in the suggested changes; and, agent 3 - reviews if the PR creates any technical debt which could become a problem later on should the PR get merged.

**NOTE:** In cases where any issues are duplicated and repeated throughout the PR, list all line numbers and filenames for each occurrence to make the process of flagging each occurence easier for the PR review process, but describe the problem once (e.g. don't repeat your review assessment at each occurrence).

## Docs mode (`--docs`)

Use this mode for a PR that changes only documentation, such as a design doc. It is selected automatically when every changed file is Markdown. Step 1 runs as above. Steps D1–D7 replace steps 2–6. Don't build or run tests.

Before step 1:
- Fetch the base branch and diff against the remote copy (`origin/<base>...HEAD`), since the local base branch can be stale.
- If the repo has audit rules or review rules, read the base branch's copy (`git show origin/<base>:<path>`), even when the PR edits it. If the PR edits the rules, review that edit too.
- Fetch the PR's existing review threads and bot reviews with `gh`.
- Collect what the doc depends on: the code it describes, on the base branch and in open PRs that it builds on or contradicts; any reference implementation it claims parity with, at the pinned version and at its latest; sibling design docs; and the product requirements or tickets behind it. Ask the user for any that aren't in the repo.

In step 1, summarize what the doc decides as a table (decision → choice), and list what it leaves open.

D1. **Fact-check.** Check every claim about existing code, a reference implementation, or another doc, and cite the source at file:line. Flag each claim that is wrong or out of date. When the reference implementation hasn't built the behavior yet, check its open PRs and tickets for the planned behavior before calling a claim wrong.
D2. **Consistency.** Find conflicts with sibling docs, the repo's rules, merged code, and open PRs. Name both sides, and what has to change on each.
D3. **Numbers.** Recompute every size, cost, count, limit, and performance figure from its inputs, and show the arithmetic. Flag each figure that doesn't reproduce.
D4. **Requirements.** Compare the doc with its product requirements and tickets. List each divergence, whether the doc names it, and who must accept it.
D5. **Decisions.** For each decision, check that the choice is sound, that its alternatives and costs are complete, and whether it can be reversed. Give the most scrutiny to choices that are hard to change later, such as storage layouts, addresses or seeds, signed data, and public interfaces.
D6. Launch 3 agents in parallel. Give each the doc, the base branch's rules, and the references collected above:
  - agent 1: threat-models the proposed design against the repo's rules (cite rule IDs) and for abuse paths: account or input substitution, replay, griefing, denial of service, and compromised roles;
  - agent 2: checks parity with the reference implementation, when the doc claims any: every interface, error, event, and order, with each difference named in the doc and given a reason. With no reference, it checks the design against its stated requirements instead;
  - agent 3: checks that the design can be built as written, within the platform's limits, and its rollout, and names the decisions that box in later work.
  Verify each agent finding against the source before reporting it.
D7. **Prior review.** For each existing review thread and bot concern, say whether you confirm it, deny it, or fail to confirm it, with evidence.

Report writing problems, such as long sentences, undefined terms, or internal IDs in committed files, but never block on them alone.

**Output.** Anchor each finding to a doc line in the diff, and give it a severity: Blocking, Correctness, Consistency, or Clarity. Where a wording change fixes it, include a `suggestion` block that matches the current text exactly. A line outside the diff goes in the review body.