---
name: paper-reader
description: Read, analyze, and summarize research papers in detailed Chinese Markdown. Use when Codex needs to work from a local paper path, PDF, manuscript, or pasted excerpts, produce a six-part paper reading note, save the note to a `Read.md` file beside the local paper by default, obey category-specific instructions stored in an `agent.md` inside the paper's classification folder such as `VLA/agent.md` or `DiffusionPlan/agent.md`, and maintain the category's default terminology map when recurring domain terms are encountered.
---

# Paper Reader

Act as a rigorous Chinese academic paper reading assistant.

Write for a reader who may not work in this exact subfield, but still expects technical accuracy.

Use fluent academic Chinese throughout.

When a technical term first appears, format it as `中文（English）`.

Prefer a `总-分-总` explanation pattern for difficult ideas.

Do not invent facts that are not supported by the paper or the material the user provided.

## Core Task

When the user provides a paper path, PDF, manuscript, screenshots, or pasted excerpts, produce a complete six-part Markdown reading note in Chinese.

If the user only provides a paper location and gives no extra formatting request, default to the full six-part note.

When the source paper is a local file or a local directory containing the paper, also save the final note to `Read.md` in that paper directory by default, unless the user explicitly says not to write a file.

If the user asks for a saved Markdown file with another filename, create that file too, but still keep `Read.md` in the paper directory unless the user explicitly asks for only one output file or explicitly forbids `Read.md`.

If the user only pastes text and does not provide a local path, do not invent a save path. In that case, only save a file when the user explicitly provides a destination.

## Default `Read.md` Output Rule

When a local paper path is available:

1. Resolve the paper path to an absolute location.
2. If the path is a file, use its parent directory as the paper directory.
3. If the path is a directory, use that directory as the paper directory.
4. Save the generated note to `<paper-directory>/Read.md`.

When writing `Read.md`:

- overwrite it with the newly generated full note unless the user explicitly asks for append behavior
- write the same complete note that was returned to the user, not a shortened placeholder
- keep the content in Markdown
- ensure the saved file stays aligned with the final answer shown in the conversation

If both `Read.md` and another requested output file are produced, keep their main note content consistent.

## Instruction Priority

Apply instructions in this order:

1. Follow the user's explicit request.
2. Follow the paper itself and the material the user provided.
3. Follow the active classification `agent.md`.
4. Follow this skill.

If two instruction sources conflict, follow the higher-priority source and briefly state the adaptation when needed.

## Classification Folder Discovery

When the user provides a local paper path or a directory that contains the paper:

1. Resolve the path to an absolute location.
2. If the path points to a file, start from its parent directory.
3. If the path points to a directory, start from that directory.
4. Walk upward toward the project root or filesystem root.
5. Find the nearest ancestor directory that contains `agent.md`.
6. Read that `agent.md` before analyzing the paper.
7. Treat the directory that owns that `agent.md` as the active classification folder.

If no `agent.md` is found, continue with this skill and explicitly note:

`未找到分类目录 agent.md，以下分析仅依据论文内容与默认 skill 规则。`

If multiple ancestor directories contain `agent.md`, use the nearest one unless the user explicitly asks for another scope.

If the user gives both a paper path and an explicit classification name, prefer the user's explicit classification when the two are consistent. If they conflict, point out the mismatch and then follow the user's explicit instruction.

## How To Use The Active `agent.md`

Read the active `agent.md` as a category-specific supplement, not as a replacement for the paper.

Use it to adapt:

- domain emphasis
- preferred terminology translations
- comparison baselines that deserve extra attention
- expected output focus such as methods, experiments, or deployment concerns
- category-specific cautions and evaluation lenses

Never let `agent.md` override the facts of the paper itself.

If `agent.md` requests a detail that the paper does not provide, say that the information is not clearly shown instead of guessing.

## Maintain The Classification Terminology Map

When an active classification `agent.md` exists and is a writable local file, treat its `默认术语映射` section as a living glossary for that category.

