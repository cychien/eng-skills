---
name: create-skill
description: Create a new skill or improve an existing one. Captures intent, drafts a SKILL.md with progressive disclosure, tries it on a few realistic prompts, and iterates on feedback. Every skill artifact is written in English. Use this whenever the user wants to write, edit, restructure, or tighten a skill, turn a workflow from the current conversation into a skill, or make a skill trigger more reliably, even if they never say the word "skill".
---

# Create skill

A skill is a folder with a `SKILL.md` that teaches an agent how to do one kind of task well. This skill walks through making one and making it better.

The loop is short:

1. Figure out what the skill should do and when it should fire.
2. Draft the `SKILL.md`.
3. Try it on two or three realistic prompts.
4. Show the user the results and ask what is off.
5. Improve the skill, then repeat until the user is happy.

Find out where the user is in this loop and jump in there. "I want a skill for X" starts at step 1. "Here is my draft, is it any good?" starts at step 3. If the user says they just want to vibe and skip the testing, do that.

## House rules

- **Write every skill artifact in English.** The `SKILL.md`, its references, scripts, comments, and examples are all English, even when the conversation with the user is in another language.
- **Be concise and to the point.** Every sentence must change what the agent does. Cut the rest.
- **Use a plain dash, never an em dash.** Write `-` where you would reach for `—`.
- **Keep `SKILL.md` under about 500 lines.** When it grows past that, move detail into `references/` and point to it.
- **Explain why, not just what.** A sentence that says why a step matters lets the agent handle cases the skill never anticipated. All-caps `ALWAYS` and `NEVER` are a yellow flag that the reasoning is missing.
- **Use the imperative.** "Read the config first", not "the config should be read first".

## Communicating with the user

People who make skills range from engineers to someone who just opened a terminal for the first time. Read the cues. Terms like "evaluation" are fine. Terms like "JSON", "frontmatter", or "assertion" need a short definition unless the user has already used them.

## Capture intent

The current conversation may already contain the workflow the user wants to capture, for example when they say "turn this into a skill". Mine it first: the tools used, the order of steps, corrections the user made, the input and output formats you saw. Then fill the gaps with the user and confirm before drafting.

Four questions to settle:

1. What should this skill let the agent do?
2. When should it trigger? Which phrases, file types, or situations?
3. What does the output look like?
4. Is the output objectively checkable (file transforms, data extraction, fixed workflows) or subjective (writing style, design taste)? Checkable outputs are worth a couple of concrete test prompts. Subjective ones are judged by eye.

Ask about edge cases, example inputs, and dependencies now, before writing anything. If MCPs or search are available, use them to look up prior art and best practices so the user carries less of the burden.

## Write the SKILL.md

### Anatomy

```
skill-name/
├── SKILL.md          required
│   ├── frontmatter   name and description, both required
│   └── body          the instructions
├── scripts/          executable code for deterministic or repetitive steps
├── references/       docs the agent loads only when it needs them
└── assets/           files used in the output: templates, icons, fonts
```

### Frontmatter

- **name**: the skill's identifier, kebab-case, matching the folder name.
- **description**: when to trigger and what the skill does. This is the main triggering mechanism, so every "when to use" fact lives here, not in the body. Agents tend to undertrigger skills, so make the description a little pushy. Instead of "How to build a dashboard for internal metrics", write "How to build a dashboard for internal metrics. Use this whenever the user mentions dashboards, charts, metrics, or wants to display company data, even if they never say 'dashboard'."

### Progressive disclosure

Skills load in three levels:

1. **Metadata** (name and description) is always in context. About 100 words.
2. **The `SKILL.md` body** loads when the skill triggers. Aim for under 500 lines.
3. **Bundled resources** load on demand. Scripts can run without ever being read.

So put the workflow and the decisions in `SKILL.md`, and push long reference material into `references/` with a clear note on when to read each file. A reference file over 300 lines gets a table of contents. When a skill covers several variants (say, AWS, GCP, and Azure), keep the selection logic in `SKILL.md` and give each variant its own reference file so the agent reads only the one it needs.

