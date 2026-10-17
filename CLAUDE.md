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

On this machine latexmk's BibTeX step runs inside `build/` and fails with "I couldn't open database file refs.bib", which leaves every citation undefined (setting `BIBINPUTS` did not help). In that case use the manual sequence above, running `bibtex build/main` from the repository root.

## Structure of main.tex

- **Preamble macros.** Use these rather than writing the expansions inline: `\Hst`, `\Est` and `\Mst` for the hypothesis history, evidence log and memory (formerly H-, E- and M-store; do not reintroduce those names; the memory belongs to one chain and is not "shared memory"); `\opp` for the opponent pool; `\reg` for the metric registry; `\gbar`, `\KL` and `\E`; `\pUp`, `\pFlat` and `\pDown` for prediction directions; `\code{}` for schema and field names, with underscores escaped as `\_`; and `\todo{}`, which renders in red, for open items.
- **Notation.** Calligraphic letters are stores, and a subscript on a store is its state at a round ($\Mst_t$). A superscript $(t)$ on a record is its round: $H^{(t)}$, $e^{(t)}$, $\mu^{(t)}$, and ranges are written $H^{(0:t)}$. Evidence $e^{(t)}$ comes from training $H^{(t-1)}$. The Coach is $\Gamma$, decoded at temperature 0; its fourth argument $\nu$ is the violation feedback from a rejected revision. The loop thresholds are $k_{\mathrm{conv}}$ and $k_{\mathrm{ret}}$, the attempts per Coach step $n_{\mathrm{att}}$, and the round budget of a campaign $B$. Benefit is $\mathrm{Pts}$ (match points), not win rate, and $\bar o^{\mathrm{exe}}$, $\bar o^{\mathrm{med}}$ are gate-weighted means. Every delta is taken against the shared λ = 0 control runs $\pi_{\mathrm{ctrl}}$, not against $\pi_0$ ($\mathrm{KL}_0^{\mathrm{ctrl}}$ is their mean KL); $\opp'$ is the held-out opponent set used only in the confirmation of converged rules (`sec:confirm`). Avoid $\mathcal{C}$, which clashes with the condition $C$, and bare $m$, which is reserved for metrics.
- **Cross-references.** Use cleveref (`\cref`/`\Cref`) with label prefixes `sec:`, `tab:`, `fig:`, `alg:`, `eq:` and `app:`. Method subsections are referenced by label throughout, for example `sec:compile`, `sec:feedback` and `sec:revision`. Keep labels stable when you edit.
- **Section order.** Introduction → Related Work → Problem Formulation → Method → Iteration Efficiency → Evaluation Protocol → Discussion → Conclusion. Each Method subsection opens with a prose sentence stating what the step reads and what it produces. Do not use bold "Input:/Output:" labels, which read like engineering documentation rather than a paper. The Evaluation Protocol has a `Configuration` subsection (`sec:config`) for settings fixed before a campaign, and the ablations subsection is `sec:ablations`. The Method section uses a TikZ loop figure, several `tabularx` tables (fields, diagnosis rules, operators, seeds, ablations) and an `algorithm2e` algorithm.
- **Back matter.** `\clearpage` before the bibliography. The appendix starts on a new page (`\clearpage\appendix`) with the section `app:schemas`, which holds the JSON schemas in `lstlisting` with the `json` style.
- **Scope and status.** The paragraph of that name in the introduction lists the open items of the draft (no results on hypothesis refinement yet; the shaping ratio $\rho$ and the opponent pool still open). When you close a `\todo` or replace an estimate with a measurement, update that paragraph too.
- **Consistency across sections.** Several concepts appear in more than one place: the seven revision operators, the diagnosis labels and the three quantities (execution, mediator, benefit). The operator list, for example, appears in `tab:ops`, the `RevisionOp` schema in the appendix, and the metrics text. When you rename or add one, update every occurrence. Diagnosis labels use the implementation's names (`belief_disconfirmed_value`, `belief_disconfirmed_mechanism`, and so on) in `tab:diagnosis` and the appendix alike.

## Companion implementation

