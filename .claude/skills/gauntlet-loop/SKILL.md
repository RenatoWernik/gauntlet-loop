---
name: gauntlet-loop
description: >-
  Use for /gauntlet-loop, "gauntlet this", "make a gauntlet prompt", "loop until it beats X", when the user names a model or effort per role (critic, developers, tester, "Opus high para todos"), when the work must reproduce a source exactly ("exatamente os mesmos textos e imagens"), or when the user wants builders and the critic on separate accounts or quotas (Claudex/ccx, "cota separada"). Turns a goal into one short, paste-ready "gauntlet loop" prompt: a concrete, fetchable quality bar, small judgeable pieces, a builder and a separate harsh critic per piece (plus a tester when the bar has a measurable half), blind comparison, loop until ours wins. Works for builds, writing, code, research, or design. Not for a single one-off review, or for running a Gauntlet process that a project already defines in its own AGENTS.md or docs.
---

# Gauntlet Loop

The user gives a goal. You give back ONE short prompt they can paste into a fresh agent session.

You are not doing the work. You are writing the prompt that makes another agent grind on the work until it beats a real reference.

## Flow

1. **Read the goal.** One line restatement in your head, not on screen.
2. **Set the bar.** If the user supplied a reference, use it. If not, offer **2 or 3 candidate bars**, one line each, and stop. Wait for their pick. Do not write the prompt yet.
3. **Write the prompt.** One block, paste-ready, no preamble, no headings inside it, no narration after it. If the goal asks for builders and critic on separate accounts, read **Split subscriptions** below first and bake its block in - that request cannot be satisfied by the default template.
4. **Offer to run it.** One flat line under the prompt: "I can run this here." Not a question.

If they say run it, you become the lead agent and follow the prompt you just wrote.

## The bar is the whole trick

Everything else in a gauntlet loop is scaffolding. The loop only produces quality if the thing it compares against is real.

A bar has to pass three tests:

- **Named.** A specific thing, not a category. "Stripe's pricing page" works. "Award-winning SaaS sites" does not.
- **Fetchable.** The critic can actually get it - screenshot the live page, read the published piece, run the binary, open the repo, watch the footage. If the agent cannot obtain it, it will hallucinate the comparison.
- **Comparable.** Both can sit side by side and a judge can pick one. If you cannot imagine the A/B, it is not a bar.

Bars by goal type:

| Goal | Bar that works |
|---|---|
| Website, app, UI | The live site of a specific best-in-class product, screenshotted at the same viewport |
| Game, 3D, visual | Real footage or screenshots from a named shipped title |
| Writing | A specific published piece by a named author or publication, same length and format |
| Code, tooling | A named repo's implementation, plus its benchmark or test suite as the measurable half |
| Research, analysis | A named analyst report or a paper's methods section, judged on rigour and coverage |
| Deck, doc, deliverable | A real artifact from a firm known for it, same page count |

When you propose bars, prefer the hardest one the agent can genuinely reach. A bar that is too easy makes the loop exit on round one.

If the goal has a measurable half (load time, token cost, benchmark score, word count, pass rate), name it alongside the reference. Taste plus a number beats taste alone.

### Fidelity bars

When the goal is to reproduce a source exactly (a client asks for "the same texts and images as this page", a migration, a port), the bar has two halves and both must pass:

- **Fidelity, measured.** Extract the source first (its text, its images at the highest resolution available, its links) into files the agents read, with any exclusions the user named listed by name. A tester checks it mechanically: every source string present verbatim, every source image present, nothing excluded anywhere, nothing added beyond what the user allowed. Binary, never judged by eye.
- **Quality, judged blind.** The critic puts ours and the source side by side at the same viewports, labels stripped, and picks the better page.

Write the user's decisions on fidelity conflicts (spelling, labels, a rule of theirs that the source breaks) into the brief before the first builder starts, so neither critic nor tester penalises what the user asked for.

## Prompt template

