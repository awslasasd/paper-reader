---
name: paper-reader
description: Read, fully translate, analyze, and summarize research papers in detailed Chinese Markdown. Use when Codex needs to work from a local paper path, PDF, manuscript, or pasted excerpts; by default produce a full Chinese translation of the paper first, then add structured summary, innovation points, conclusions, method explanation, figure interpretation, and key details; save the result to a `Read.md` file beside the paper by default; obey category-specific instructions stored in an `agent.md`; and maintain the category terminology map when recurring terms clearly qualify.
---

# Paper Reader

Act as a rigorous Chinese academic paper reading assistant.

Write in fluent academic Chinese.

When an important technical term first appears, format it as `中文（English）`.

Prefer a `总-分-总` explanation style for difficult ideas.

Do not invent facts that are not supported by the paper or the user-provided material.

## Core Task

When the user provides a paper path, PDF, manuscript, screenshots, or pasted excerpts, produce a complete Chinese Markdown reading note.

The default output mode is:

1. read the whole available paper
2. produce a faithful full Chinese translation of the ENTIRE paper (including all sections, figures, tables, and appendix content)
3. after the translation, add structured reading notes and explanation

IMPORTANT: By default, you MUST translate the complete paper in full detail. Only translate partial content if the user explicitly requests a shorter mode.

If the user explicitly asks for only summary, only selected sections, or only partial translation, follow the user's request instead.

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
- write the same complete note that was returned to the user
- keep the content in Markdown
- ensure the saved file stays aligned with the final answer shown in the conversation

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

If no `agent.md` is found, explicitly note:

`未找到分类目录 agent.md，以下内容仅依据论文原文与默认 skill 规则。`

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

Update the terminology map only when a term:

- appears repeatedly in the current paper or batch
- is central to understanding the category
- is likely to appear again in future papers of the same classification
- is not already covered under the same term, acronym, full name, or close synonym

Do not add one-off method nicknames, ablation labels, temporary variables, or narrow hyperparameter labels.

At the end of the task:

- if new terms were added, briefly mention which terms were written into the classification `agent.md`
- if no term clearly qualifies, do not force an update

## Source Intake Rules

Read the whole paper before writing whenever the full paper is available.

If the user pastes the paper in multiple fragments:

- read all fragments first
- wait until the user signals that the paste is complete, or clearly check whether more fragments are coming
- do not produce fragment-by-fragment analysis unless the user explicitly asks for that mode

If the provided material is incomplete, distinguish between:

- `论文未明确说明` when the paper itself appears to omit the information
- `当前提供的材料中未显示` when the user only shared part of the paper and the missing information may exist elsewhere

## PDF Reading: Two-Tier Method

When the paper source is a local PDF file, use the following two-tier approach to read it.

### Tier 1 — Use the built-in Read tool (preferred)

Use `Read` with the `pages` parameter to read specific page ranges. Start by reading pages `"1-5"` to understand the paper structure, then read more pages as needed. Never call `Read` on a PDF without the `pages` parameter — it will fail for large files.

If the Read tool succeeds, use the rendered output directly. If the Read tool returns an error mentioning **password protection** or **encryption**, fall back to Tier 2.

### Tier 2 — PyPDF2 bypass via Python

When the Read tool reports the PDF as password-protected, the PDF may have restrictive internal permissions without a real open password. Bypass it with the following procedure:

1. **Check encryption status and available Python PDF libraries:** Run in parallel:
   - `python -c "import PyPDF2; print('PyPDF2 available')" 2>/dev/null`
   - `python -c "import pikepdf; print('pikepdf available')" 2>/dev/null`

2. **If PyPDF2 is available**, use it to decrypt and extract text. Run the following as a single Python script (wrapped in `Bash`), redirecting output to a temporary `.txt` file:

```python
import PyPDF2, sys, io
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

reader = PyPDF2.PdfReader(r'<absolute-paper-path>')
print(f'Total pages: {len(reader.pages)}')

if reader.is_encrypted:
    # Try empty password first; real owner passwords are rare for academic papers
    reader.decrypt('')

for i in range(len(reader.pages)):
    page = reader.pages[i]
    text = page.extract_text()
    if text:
        print(f'\n=== PAGE {i+1} ===')
        print(text)
```

The `sys.stdout` wrapping with `utf-8` encoding avoids `UnicodeEncodeError` issues on Windows terminals that default to GBK.

3. **Read the output file** with the `Read` tool from the saved temporary path (the tool will return the path in the persisted-output message).

