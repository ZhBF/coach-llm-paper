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
- **Back matter.** `\clearpage` before the bibliography. The appendix starts on a new page (`\clearpage\appendix`) with the section `app:schemas`, which holds the JSON schemas in `lstlisting` with the `json` style.
- **Scope and status.** The paragraph of that name in the introduction lists the open items of the draft (no results, open design parameters, estimated efficiency figures). When you close a `\todo` or replace an estimate with a measurement, update that paragraph too.
- **Consistency across sections.** Several concepts appear in more than one place: the seven revision operators, the diagnosis labels and the three quantities (execution, mediator, benefit). The operator list, for example, appears in `tab:ops`, the `RevisionOp` schema in the appendix, and the metrics text. When you rename or add one, update every occurrence.

## Companion implementation

The method is implemented in a separate repository, https://github.com/ZhBF/marladona-11v11 (MARLadona at 11v11 on Isaac Lab, plus the Coach loop in its `coach/` package, `marl_coach`). It is not checked out here. To compare the paper with it, clone it into a scratch directory. The implementation is the source of truth for the experimental facts that the paper should report:

- Its `CLAUDE.md` is the living status record: decisions, calibrations (π0, registry, $\bar R_0$, $\rho$, $\beta_{\mathrm{KS}}$), the results of each phase (P0–P5), and the measured cost of a round.
- `coach/marl_coach/core/schemas.py` defines what the Coach writes (`HypothesisEntry`, `RevisionOp`, `MemoryEntry`, evidence fields); the appendix schemas should follow it.
- `core/attribution.py` (the three deltas, the bootstrap that resamples seeds and then episodes, the diagnosis table), `core/refchecks.py` (reference checks), `core/chain.py` (convergence and retirement), `core/loop.py` (the order of steps in Algorithm 1) and `sim/objective.py` (the gate and the potential) correspond to the Method subsections.
- `coach/specs/env_spec_v1.json` is the frozen `env_spec`.

The paper and the implementation have drifted apart in places. For example, the implementation uses points rather than win rate, unpaired seed-level bootstrap, PPO with parameter sharing rather than MAPPO, kickstarting only, and replicates on fresh seeds instead of cache hits. Before you describe a mechanism as implemented or a number as measured, check it against the implementation, and say in the text when something is still planned.

## Writing conventions

- Use American spelling, for example behavioral, parameterized, normalize, artifact and judgment. The cleveref option `capitalise` in the preamble is the package's own option name, so do not change it.
- Avoid hyphenated compounds unless the hyphen is required or standard academic usage, for example multi-agent, fine-tuning, on-policy, zero-shot, self-play, or a soccer term such as counter-press. Prefer prepositional phrases, for example "rollouts of the base policy" rather than "base-policy rollouts".
- Write "soccer", not "football", except in proper names such as Google Research Football.
- Avoid em dashes (`---`) unless truly necessary; use commas, parentheses, colons or separate sentences instead. Use `--` for ranges.