Adapt the wording every time. Fill the brackets, keep it short, keep the last line.

```
Build [GOAL].

The bar is [BAR]. Get the real thing first and compare against it directly, not against a description of it.

Break this into the smallest pieces that can be improved and judged on their own. For each piece, fan out a builder and a separate critic with fresh context. The critic inspects the actual output, puts it next to the bar blind with the labels stripped, says which one is better, and names the single biggest remaining gap. Then it goes back to the builder.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

Rules for what you fill in:

- Bake the bar in as a concrete, fetchable thing. URL, product name, repo, title.
- Add a budget or cost ceiling line **only if the user named one**. No default cap.
- Add tool names only if the goal needs them (image or video generation, a browser, a deploy target).
- Everything else stays out. No architecture, no file layout, no decomposition, no round count, no stack choice unless the user demanded it. The agent decides those, and it decides better than a spec written before the work started.

## Roles, models and effort

Trigger this section when the user names a model or an effort for one or more roles ("crítico com Opus high, developers com Sonnet", "todos em Opus 5.5 high"), or asks for a tester.

- **Three roles, three contexts.** Builders build one piece each. The **critic** judges blind against the bar and names the single biggest gap. The **tester** owns the measurable half (fidelity, tests, accessibility, performance) and reports pass or fail per criterion with the command that proves it. Critic and tester run in parallel on the same round, never in the builder's context, and never edit the work.
- **Effort is not a per-call setting in Claude Code.** The subagent call takes a model but not an effort. Give each role an agent definition in `.claude/agents/<role>.md` with `model:` (the exact model id the user named, e.g. `claude-opus-5-5`) and `effort:` in its frontmatter, and name those agent types in the prompt. Add one line per role to the prompt: "The critic is the `<name>` agent", and so on.
- **Parallel builders need disjoint files.** Split pieces so no two builders touch the same file in the same round (assets, content, layout), and only the lead agent commits, one commit per piece the critic accepts.
- **A brief file beats a long prompt.** When the user answered several questions before the run, put every decision in one brief file that every role reads first; the prompt stays short and points at it.

## Split subscriptions

Trigger this section when the user wants the builders and the critic billed to different accounts. Phrasings that mean this: "Fable as critic and Astra as developers", "cada um usando sua propria cota", "GPT quota for the builders, Claude quota for the critic", or any mention of Claudex or `ccx`.

One fact decides the whole shape. A Claude Code process has a single `ANTHROPIC_BASE_URL`, and subagents inherit it. Inside a `ccx` session every subagent is therefore a GPT model no matter what you call it, so a critic requested as a subagent is the builder's own model judging its own work on the builder's own quota. That is the exact failure this section exists to prevent, and it fails silently: the run looks correct and one quota pays for everything.

The critic has to be a separate process. In the prompt, replace the template's "fan out a builder and a separate critic" sentence with the builder half only, then append this block verbatim, changing nothing but the two model names:

```
The developers are normal subagents of this session, on its model.

The critic is never a subagent. Every critic pass runs as a separate process through the Bash tool, exactly like this:

  ccx.cmd ask --model fable --allowedTools Read Glob Grep -p "<critique request>"

Give the critic the paths of the files to inspect, not a description of them. On macOS or Linux the command is `ccx ask` instead of `ccx.cmd ask`.