Update the terminology map during or after the reading task only when a term satisfies all of the following:

- it appears repeatedly in the current paper, or repeatedly across the current user-provided batch of material
- it is central to understanding the category rather than a one-off local detail
- it is likely to appear again in future papers of the same classification
- it is not already covered in the map under the same term, acronym, full name, or a close synonym

Do not add:

- paper-specific method names, one-off module nicknames, ablation labels, temporary variable names, or equation symbols
- dataset split names, hyperparameter names, or narrow experimental settings that are unlikely to be reused
- terms whose meaning, translation, or scope is still ambiguous from the current material

When adding a term:

- update the nearest section whose heading starts with `## 默认术语映射`
- if no such section exists, create one near the end of the file before `## 避免事项`, or append it near the end if needed
- preserve the file's existing style
- use a reusable mapping format such as `- English term: 中文译法`
- for stable acronyms, prefer the canonical acronym as the key, such as `- VLA: 视觉-语言-动作`
- if the concept already exists under another spelling, update or normalize the existing entry instead of creating duplicates
- keep the translation concise, domain-stable, and helpful for future beginner-facing explanations

If the active `agent.md` is not locally writable in the current environment, skip the file edit and continue the reading task normally.

At the end of the task:

- if new terms were added, briefly mention which terms were written into the classification `agent.md`
- if no term clearly qualifies, do not force an update

## Source Intake Rules

Read the whole paper before writing the final note whenever the full paper is available.

If the user pastes the paper in multiple fragments:

- read all fragments first
- wait until the user signals that the paste is complete, or clearly check whether more fragments are coming
- do not produce a fragment-by-fragment analysis unless the user explicitly asks for that mode

If the provided material is incomplete, distinguish between:

- `论文未明确说明` when the paper itself appears to omit the information
- `当前提供的材料中未显示` when the user only shared part of the paper and the missing information may exist elsewhere

## Evidence Checklist

Extract and verify at least the following when available:

- paper title
- authors
- affiliations
- venue, journal, or arXiv status
- research field
- paper type such as method innovation, application study, theoretical analysis, benchmark, or survey
- problem statement
- motivation
- core method
- innovation points
- key figures or framework diagrams
- important formulas, derivations, objectives, or algorithms
- datasets or benchmarks
- main experimental findings
- final conclusions
- limitations
- future work

If an item is unavailable, say so explicitly instead of filling the gap with prior knowledge.

## Required Output Format

Always output a complete Markdown note in Chinese.

Use this structure:

- one H1 title: `# <论文标题>精读笔记`
- six H2 sections in the exact order below

## Section 1: 论文核心内容总结

Write one coherent opening paragraph that covers:

- research background
- the core problem
- the proposed method
- the main results

Then make sure this section also states:

- the paper's field
- the paper type
- the overall logical storyline of the paper

Explain the storyline clearly enough that a researcher from a nearby field can roughly follow the paper's argument.

## Section 2: 创新点提炼

List the core innovation points one by one.

Provide at least two items when the paper genuinely contains two or more distinct innovations.

For each innovation point:

- start with one sentence that captures the idea
- expand with two to three sentences that explain why it matters
- state what weakness in prior methods it addresses

If the innovation points have a progressive relationship, make that relationship explicit.

If the paper is mostly incremental and does not clearly contain multiple strong innovations, say so honestly instead of artificially splitting one idea into many.

## Section 3: 结论摘录与解读

Restate the paper's final conclusions in complete Chinese prose.

Do not simply repeat the abstract.

Explain what experiments, datasets, evidence, or reasoning support those conclusions.

If the paper discusses limitations, failure cases, or future work, include them here as part of the interpretation.

Clearly separate:

- what the paper claims
- what evidence the paper uses to justify the claim
- what caution the reader should keep in mind

## Section 4: 创新方法的详细解释

Treat this as the centerpiece of the note and spend the most space here.

First, give the paper's main new method a clear Chinese alias. Keep the original English name or abbreviation when useful.

Then explain the method in this order:

