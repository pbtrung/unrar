---
description: Commit staged/modified changes with a detailed message and push, no AI co-author attribution
---

# Commit and Push

This repo is a mirror of the upstream RARLAB UnRAR C++ sources (flat layout,
`makefile` for Unix, `UnRAR.vcxproj`/`UnRARDll.vcxproj` for Windows). There is
no `.gitignore`, no formatter config, and no test suite.

## Steps

1. Run `git status` and `git diff` (and `git diff --staged` if anything is already staged) to see all changes.
2. If any `*.cpp`, `*.hpp`, or `makefile` changed, verify the build before staging:
   - `make clean && make -j$(nproc)` — must finish with no errors. Also run
     `make clean && make lib -j$(nproc)` if `dll.*` or anything guarded by
     `RARDLL` changed.
   - Run `./unrar` once to confirm the binary starts and prints its usage banner
     (version should match `version.hpp`).
   - Then `make clean` again so build products (`*.o`, `unrar`, `libunrar.so`)
     don't show up as untracked files — there's no `.gitignore` to hide them.
   - If the build fails, fix it and re-run before continuing.
   - **Do not** run `clang-format` or any other reformatter over the sources —
     they follow upstream's style, and reformatting would bloat diffs against
     future upstream releases.
3. If nothing is staged, stage the relevant modified/new files with explicit
   paths (`git add <files>`). Never stage build artifacts (`*.o`, `unrar`,
   `libunrar.so`, `*.a`) or `.claude/` unless the user asks.
4. Write the commit message:
   - **Upstream version sync** (the diff is a drop-in of a new RARLAB release,
     with `version.hpp` bumped): use the existing convention exactly —
     `Updated to <major>.<minor>.<patch>`, e.g. `RARVER_MAJOR 7`,
     `RARVER_MINOR 30`, `RARVER_BETA 1` → `Updated to 7.3.1` — no body needed.
   - **Any other change** (local patch, build tweak, tooling): write a
     **detailed** message:
     - Subject line: concise summary in imperative mood (e.g. "Add", "Fix", "Refactor").
     - Body: explain _what_ changed and _why_, as bullet points if there are multiple distinct changes.
   - Base the message only on the actual diff — do not include conversational back-and-forth, dead ends, or trial-and-error from the session.
5. Create the commit using a HEREDOC so formatting is preserved, e.g.:
   ```bash
   git commit -m "$(cat <<'EOF'
   Short summary of the change

   - Detail one
   - Detail two
   - Why this change was made
   EOF
   )"
   ```
   (The user's shell is fish — wrap it in `bash -c '...'` or write the message
   to a temp file and use `git commit -F <file>` if heredoc syntax fails.)
6. **Do not** add any AI attribution — no `🤖 Generated with Claude Code` line, no `Co-Authored-By: Claude` trailer, no mention of Claude/AI anywhere in the message.
7. Push the commit to the current branch's remote (`git push`, or `git push -u origin <branch>` if it has no upstream yet).
8. Confirm success by showing `git log -1` and `git status` after pushing.

## Rules

- Never include Claude/AI co-authorship or attribution in the commit message.
- Always push after committing — don't stop at just the local commit.
- If the push fails (e.g. diverged branch), report the error and ask before force-pushing or rebasing.
- Never reformat upstream sources or commit build artifacts.
