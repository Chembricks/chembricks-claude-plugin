---
name: chemistry-guide
description: >
  Use when the user works with molecules, SMILES, chemical structures,
  molecular properties, pKa, logP, solubility, retrosynthesis, conformers,
  xTB, CREST, g-xTB, spectroscopy, IR spectra, NMR, Bayesian optimization,
  steric descriptors, solvation energy, druglikeness, RDKit descriptors,
  metal docking, screening, or any computational chemistry task.
  Guides tool selection across Chembricks tools.
---

# Chembricks AI — Chemistry Tools Guide

Computational chemistry tools across 21 categories, accessed via the
`chembricks` MCP server. Most tools accept a SMILES string as input.

The server advertises the full tool list via `tools/list` — use it to
discover newly added tools not yet covered here.

## Critical Rules

1. **Never do arithmetic manually.** Use `calculate` for expressions,
   `convert_units` for unit conversion, and `reaction_energy` for
   reaction energies. `reaction_energy` validates atom and charge balance,
   catching wrong structures before they corrupt results.

2. **Validate SMILES first.** Run `smiles_validate` before passing SMILES
   to expensive tools. For batch input, use `batch_smiles_parse`.

3. **Energies are not interchangeable.** Different tools use different
   reference states, units, and levels of theory. Never subtract energies
   from different tools — use `reaction_energy` to combine them safely.

4. **Preserve provenance.** Report which tool produced each number, the
   units, and the level of theory. Treat all predictions as estimates.

5. **Long-running tools return job IDs.** Some heavy tools return
   `status='queued'` with a `job_id`. Poll with `job_result` or
   `job_status`. Results are retained for 30 days.

## Quick Tool Selection

| Task | Tool | Notes |
|------|------|-------|
| Validate SMILES | `smiles_validate` | Always do this first |
| Name → SMILES | `name_to_smiles` | IUPAC or common name |
| SMILES → common name | `smiles_to_name` | PubChem lookup |
| SMILES → 3D geometry | `smiles_to_3d` | `theory='gxtb'` (default, server-side) or `'mmff'` (local) |
| Basic molecular properties | `smiles_to_properties` | MW, LogP, TPSA, HBD/HBA, formula |
| Full RDKit descriptors | `compute_all_rdkit_descriptors` | ~200 2D descriptors |
| pKa prediction | `predict_pka` | Supports acids, phenols, amines |
| Aqueous solubility (quick) | `predict_solubility` | ESOL model, water only |
| Solubility with uncertainty | `fastsolv_solubility` | Any solvent + temperature |
| LogP | `logp_ensemble` | RDKit Crippen estimate |
| Solvation + logP (ML) | `cmbx_solvation_logp_v2` | Vacuum energy + water/octanol solvation + logP |
| Drug-likeness | `smiles_to_druglikeness` | Lipinski, Veber, Ghose, QED |
| Drug-likeness batch | `filter_druglikeness_batch` | Screen a library |
| IP / EA (fast) | `cmbx_redox` | Gas-phase, near-DFT accuracy |
| Optimized geometry + energy | `cmbx_base_ai_model` | g-xTB server-side |
| Partial charges | `xtb_charges` | GFN2-xTB Mulliken charges |
| Dipole + HOMO/LUMO | `xtb_dipole` | GFN2-xTB single point |
| Conformer ensemble | `crest_conformers` | CREST metadynamics |
| Thermochemistry (H, G, S) | `xtb_thermochemistry` | GFN2-xTB + RRHO |
| IR spectrum | `xtb_ir_spectrum_full` | GFN2-xTB numerical Hessian |
| ¹³C NMR spectrum | `nmr_c13_spectrum` | CASCADE-2.0 graph-NN |
| Retrosynthesis plan | `retrosynthesis_plan` | AiZynthFinder MCTS |
| Synthetic feasibility (1–10) | `retrosynthesis_feasibility` | Lower = easier |
| Buried volume | `morfeus_buried_volume` | Steric descriptor for catalysis |
| Sterimol L, B1, B5 | `morfeus_sterimol` | Directional steric parameters |
| Bayesian optimization | `bo_create_experiment` → `bo_suggest_candidates` | Multi-objective supported |
| Similar compounds (PubChem) | `screening_similar` | ~99.7M molecules, ECFP4 Tanimoto |
| Arithmetic | `calculate` | Never do math in text |
| Unit conversion | `convert_units` | CODATA exact factors |
| Reaction energy | `reaction_energy` | Validates atom + charge balance |