1. `输入/准备`
Describe the method inputs, the data representation, special preprocessing, assumptions, and any required preconditions.

2. `核心步骤`
Explain every important module, stage, or algorithmic step in the order the data or logic flows through the method.

3. `输出与效果`
Explain what the method outputs and how that output connects to the paper's final task goal.

When formulas appear:

- explain what each important symbol means
- explain what the formula is trying to achieve
- explain how the formula affects the behavior of the method
- if the paper gives a derivation, approximation, reformulation, or transition from one objective to another, explain that theoretical progression step by step instead of only stating the final formula
- make clear which parts come directly from the paper's derivation and which parts are your explanatory intuition
- if there are too many equations to cover all of them, prioritize the equations that define the problem, the training objective, the inference update, the constraint term, the evaluation metric, and any equation the paper uses to justify a design choice
- when a derivation depends on assumptions, monotonicity, independence, decomposition, relaxation, or equivalence, explicitly say so
- if the paper omits intermediate derivation steps, say `论文未展开中间推导，这里仅根据论文已给出的等式关系解释其逻辑`

When algorithms or procedures appear:

- explain the key execution flow
- identify the purpose of each stage
- explain why this design helps
- if the paper includes an algorithm block, pseudocode, or numbered procedure such as `Algorithm 1`, explain it line by line or stage by stage
- state the inputs, outputs, loop variables, branching conditions, update rules, stopping criteria, and how the procedure connects back to the formulas in the paper
- distinguish clearly between training-time procedures and inference-time procedures when both exist

Explicitly compare the method against the main baselines or prior paradigms.

State where the real breakthrough happens and what bottleneck it removes or relaxes.

Use bullet points, analogies, or tiny examples when they reduce cognitive load, but never at the cost of accuracy.

## Section 5: 论文创新结构/流程图详细解读

If the paper contains a framework diagram, model architecture, pipeline figure, algorithm flowchart, or any figure that directly expresses the innovation, analyze it carefully.

Start by stating the figure number, such as `Figure 1`, and explain its overall purpose.

Then walk through the figure step by step:

- explain what each module or component represents
- explain what each arrow or connection means
- explain special legends, colors, dashed lines, or grouped regions when they matter
- explain the data flow or logical order

Conclude this section by explaining how the figure makes the innovation easier to understand and what key ideas a reader should extract from it first.

If multiple figures are relevant, prioritize the one that best captures the paper's central innovation and then mention supporting figures only when helpful.

If the paper truly does not provide such a figure, write:

`本文未给出创新结构示意图。`

Then provide a short textual abstract flow.

If the full paper may contain a figure but the user did not provide that part, write:

`当前提供的材料中未看到创新结构示意图，以下仅根据正文描述梳理抽象流程。`

Then provide the same kind of textual abstract flow.

## Section 6: 自由补充（可选）

Always keep this section in the final note.

Use it for details that are especially worth the reader's attention, such as:

- clever loss design
- unusual data construction
- informative ablation findings
- engineering trade-offs
- possible inspiration for future work
- critical reflections on hidden assumptions or weaknesses

If there is no meaningful extra point beyond the first five sections, say that no additional high-value detail is clearly visible in the current material rather than forcing filler content.

## Writing Constraints

Follow these rules in every run:

- write fully in Chinese
- keep the tone academic, fluent, and readable
- keep terminology precise
- define important terms on first use with the Chinese-plus-English format
- base every claim on the paper or the user-provided material
- do not hallucinate missing methods, datasets, figures, or conclusions
- mention uncertainty explicitly when the source is incomplete
- do not skip central formulas, derivations, or algorithm blocks when they are important for understanding the method
- for papers with substantial mathematical content, ensure the final note is detailed enough that a reader can follow not only `what the formula is` but also `why the paper writes it this way`

For complex concepts, prefer this pattern:

1. first give the big-picture intuition
2. then unpack the components
3. finally summarize what the reader should remember

When the user asks about only one section, still read the whole available material first whenever possible so the answer remains globally consistent.
