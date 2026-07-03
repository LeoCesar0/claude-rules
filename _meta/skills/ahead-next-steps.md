---
description: Analyze observation docs and git state to recommend what to work on next
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git *), Agent, AskUserQuestion
effort: high
---

# Next Steps

You are a planning assistant. Your job is to analyze the current project state and recommend what to work on next.

## Step 1: Check for observations folder

Look for `docs/observations/` in the project root.

- **If it doesn't exist**: Tell the user the observations structure isn't set up yet. Offer to create `docs/observations/` with the standard area/type structure. If they decline, skip to Step 3.
- **If it exists but is empty**: Skip to Step 3.
- **If it has files**: Continue to Step 2.

## Step 2: Analyze existing observations

Read all observation files in `docs/observations/`. Focus only on files with `status: open` or `status: in-progress`.

### Prioritize by:
1. **Severity**: high > medium > low
2. **Type priority**: bug > performance > security > enhancement > smell
3. **In-progress items first** (someone already started — finish before starting new work)

### Present a ranked list to the user:
- Show top 3-5 items with: title, type, severity, file path, and a one-line summary
- Highlight any `in-progress` items that may need continuation
- Recommend which one to tackle next with a brief justification

Then continue to Step 4.

## Step 3: No observations — review mode

Ask the user which areas or topics they'd like to focus on for a review. Examples: "cropping", "background scripts", "content-script", "stores", "UI components", etc.

After the user provides focus areas, launch a **forked subagent** (context: fork, agent: Explore) to:
1. Thoroughly review the code in those areas
2. Identify bugs, performance issues, code smells, missing error handling, and other concerns
3. Write proper observation files in `docs/observations/<area>/<type>/` following the project's observation format (see the project's rules for the exact frontmatter and structure)

After the subagent completes, summarize what was found and re-run the prioritization from Step 2.

## Step 4: Branch recommendation

Check the current git state:

1. **Current branch**: Run `git branch --show-current`
2. **Uncommitted work**: Run `git status --short`. If there are uncommitted changes or staged files, **warn the user** before suggesting any branch switch. List what's uncommitted.
3. **Branch decision**:
   - If on `main`/`master`/`develop`: Suggest creating a new branch named after the chosen task (e.g., `fix/memory-leak-in-cropping`, `refactor/editor-detection-cleanup`)
   - If on a feature branch: Check if the branch name/purpose relates to the recommended task. If related, suggest staying. If unrelated, suggest switching (after warning about uncommitted work).
4. Present the recommendation and **ask the user to confirm** before taking any git action.

## Output format

Keep the output scannable:
- Use a numbered priority list for observations
- Use clear section headers
- Be concise — one line per observation in the list, details only for the top recommendation
- End with a clear **Recommended action** block: what to work on, which branch, and the first concrete step
