# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A single LaTeX paper draft: "Coach LLM: Evidence-Grounded Refinement of Tactical Hypotheses for Multi-Agent Reinforcement Learning". The whole paper is in `main.tex`, and its bibliography is in `refs.bib`, using natbib with `plainnat`. There is no code. The paper describes the method and the evaluation protocol, and its results section is still a `\todo`.

## Build

The VS Code LaTeX Workshop configuration (`.vscode/settings.json`) builds on save with latexmk. Output goes to `build/`, which is gitignored. To build from the command line (MiKTeX):

```sh
latexmk -pdf -synctex=1 -interaction=nonstopmode -file-line-error -outdir=build main.tex
```

If latexmk is not available, run `pdflatex` → `bibtex build/main` → `pdflatex` → `pdflatex`, with `-output-directory=build`. Check `build/main.log` for undefined references and citations.

## Structure of main.tex

- **Preamble macros.** Use these rather than writing the expansions inline: `\Hst`, `\Est` and `\Mst` for the hypothesis history, evidence log and shared memory (formerly H-, E- and M-store; do not reintroduce those names); `\opp` for the opponent pool; `\reg` for the metric registry; `\gbar`, `\KL` and `\E`; `\pUp`, `\pFlat` and `\pDown` for prediction directions; `\code{}` for schema and field names, with underscores escaped as `\_`; and `\todo{}`, which renders in red, for open items.
- **Notation.** Calligraphic letters are stores, and a subscript on a store is its state at a round ($\Mst_t$). A superscript $(t)$ on a record is its round: $H^{(t)}$, $e^{(t)}$, $\mu^{(t)}$, and ranges are written $H^{(0:t)}$. Evidence $e^{(t)}$ comes from training $H^{(t-1)}$. The Coach is $\Gamma$, decoded at temperature 0; its fourth argument $\nu$ is the violation feedback from a rejected revision. The loop thresholds are $k_{\mathrm{conv}}$ and $k_{\mathrm{ret}}$. Avoid $\mathcal{C}$, which clashes with the condition $C$, and bare $m$, which is reserved for metrics.
- **Cross-references.** Use cleveref (`\cref`/`\Cref`) with label prefixes `sec:`, `tab:`, `fig:`, `alg:`, `eq:` and `app:`. Method subsections are referenced by label throughout, for example `sec:compile`, `sec:feedback` and `sec:revision`. Keep labels stable when you edit.
- **Section order.** Introduction → Related Work → Problem Formulation → Method → Iteration Efficiency → Evaluation Protocol → Discussion → Conclusion. Each Method subsection opens with a prose sentence stating what the step reads and what it produces. Do not use bold "Input:/Output:" labels, which read like engineering documentation rather than a paper. The Method section uses a TikZ loop figure, several `tabularx` tables (fields, diagnosis rules, operators, seeds, ablations) and an `algorithm2e` algorithm.
- **Back matter.** `\clearpage` before the bibliography. The appendix starts on a new page with a centered "Appendix" heading, followed by JSON schemas in `lstlisting` with the `json` style.
- **Consistency across sections.** Several concepts appear in more than one place: the seven revision operators, the diagnosis labels and the three quantities (execution, mediator, benefit). The operator list, for example, appears in `tab:ops`, the `RevisionOp` schema in the appendix, and the metrics text. When you rename or add one, update every occurrence.

## Writing conventions

- Use American spelling, for example behavioral, parameterized, normalize, artifact and judgment. The cleveref option `capitalise` in the preamble is the package's own option name, so do not change it.
- Avoid hyphenated compounds unless the hyphen is required or standard academic usage, for example multi-agent, fine-tuning, on-policy, zero-shot, self-play, or a soccer term such as counter-press. Prefer prepositional phrases, for example "rollouts of the base policy" rather than "base-policy rollouts".
- Write "soccer", not "football", except in proper names such as Google Research Football.
- Avoid em dashes (`---`) unless truly necessary; use commas, parentheses, colons or separate sentences instead. Use `--` for ranges.