## Common Workflows

### Molecule characterization

```
smiles_validate → smiles_to_properties → predict_pka
                                       → logp_ensemble
                                       → smiles_to_druglikeness
```

For deeper analysis, add `cmbx_base_ai_model` (optimized 3D + energy),
`xtb_charges` (partial charges), and `xtb_dipole` (dipole + HOMO/LUMO).

### Retrosynthesis

```
smiles_validate → retrosynthesis_feasibility  (quick: is it synthesizable?)
               → retrosynthesis_plan          (full: ranked routes + precursors)
               → check_stock_availability     (are precursors purchasable?)
```

### Conformer analysis + thermochemistry

```
smiles_validate → crest_conformers      (conformer ensemble with populations)
               → xtb_thermochemistry    (H, G, S at 298 K for lowest conformer)
```

### Spectral elucidation (identify unknown compound)

```
spectral_spec_from_observed   (build target from experimental peaks)
→ spectral_elucidate_retrieve (candidate pool from PubChem by formula)
→ spectral_rank_candidates    (rank by ¹³C + IR match score)
```

### Bayesian optimization over molecules

```
bo_create_experiment   (define objectives + variables)
→ bo_featurize_molecules (SMILES → descriptor vectors)
→ bo_add_observations  (add experimental results)
→ bo_fit_surrogate     (fit GP/XGBoost model)
→ bo_suggest_molecules (pick next molecules to test)
```

### Steric analysis for catalysis

```
show_atom_indices       (find 1-based atom numbering)
→ morfeus_buried_volume (% buried volume around metal)
→ morfeus_cone_angle    (Tolman cone angle)
→ morfeus_sterimol      (L, B1, B5 directional parameters)
```

## Tool Categories

| Category | Key tools |
|----------|-----------|
| simulation | `cmbx_base_ai_model`, `cmbx_redox`, `cmbx_solvation_logp_v2` |
| properties | `predict_pka`, `fastsolv_solubility`, `detect_donor_atoms` |
| representation | `name_to_smiles`, `smiles_to_3d`, `smiles_validate` |
| xtb_crest | `xtb_charges`, `crest_conformers`, `xtb_ir_spectrum_full` |
| solvation | `rmg_solvation_energy`, `logp_ensemble` |
| descriptors | `compute_all_rdkit_descriptors`, `calculate_tpsa` |
| steric | `morfeus_buried_volume`, `morfeus_sterimol`, `morfeus_cone_angle` |
| synthesis | `retrosynthesis_plan`, `check_stock_availability` |
| optimization | `bo_create_experiment`, `bo_suggest_candidates` |
| spectroscopy | `nmr_c13_spectrum`, `spectral_rank_candidates` |
| kinetics | `rmg_estimate_kinetics`, `rmg_generate_reactions` |
| thermodynamics | `rmg_estimate_thermo`, `rmg_group_contributions` |
| data_science | `train_regression_model`, `perform_pca`, `cluster_kmeans` |
| filtering | `filter_druglikeness_batch`, `filter_synthetic_feasibility` |
| safety | `safety_check`, `patent_check` |
| generation | `enumerate_analogs`, `similarity_search` |
| ranking | `rank_multiobjective`, `active_learning_suggest` |
| screening | `screening_similar`, `screening_sample` |
| calculations | `calculate`, `convert_units`, `reaction_energy` |
| polymer | `calculate_polymer_descriptors` |
| jobs | `job_status`, `job_result`, `job_cancel` |

The server may have additional tools not listed here. Use `tools/list`
to see the complete, up-to-date catalog with full parameter schemas.