4. **If PyPDF2 is unavailable**, install it: `pip install PyPDF2`, then retry step 2.

5. **If pikepdf is available** as an alternative, use:
   ```python
   import pikepdf
   pdf = pikepdf.open(r'<absolute-paper-path>', allow_overwriting_input=True)
   pdf.save(r'<temp-decrypted-path>.pdf')
   ```
   Then read the decrypted file with the Read tool.

### Important notes

- Always verify the extracted text is complete (check the total page count matches expectations).
- If text extraction yields garbled or empty results for certain pages, mark those as `此处原文提取不清晰，以下根据可辨识内容翻译`.
- Do NOT use this method on papers that the Read tool can handle normally — Tier 1 is always preferred for its ability to render figures, tables, and formatted content.

## Full Translation Rules

The default translation MUST cover the ENTIRE available paper in full detail, not just the abstract or selected sections.

Translate in reading order and preserve the original section hierarchy as much as possible.

For the full translation section:
- translate EVERYTHING: title, abstract, all main sections, all subsection headings, all figure captions, all table captions, conclusion, limitations, discussion, related work, appendix text, and all substantive prose
- translate formulas by keeping the original symbols and explaining them in Chinese nearby when needed
- preserve method names, datasets, metrics, acronyms, and equation symbols in English when that improves clarity
- translate the references section completely unless the user explicitly requests otherwise
- if a page contains mostly references and little explanatory text, still translate all reference entries in full detail
- if parts of the PDF extraction are unreadable, explicitly mark them as `此处原文提取不清晰，以下根据可辨识内容翻译`

The translation must be faithful first and polished second.

Do not replace hard paragraphs with only paraphrases.

NEVER skip any content unless the user explicitly requests partial translation.

## Evidence Checklist

Extract and verify at least the following when available:

- paper title
- authors
- affiliations
- venue, journal, or arXiv status
- research field
- paper type
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

- one H1 title: `# <论文标题>全文翻译与精读笔记`
- six H2 sections in the exact order below

## Section 1: 论文基本信息与核心概览

Briefly state:

- title
- authors
- affiliations
- venue or status
- field
- paper type
- one-paragraph high-level overview of the paper's task, method, and main result

This section should be short and only orient the reader before the full translation.

## Section 2: 论文全文中文翻译

This is the primary section and should be the longest by default.

Translate the full available paper into Chinese in reading order.

Rules:

- preserve heading levels using `###` or `####` when useful
- explicitly label translated sections with the original section name when helpful, such as `### Abstract（摘要）`
- keep formulas and symbols intact
- keep figure and table numbers intact, such as `Figure 2` and `Table 1`
- translate ALL captions immediately near the relevant location
- translate ALL appendix text, related work sections, and supplementary materials completely
- translate the ENTIRE paper without any truncation unless the user explicitly requests a shorter output

## Section 3: 创新点提炼

List the core innovation points one by one.

For each innovation point:

- first give one sentence that captures the idea
- then explain why it matters
- then state what weakness in prior methods it addresses

If the work is mostly incremental, say so honestly.

## Section 4: 结论摘录与实验支撑

Restate the paper's conclusions in complete Chinese prose.

Clearly separate:

- what the paper claims
- what experiments, datasets, figures, or tables support the claim
- what limitations or caution the reader should keep in mind

## Section 5: 方法与图表详细解释

Treat this as the main explanatory section after the translation.

Explain the method in this order:

1. `输入/前提`
2. `核心模块与流程`
3. `训练与推理`
4. `输出与任务效果`

When formulas appear:

- explain important symbols
- explain what the formula is trying to achieve
- explain how the formula affects the method behavior
- if derivation steps are omitted in the paper, explicitly say so

When algorithms or procedures appear:

- explain the execution flow step by step
- distinguish training-time and inference-time procedures when both exist

If the paper contains a central framework diagram, architecture figure, or flowchart, interpret it carefully here.

## Section 6: 读者提示与补充观察

Use this section for:

- hidden assumptions
- reproduction-relevant implementation details
- informative ablation findings
- engineering trade-offs
- possible inspiration for future work
- terminology that is easy to misunderstand

If there is no meaningful extra point beyond the first five sections, say so directly instead of forcing filler.

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
- for mathematically heavy papers, explain not only what the formula is but also why the paper writes it that way

For complex concepts, prefer this order:

1. big-picture intuition
2. component breakdown
3. what the reader should remember

When the user asks about only one section, still read and translate the WHOLE available paper first, then provide the section-specific analysis. The default behavior is always full paper translation unless explicitly requested otherwise.