The method is implemented in a separate repository, https://github.com/ZhBF/marladona-11v11 (MARLadona at 11v11 on Isaac Lab, plus the Coach loop in its `coach/` package, `marl_coach`). It is not checked out here. To compare the paper with it, clone it into a scratch directory. For settled decisions, the implementation is the source of truth (see "What to sync" below):

- Its `CLAUDE.md` is the living status record: decisions, calibrations (π0, registry, $\bar R_0$, $\rho$, $\beta_{\mathrm{KS}}$), the results of each phase (P0–P5), and the measured cost of a round.
- `coach/marl_coach/core/schemas.py` defines what the Coach writes (`HypothesisEntry`, `RevisionOp`, `MemoryEntry`, evidence fields); the appendix schemas should follow it.
- `core/attribution.py` (the three deltas, the bootstrap that resamples seeds and then episodes, the diagnosis table), `core/refchecks.py` (reference checks), `core/chain.py` (convergence and retirement), `core/loop.py` (the order of steps in Algorithm 1) and `sim/objective.py` (the gate and the potential) correspond to the Method subsections.
- `coach/specs/env_spec_v1.json` is the frozen `env_spec`.

### What to sync

The implementation is still a minimal loop under active change: one seed hypothesis (H-001), one chain, a provisional π0, and an adaptation budget and thresholds that are still being revised. **Never align the paper with code that is clearly in an intermediate state.** The paper describes the full design.

- Sync the paper with settled design decisions: methods, definitions, data formats, and rules the user has decided. Examples already synced (October 2026): benefit as match points at a 300 s cutoff, the unpaired bootstrap over seeds and then episodes, gate-weighted execution and mediator, replicates on fresh seeds instead of cache hits, PPO with a shared policy rather than MAPPO, the attempt cap and round budget in Algorithm 1, the full list of reference checks, and the appendix schemas.
- Do not sync minimal-loop gaps or provisional values: which seeds exist, the current π0 and its training details, the budget (seeds, iterations, environments), $\beta_{\mathrm{KS}}$, the $\epsilon$ thresholds, calibrated numbers such as $\bar R_0$, or readout workarounds. Do not "downgrade" the paper to describe what the minimal loop lacks; a missing component is a task for the implementation, not for the paper.
- When it is unclear whether something is settled, ask the user before editing.
- Settled by the user but not yet in the code (the paper leads here): the "A+" opponent design (October 2026). Training and evaluation use one fixed pool $\opp$ of π0 plus compiled trait variants (each passing a trait check and a strength band), mixed in fixed proportions shared by all hypotheses, with opponents sampling their actions during training and never learning; no self-play. In evaluation both teams take the mean of their action distributions. Deltas are against shared λ = 0 control runs; results are also reported per opponent, and conditions may not name an opponent. Converged rules are confirmed on held-out opponents $\opp'$ and against opponents that adapt within a limited budget; confirmation is hidden from the Coach and only grades the terminal state. Keep the paper consistent with this design when editing related sections.
- Chains do not interact (user decision, October 2026; follow the code here, not the earlier paper design). Each chain has its own three stores and shares only the frozen artifacts (`env_spec`, registry, π0, opponent pools, control runs). The memory carries lessons across versions of one chain; there is no promotion of findings across chains, no chain-index covariate, and no threshold for memory entries beyond the check that cited evidence belongs to the chain.

Before you describe a mechanism as implemented or a number as measured, check it against the implementation. The item-by-item audit of the paper against the implementation, with what is done, deferred or awaiting a decision, is at https://claude.ai/artifact/74iBxWRvemgwyJhXxD2rj4.

## Writing conventions

- Use American spelling, for example behavioral, parameterized, normalize, artifact and judgment. The cleveref option `capitalise` in the preamble is the package's own option name, so do not change it.
- Avoid hyphenated compounds unless the hyphen is required or standard academic usage, for example multi-agent, fine-tuning, on-policy, zero-shot, self-play, or a soccer term such as counter-press. Prefer prepositional phrases, for example "rollouts of the base policy" rather than "base-policy rollouts".
- Write "soccer", not "football", except in proper names such as Google Research Football.
- Avoid em dashes (`---`) unless truly necessary; use commas, parentheses, colons or separate sentences instead. Use `--` for ranges.