### Writing patterns

Define output formats with a template:

```markdown
## Report structure
Use this exact template:
# [Title]
## Summary
## Findings
## Recommendations
```

Show examples as input and output pairs:

```markdown
## Commit message format
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### Writing style

Write a draft, then reread it with fresh eyes and cut. Keep the skill general rather than tuned to the one example in front of you. Prefer a short reason over a rigid rule. Delete sentences that do not change what the agent would do.

### Principle of lack of surprise

A skill's contents should match what its description says. Do not build skills that mislead, exfiltrate data, or facilitate unauthorized access. A "role-play as X" skill is fine.

## Try it out

After the draft, write two or three test prompts that sound like something a real user would type, with concrete details: file names, a bit of backstory, casual phrasing. Show them to the user and ask if they look right or if they want to add one.

For a skill with checkable output, also write two to four assertions per prompt: concrete statements about the output that are true only when the skill did its job ("the CSV has a header row", "the summary names the root cause, not the symptom"). An assertion that a wrong output would also pass is worthless. Skip assertions for subjective skills.

Then run each prompt twice, in the same turn:

- **With the skill.** A fresh subagent gets the skill path and the prompt, and saves its outputs to a scratch directory. Fresh context matters, because you wrote the skill and would otherwise fill its gaps from memory.
- **Without the skill.** Another fresh subagent gets the same prompt and no skill path. When improving an existing skill, give it a snapshot of the old version instead. This is the baseline. Without it you cannot tell whether the skill adds anything or the model would have done the same on its own.

Without subagents, read the `SKILL.md` and follow it yourself, one prompt at a time, and skip the baseline. Less rigorous, still useful.

When the runs finish, grade them if there are assertions: a fresh subagent reads each run's outputs and transcript, marks every assertion pass or fail with the evidence it found, and flags assertions that passed too easily or outcomes nothing checked. Grade from the real output, never from the run's own summary.

Present the outputs side by side in the conversation, with the grades when there are any. For files the user needs to open, save them and give the path. Ask inline: "How does this look? What would you change?" Empty feedback means fine. Focus the next revision on the prompts where the user had specific complaints, and on any assertion the skill run failed while the baseline passed.

## Improve the skill

1. **Generalize from the feedback.** The skill will run on prompts nobody has seen yet. A fix that only works for the three test prompts is overfitting. If an issue is stubborn, try a different framing or a different working pattern rather than adding a stricter rule.
2. **Keep it lean.** Read the transcripts, not just the final outputs. If the skill makes the agent do unproductive work, cut the part that causes it and see what happens.
3. **Explain the why.** If the user's feedback is terse, work out what they actually need and write that reasoning into the skill so the agent can apply it to new cases.
4. **Bundle repeated work.** If every test run wrote the same helper script or took the same multi-step detour, write that script once, put it in `scripts/`, and tell the skill to use it.

Then rerun the test prompts and show the user again. Stop when the user is happy, the feedback is all empty, or you are no longer making progress.

## Make it trigger

An agent sees only the name and description when deciding whether to consult a skill, and it consults skills mainly for tasks it cannot handle in one step. A simple "read this PDF" may never trigger a PDF skill no matter how good the description is.

Write six to ten realistic queries: half that should trigger, half that should not. The valuable negatives are near-misses that share keywords but need something else. Then test them instead of guessing: run each query in a fresh session that has the skill installed (`claude -p "<query>"` works in Claude Code) and check the transcript for whether the skill was read. Two or three runs per query, because triggering is not deterministic. Fix misses by changing the description, not the body, and rerun.

## Update an existing skill

Keep the original folder name and `name` field. Snapshot the current version before editing so the user can compare. Edit in place, rerun the test prompts, and show both versions when the change is contested.

## Before you hand it over

- Frontmatter has `name` and `description`, and `name` matches the folder.
- Every file the body references exists.
- Everything is in English, with plain dashes.
- `SKILL.md` is under about 500 lines.
- The description says when to use the skill, and sounds a little pushy.
- The test prompts ran with and without the skill, and the user saw both.
