---
name: research-note-style
description: Use when drafting, compressing, revising, or saving Chinese AI research notes in this project, especially notes about model training, distillation, RL, KL, papers, or technical blogs. Apply the user's preference for concise Thinking Machines / Anthropic-style research prose, formal section titles, centered Markdown LaTeX, low AI flavor, and minimal quote/bullet-heavy explanation.
---

# Research Note Style

## Core Style

Write in concise Chinese research-note prose. Prefer a calm, clean blog-note style: clear enough for future rereading, but not tutorial-like.

Use formal section titles. Avoid titles such as "是什么" unless the user explicitly asks for beginner-oriented explanation.

Reduce AI-flavored scaffolding:

- Avoid excessive bullets, quote blocks, and "直观理解" labels.
- Avoid repeated transition phrases and over-explaining obvious math.
- Keep the final note compact; preserve only distinctions that matter.

## Math And Formatting

Use Markdown LaTeX. Put important formulas in centered display blocks:

```markdown
$$
D_{\mathrm{KL}}(P \| Q)
=
\sum_x P(x)\log\frac{P(x)}{Q(x)}
$$
```

Use inline math only for short symbols, e.g. \(\pi_\theta\), \(\pi_T\), \(s_t\).

Prefer short paragraphs over nested lists. Use tables only when they make comparisons easier to scan.

## Research Note Workflow

When revising user-provided notes:

1. Compress aggressively while preserving the conceptual spine.
2. Keep the user's intended sections if they specify them.
3. Remove chatty explanation, repeated caveats, and broad textbook setup.
4. Preserve subtle distinctions, especially weighting distribution, prefix/state distribution, and objective semantics.
5. If saving locally, use a filename in the form `YYYY-MM-DD-short-topic.md`.

When citing a web article, add a short `参考` section with title, source, URL, and publication date when available.
