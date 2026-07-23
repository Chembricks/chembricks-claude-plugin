---
name: chembricks-computational-chemistry
description: >-
  Use the live Chembricks MCP for computational chemistry and molecular
  science: resolve or validate molecular structures; calculate molecular
  properties, descriptors, pKa, logP, solubility, redox, energies,
  thermochemistry, spectra, conformers, steric descriptors, kinetics, and
  solvation; search or generate molecules; assess synthesis, stock, stability,
  purchasability, and narrow safety alerts; investigate metal complexes and
  polymers; or run multi-objective molecular discovery, screening, and
  optimization. Apply this skill whenever a user supplies a molecule, name,
  formula, SMILES, InChI, SELFIES, spectrum, reaction, ligand, candidate
  library, or chemistry design objective that can benefit from Chembricks
  evidence.
---

# Chembricks Computational Chemistry

Use Chembricks as the source of scientific computations and retrieved chemistry data. Use Claude's native capabilities for reasoning, arithmetic, unit conversion, task tracking, memory, and code or harness generation.

## Non-negotiable operating contract

1. Treat the tools and schemas currently exposed by `https://mcp.cmbx.ai/mcp/` as authoritative. Do not rely on a remembered tool count, parameter, default, or capability when the live schema is available.
2. Extract every requested deliverable, strategy, constraint, count, comparison, and stopping condition before calling tools. Complete all explicitly requested branches rather than a convenient subset.
3. Choose the least expensive scientifically adequate method. Start with identity and fast descriptors; reserve geometry optimization, Hessians, CREST, full retrosynthesis, and metal-complex jobs for questions or finalists that need them.
4. Call independent tools in parallel and use batch inputs when the live schema supports them and every item shares the same conditions. Never batch incompatible methods, solvents, temperatures, charges, or spin states.
5. Preserve molecular identity and provenance. Associate every reported value with the molecule, tool, method/model, dataset, phase or solvent, conditions, and unit that produced it.
6. Treat all predictions as estimates. Distinguish computed, ML-predicted, database-derived, and heuristic results, and state material limitations.
7. Never silently drop a failed molecule or calculation. Repair validly repairable inputs, retry transient failures, or replace the candidate. Explain any genuine blocker.
8. If the user requests N examples or successful candidates, return N unique successful candidates. Invalid, duplicate, failed, interrupted, or still-queued entries do not count.
9. Re-show molecular identities throughout long work. Use names plus canonical SMILES and include structures when the client supports them, especially in comparisons and design iterations.
10. Before concluding, audit the answer against the original request and identify any remaining unsupported or incomplete item.

## Keep generic work native to Claude

- Do not call the Chembricks `calculate` or `convert_units` tools. Perform ordinary arithmetic and unit conversion with Claude's native capabilities, showing the formula and source values when the result matters.
- Do not seek MCP tools for code execution, code generation, memory, planning, task progress, progress reports, or report harnesses.
- Do not use generic statistical or data-science tools merely to reproduce analysis Claude can already perform. Use chemistry-domain Bayesian optimization, active learning, or Pareto ranking when the scientific workflow itself requires those methods.
- Use `reaction_energy` when combining per-species energies for a chemical reaction. Its purpose here is chemical validation: atom balance, formal-charge balance, stoichiometry, compatible energy provenance, and an auditable species ledger.

## Resolve identity before expensive work

1. Convert names with `name_to_smiles`; convert InChI or SELFIES with the corresponding representation tool.
2. Use `smiles_validate` for uncertain, user-entered, generated, or transformed structures. Use `batch_smiles_parse` for a batch. Do not add redundant validation calls for an already canonical, tool-returned structure unless its identity is in doubt.
3. Canonicalize before deduplication with `smiles_canonicalize`.
4. Treat stereochemistry, tautomer, protonation state, salt form, isotope, formal charge, and spin as scientific choices. Ask only when the choice materially changes the requested result and cannot be inferred.
5. Use `show_atom_indices` before any atom-indexed steric, coordination, or constrained calculation.
6. Do not infer that similarly named or visually similar structures are the same molecule. Verify the actual representation.

## Select the smallest useful tool chain

