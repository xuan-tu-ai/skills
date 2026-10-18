---
name: gemini
description: Hand off a scoped implementation, code review, or technical second opinion to Gemini through the oh-my-pi (`omp`) CLI from Claude Code. Gemini always runs with full autonomy (yolo). Use when the user asks to delegate work to Gemini.
disable-model-invocation: true
---

# Gemini handoff (via omp)

Gemini runs through `omp` in non-interactive print mode with **full autonomy**: every tool enabled
(read, edit, write, bash, …) and all approvals skipped. The user chose this on purpose — do not
downgrade to a read-only or restricted tool set.

## Prepare the task

1. Check `command -v omp`. If missing, tell the user to install oh-my-pi and run `omp login`
   (Google Antigravity). Do not claim a handoff ran when the CLI is unavailable.
2. Read the repository instructions and inspect `git status --short`. Identify the working
   directory, the task (review, advice, or implementation), the files involved, and acceptance
   criteria.
3. Because Gemini can change anything, commit nothing yourself but note the current
   `git status` / `git stash list` so the diff afterwards is attributable. If the directory is not a
   git repo, warn the user that changes cannot be easily reverted.
4. Write a focused prompt to a temporary file (scratchpad, not the repo) containing the user's
   task, relevant paths, constraints, and required checks. Include enough context for a separate
   agent without passing credentials or unrelated private material. Tell Gemini which files are
   already modified and to preserve those changes, and to finish with a short summary of what it
   changed and what it verified.

## Run Gemini

Default model is `gemini-3.8-flash`. If the user names another model (e.g. `pro` →
`gemini-3.1-pro-preview`), pass that instead; `omp models list` shows what is available.

Attach the prompt file with `@` instead of interpolating it into shell text, and always close
stdin. Replace the example paths with the actual absolute paths:

```sh
omp -p --no-session --auto-approve --model=gemini-3.8-flash \
  --cwd='/absolute/project/path' \
  @'/absolute/scratchpad/gemini_task.md' \
  "Carry out the task in the attached file." < /dev/null
```

- Do not pass `--max-time` unless the user asks for a time limit; a hard task must be allowed to
  finish. Always run `omp` as a background command (the foreground shell times out after
  10 minutes) and wait for the completion notification instead of polling.
- Add `@path` arguments for images or PDFs Gemini should look at.
- Use `--add-dir` only when the task genuinely spans another directory.

Wait for completion and read the exit status and output. Do not let Claude and Gemini edit the
same files concurrently. For parallel work, give each worker its own git worktree
(`git worktree add`) and explicit file ownership, run each `omp` with `--cwd` set to its
worktree, and integrate only after reviewing each worker's diff.

## Verify and report

For a review, inspect the cited code and assess each finding before presenting it as confirmed.
For implementation, inspect `git diff` (including untracked files and anything outside the
expected files) and run the relevant checks yourself. Report what Gemini changed or found, what
was verified, anything it touched unexpectedly, and any remaining failures. Do not automatically
commit, push, or deploy unless that is part of the user's task.

Task: $ARGUMENTS
