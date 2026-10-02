# loop-code-review

A Claude Code skill: the changes of the current task get reviewed by a fresh, independent agent.
A pass ends with a verified report, not with edits. The human decides what gets fixed.

Based on [di-sukharev/loop-review](https://github.com/di-sukharev/loop-review) at commit
`be50a74` (MIT). The workflow and the reviewer brief were rewritten on top of it. Techniques
were taken from eight more repositories, see the table below.

## What it is

The usual loop "the reviewer found it, the agent fixed it, next reviewer" is cut in the middle:

1. Separate the changes of the current task from unrelated edits in the working tree.
2. Run the repository's trigger index, if it has one: a "token in the diff, check to run" table
   in its `AGENTS.md`. Checks fire off the shape of the code, not off the reviewer's memory.
3. Run validation (tests, types, linter) before the review.
4. Start one reviewer with no chat history. It gets the path, the task scope, the ticket text
   and the list of things it may reach, but not the author's reasoning.
5. Verify every finding against the quoted line and give it a status: confirmed, rejected
   (with the refuting line or a run result), or unresolved.
6. Report and stop. Only what the human picked gets fixed, then a new reviewer runs.

A review is accepted when validation is green, the latest reviewer has no open findings, and it
scores at least 9.5 out of 10 or says outright that it has no actionable comments. Five passes
at most by default. Hitting the limit, or two passes in a row with nothing but repeats, counts
as "incomplete", not as success.

## What the reviewer does

The brief is 49 lines. In order:

- **Understand first.** The reviewer reconstructs what the change does, how the data flows and
  which invariants it relies on. Whatever it cannot explain is a maintainability finding, with
  the symbol named and the future change that this ambiguity makes risky.
- **Then try to break it.** A list of risk points and the check for each, then four techniques:
  a violated assumption, a seam between components, normal use with a bad outcome, a failure
  chain. Depth scales with the size and risk of the change.
- **Mechanical passes.** The trigger index and an itemized audit: every new comment, every new
  name and every test is checked against the project rule one by one.
- **Tests, reuse, conventions.** Duplicates are searched three ways: who else uses the same
  underlying API, which symbols have a similar name, where such code would live if it existed.
- **The reviewer edits nothing and starts no agents.**

## What the reviewer returns

A finding starts with the verbatim line of code and its `file:line`. Then come the problem, the
trigger and how often it occurs (as a number when the data can show it), a label "verified by
running", "verified by reading" or "assumption", and the concrete fix.

The report always has the same order:

1. Findings, most severe first.
2. Task compliance: missing, extra, implemented wrongly. Each quotes the requirement line.
3. Not verified, and what was missing to verify it.
4. Set aside as outside the task, one line each.
5. Coverage: what was covered and what was not.
6. Understanding summary.
7. Scores.

## Where the ideas came from

| Source | What was taken |
|---|---|
| [di-sukharev/loop-review](https://github.com/di-sukharev/loop-review) | The base: a fresh reviewer with no chat history, "understand first", task-scoped review, the test quality score, score anchors, the pass limit and stagnation rule. From the newer version: marking assumptions in a finding |
| [obra/superpowers](https://github.com/obra/superpowers) | The reviewer may not start agents; the "not verified" and "set aside" blocks; "a spec's silence is not permission" |
| [mattpocock/skills](https://github.com/mattpocock/skills) | Task compliance as a separate axis that is never merged with the other findings; quoting the requirement line |
| [everyinc/compound-engineering-plugin](https://github.com/everyinc/compound-engineering-plugin) | The quoted line as a condition for a finding, including symbols a framework generates; the four attack techniques; incidence measured from data; no rejecting a high-risk finding without evidence |
| [openai/codex](https://github.com/openai/codex) | "May break something elsewhere" counts only when the place is named; citing the rule's file and line |
| [warpdotdev/common-skills](https://github.com/warpdotdev/common-skills) | An itemized audit before the verdict instead of a holistic read |
| [alibaba/open-code-review](https://github.com/alibaba/open-code-review) | A risk plan before the review: what to check and with what |
| [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) | The three search directions for duplicates |
| [sanyuan0704/sanyuan-skills](https://github.com/sanyuan0704/sanyuan-skills) | A coverage block even in a clean report |

Our own: stopping after the report, finding statuses, the trigger index, the reviewer access
block, and the rule that a missing tool is stated in the report instead of skipped silently.

## What is left out on purpose

- **Parallel reviewers and voting.** One reviewer per pass: every agent carries its own context
  and multiplies token spend.
- **A reviewer that fixes its own findings.** That is how the newer version of the original
  skill works. Here an edit happens only after the human decides.
- **Review by another vendor's model.** The code does not leave the machine.
- **A strict structural audit** in the spirit of "find the restructuring that makes the
  branches disappear". The skill does not impose a new architecture and does not ask for
  refactoring for the sake of uniformity.
- **Passing earlier findings to a new reviewer.** Every scoring pass is independent.

## Install

```bash
git clone git@github.com:naliferov/loop-code-review.git
ln -s "$(pwd)/loop-code-review/skills/loop-code-review" ~/.claude/skills/loop-code-review
```

The skill is then invoked as `/loop-code-review`.

## What to give the workspace

The skill works without these, but gets more precise when the project instructions have:

- **A trigger index** in the repository's `AGENTS.md`: a "token in the diff, check to run"
  table for the classes of mistakes that already happened in this repository.
- **A reviewer access block**: how it may read the database, which sibling repositories consume
  a shared contract, where the ticket text and earlier merge request discussions come from,
  whether LSP is available.
- **Written conventions**: a convention finding has to cite the rule's file and line.

## License

MIT, see [LICENSE](LICENSE). Author: Nikolay Aliferov.