| Scientific need | Start with | Escalate when needed |
| --- | --- | --- |
| Identity or representation | `name_to_smiles`, `smiles_validate`, `smiles_canonicalize`, `smiles_to_formula` | `smiles_to_name`, `mol_to_cas`, `formula_to_smiles_list` |
| Compact 2D properties | `smiles_to_properties` | `compute_rdkit_descriptors_subset`; use all descriptors only when justified |
| Drug-likeness or library filters | `smiles_to_druglikeness`, `filter_druglikeness_batch` | `filter_stability`, `filter_elements`, `filter_synthetic_feasibility` |
| Acidity, lipophilicity, solubility | `predict_pka`, `logp_ensemble`, `predict_solubility` | `cmbx_solvation_logp_v2`, `fastsolv_solubility`, or a temperature curve |
| Fast redox | `cmbx_redox` | `compute_ip_ea` for the async xTB-coordinate route; `redox_potential` for a solution potential |
| Geometry or electronic structure | `smiles_to_3d`, `cmbx_base_ai_model` | `gxtb_properties` or the relevant `xtb_*` property |
| Conformers | `smiles_to_3d` for one geometry | `crest_conformers` or the specific CREST workflow |
| IR or carbon-13 NMR | `xtb_ir_spectrum_full`, `nmr_c13_spectrum` | spectral retrieval, scoring, ranking, or elucidation tools |
| Similar molecules | `screening_similar` | `enumerate_analogs`, `mutate_molecule`, `substructure_search`, then filters |
| Synthesis | `retrosynthesis_feasibility` or `retrosynthesis_single_step` | `retrosynthesis_plan`, then `check_stock_availability` |
| Sterics | `show_atom_indices` | the required `morfeus_*` descriptor |
| Metal complexes | binding-site and geometry helpers | starting structures, xTB/g-xTB calculations, then async stability or series jobs |
| Thermodynamics or kinetics | the relevant `rmg_*` lookup or estimate | xTB thermochemistry for structure-specific RRHO quantities |
| Molecular optimization | fast descriptors and filters | `bo_*`, `active_learning_suggest`, or `rank_multiobjective` |
| Polymers | `calculate_polymer_descriptors` | explain that repeat-unit descriptors are not bulk experimental properties |

Do not call every tool in a row. Add a tool only if its result can change the answer, ranking, next experiment, or confidence.

## Handle energies and thermochemistry rigorously

- Compare energies only when they share the same tool, model, dataset, phase or solvation model, charge/spin convention, and compatible geometry protocol.
- Treat an absolute electronic energy as an intermediate, not a standalone measure of stability across unrelated molecules.
- For `cmbx_internal_energy_v2`, specify the dataset on every related call:
  - `heavy-metal` is the legacy wB97X plus SMD-water endpoint.
  - `organic` is a wB97M-V vacuum endpoint with ECP/valence-only total energies and atomization energies.
  - Never mix the two datasets or compare `organic` total energies with all-electron references.
- For a reaction, obtain every species energy from the same method and conditions, then call `reaction_energy`. Give each species a unique `name`; use that name for `per_species` normalization. Represent molecular hydrogen as `[H][H]`.
- Keep balance checking enabled unless the user explicitly requests a deliberately lumped model such as a surface-site cycle. State when balance checking is skipped.
- Do not use `reaction_energy` for an unbalanced redox half-reaction containing implicit electrons.
- Treat `cmbx_redox` as gas-phase IP/EA, not a potential versus a reference electrode. Use `redox_potential` when the user asks for a solution electrochemical potential.
- Treat `xtb_thermochemistry` outputs as GFN2-xTB RRHO molecular quantities at the stated temperature and pressure, not standard formation enthalpies. Check imaginary modes before interpreting a stationary point. Use RMG formation-property tools for group-additivity estimates.

## Apply model-specific limits

- `cmbx_base_ai_model` uses g-xTB; describe it as semiempirical rather than DFT.
- `cmbx_catdis` is a coarse risk flag for large DFT-versus-QMC total-energy disagreement, not a calibrated uncertainty on the user's target property.
- `predict_pka` supports limited ionizable classes, refuses unsupported classes, and can miss substituent trends. Use it as a screening signal rather than a regulatory or experimental value.
- Use `predict_solubility` for rough aqueous ranking. Use `fastsolv_solubility` only when solvent or temperature specificity and its uncertainty are useful; do not use FastSolv to rank chemically diverse solutes, and warn that absolute values can be strongly optimistic.
- Treat `logp_ensemble` as a descriptor estimate. Do not equate logP with pH-dependent logD.
- `nmr_c13_spectrum` predicts carbon-13 shifts only. Do not claim proton NMR, multiplicities, or coupling constants.
- `xtb_ir_spectrum_full` is a GFN2-xTB double-harmonic spectrum. Use it for functional-group interpretation; describe frequencies as approximate and intensities as semi-quantitative.
- `safety_check` covers a narrow set of reactive/energetic alerts and crude CWA motifs. A pass is not a toxicity, carcinogenicity, environmental, legal, or handling assessment.
- `filter_purchasability=false` is definitive only for absence from its ZINC20 Bloom-filter source; `true` is probabilistic and requires supplier confirmation. `check_stock_availability` uses a different stock corpus.
- Treat retrosynthesis routes as computational proposals. Check precursor availability, reaction plausibility, selectivity, protecting groups, and hazardous conditions before recommending a route.
- Treat screening hits as database candidates, not validated purchasable or synthesizable compounds.
- For metal-complex work, specify metal identity, oxidation state, total charge, multiplicity, stoichiometry, solvent, geometry assumptions, and method. Do not compare xTB and g-xTB energy scales. Use ensemble Gibbs energies for stability-constant rankings when available.