If a critic call fails with a quota or rate-limit error, stop and tell me which account ran out. Never replace the critic with a subagent to keep going: that puts both roles on one model and one quota, which defeats the whole setup.
```

Two details make this fail silently if you get them wrong:

- **`ccx.cmd`, not `ccx`.** The Bash tool on Windows is Git Bash, which will not execute `ccx`. PowerShell and cmd accept either.
- **`--allowedTools` is required.** Headless Claude Code has no way to ask for approval, so without it the critic cannot read the files it is supposed to judge. Read, Glob and Grep cover text and code; visual critique needs more tools named there.

This block is exempt from the length rule. Everything outside it stays as short as always.

Tell the user, in one line each, what to do when a side runs dry, because the loop cannot fix this itself:

| Ran out | Stops | Do |
|---|---|---|
| ChatGPT quota | the builders | `ccx handoff` moves the conversation to the Claude subscription; the loop continues with Opus builders and a Fable critic. Separation of quotas is lost, separation of models is kept. |
| Claude quota | the critic | Move the critic to a subagent on a different GPT lane (`sol`, `terra`), so a different model still judges the work. |

Never put `ccx handoff` inside the prompt as something the agent runs. It replaces the process with an interactive session and would hang a tool call. It is a command for the user, in their own terminal.

## Length and voice

Short enough to read at a glance, around 120 to 180 words, with no headings. The bar and the loop instruction are never cut to save words.

Plain sentences. No bullet lists inside the prompt. It should read like someone telling an agent what perfect looks like and refusing to accept less.

## Portability

`/loop` and `ultracode` are Claude Code features. `/loop` reruns the prompt on an interval or lets the model pace itself. `ultracode` opts the turn into multi-agent orchestration.

For any other agent, swap the last two lines for: "Keep looping until the critic picks ours. Run the builders and critics as parallel subagents." The structure carries over unchanged.

## Two filled examples

**Visual goal.** User: "landing page for my running brand, athletic, green and dark, has to feel alive."

Bars offered: A) Nike's current running campaign page B) On Running's homepage C) Gymshark's product landing page. User picks A.

```
Build a landing page for a running brand. Athletic, peak performance, green and dark, energetic, aimed at a young healthy audience. It needs to be interactive and visually unmistakable.

The bar is Nike's current running campaign page. Screenshot it at desktop and mobile and compare against those directly, not against a description of them.

Break this into the smallest pieces that can be improved and judged on their own - hero, motion, type, colour, imagery, interaction, mobile. For each piece, fan out a builder and a separate critic with fresh context. The critic opens the real page in a browser, puts our screenshot next to Nike's blind with the labels stripped, says which is better, and names the single biggest remaining gap. Then it goes back to the builder.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

**Non-visual goal.** User: "a 2000-word explainer on vector databases for non-engineers."

Bars offered: A) a specific Stripe engineering blog explainer B) a named Julia Evans post C) the Wikipedia article plus a comprehension test. User picks B.

```
Write a 2000-word explainer on vector databases for readers who are smart but not engineers.

The bar is Julia Evans' writing on hard technical topics. Pull three of her actual posts and compare against them directly, not against a description of her style.

Break this into the smallest pieces that can be judged on their own - the opening, each explanation, the diagrams, the analogies, the ending. For each piece, fan out a writer and a separate critic with fresh context. The critic reads ours and hers blind with the bylines stripped, says which one a non-engineer would understand faster, and names the single biggest remaining gap. Then it goes back to the writer.

The critic should be a harsh critic. Praise is not useful. If ours does not win, it keeps going.

/loop on each piece until the critic picks ours blind. Do not stop before that.

Keep a live progress page updating as the work evolves so I can watch it.

Fan out subagents and ultracode.
```

## What breaks a gauntlet loop

- **A vague bar.** The critic invents a comparison and approves everything. Most common failure by far.
- **The builder judging its own work.** The critic must be a separate agent with fresh context. It should not know how hard the builder tried.
- **A soft critic.** Say "harsh" in the prompt and give it a binary job: which one is better, A or B. Scores out of 10 drift upward every round.
- **Named exit after N rounds.** The exit is winning the comparison, or the user stopping the run. Never a round count you invented. When the user names a ceiling ("máximo 2 rondas por peça"), it is a stop, not a pass: the piece halts and the report names the gap the critic still sees.
- **A cross-account critic asked for as a subagent.** Subagents share the session's backend, so the critic silently becomes the builder's model on the builder's quota. See **Split subscriptions**.
- **Over-specifying.** Every extra instruction is one fewer decision the agent makes with its own judgment. Minimal wins.