## Run long or autonomous searches without losing work

1. Define objectives, directions, units, constraints, evaluation method, success count, and stopping condition.
2. Start from chemically diverse seeds, not cosmetic variants of one scaffold.
3. Screen broad candidate sets with cheap descriptors and hard filters.
4. Evaluate survivors with the selected higher-cost model under identical conditions.
5. Keep a compact native-Claude ledger with canonical SMILES, attempted methods, successes, failures, and reasons for rejection. Do not depend on an MCP memory or progress tool.
6. Deduplicate before each batch and do not recompute an identical structure under identical conditions.
7. Replace unsuccessful candidates before advancing to the next design iteration.
8. For multi-objective work, keep the objective definitions fixed and retain nondominated candidates. Use domain BO or Pareto tools when they materially improve candidate selection.
9. Revisit a user-selected or pinned structure as the new seed when asked, while preserving the earlier evidence as a separate branch.
10. Stop only at the declared criterion, a user request, or a concrete external blocker such as exhausted upstream capacity.

If the user asks for strategies A, B, and C, maintain separate ledger entries for all three. If one strategy fails, explain the failure and continue the others rather than silently collapsing the plan.

## Reconcile queued and failed calls

- Preserve every returned job ID and do not resubmit the same expensive job while it remains queued.
- Poll `compute_ip_ea` jobs with `get_ip_ea_result`.
- Poll `metal_dock_*_submit` jobs with `metal_dock_get_result`.
- Poll other generic queued jobs with `job_status` and retrieve completed output with `job_result`.
- Poll at reasonable intervals; avoid a tight loop. A timeout does not prove that scientific work failed.
- On schema or validation errors, inspect the live schema and correct the arguments before retrying.
- On invalid identity, repair only when the intended structure is unambiguous; otherwise ask the user.
- On transient transport, proxy, or retryable interruption errors, retry a bounded number of times and reconcile any existing job first.
- On deterministic scientific failure, record the failure and replace the candidate if a success count remains open.
- Never report `null`, queued, interrupted, or errored output as a completed scientific result.

## Present a decision-ready answer

Lead with the answer, then provide the minimum evidence needed to audit it.

For multiple molecules, prefer a compact table containing:

| Molecule | Canonical SMILES | Result | Unit | Method and conditions | Confidence or caveat |
| --- | --- | --- | --- | --- | --- |

Also include:

- the structures or clear molecular identities used;
- the requested ranking, recommendation, or interpretation;
- provenance for every decision-driving number;
- failures, replacements, and still-missing evidence;
- whether every requested strategy and success count was completed;
- the most important model limitations and the next experiment or calculation that would reduce uncertainty.

Do not expose repetitive internal reconciliation narration. Give concise milestone updates during long work and a coherent scientific report at completion.

## Example routes

- **“Characterize aspirin.”** Resolve identity, obtain compact properties, add pKa/logP/solubility or drug-likeness only if relevant, and escalate to 3D or spectra only when requested.
- **“Give me five analogs with lower logP.”** Canonicalize the seed, retrieve or generate a diverse batch, deduplicate, compute the same logP method for all, replace failures until five successes remain, and report names, structures, SMILES, values, and filters.
- **“Compare a hydrogen-rich and hydrogen-lean pair.”** Verify the balanced structures and hydrogen count, compute every species with one energy model and dataset, use `reaction_energy` with named species, normalize per H2, and report the complete energy lineage.
- **“Predict the IR and carbon-13 NMR of vanillin.”** Resolve vanillin once, run the two independent spectrum tools in parallel, label IR as GFN2-xTB and NMR as ML-predicted carbon-13, and keep their limitations separate.
- **“Optimize candidates using strategies A, B, and C.”** Create three explicit branches, use the same objectives and evaluation methods, maintain a shared deduplication ledger, replace failed candidates, and compare the resulting nondominated sets before recommending a winner.
