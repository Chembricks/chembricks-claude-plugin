# Chembricks AI — Claude Code Plugin

Computational chemistry tools as native AI tool calls, powered by the
[Model Context Protocol](https://modelcontextprotocol.io/).

Predict pKa, solubility, and logP. Plan retrosynthetic routes. Run xTB/CREST
simulations. Optimize molecular libraries with Bayesian optimization. Search
99.7 million PubChem compounds by similarity. All from inside Claude Code.

## Quick Start

### 1. Get a token

Register at [cmbx.ai](https://cmbx.ai) and create an API key in the
self-service panel. Your key starts with `cmbx_live_` and works for both
the prediction API and the MCP server.

### 2. Set the environment variable

```bash
export CHEMBRICKS_TOKEN="cmbx_live_<your-token>"
```

Add this to your `~/.zshrc` or `~/.bashrc` to persist it across sessions.

### 3. Install the plugin

**Option A — from the marketplace:**

```
/plugin marketplace add Chembricks/chembricks-claude-plugin
/plugin install chembricks@chembricks-ai
```

**Option B — from the official community marketplace** (after approval):

```
/plugin install chembricks@claude-community
```

### 4. Verify

Reload plugins and try a tool:

```
/reload-plugins
```

Then ask Claude: *"Predict the pKa and aqueous solubility of aspirin."*

## What You Get

Tools are organized into 21 categories. The server advertises the full,
up-to-date list via `tools/list` — below are the highlights.

| Category | Examples |
|----------|---------|
| Simulation | g-xTB geometry + energy, IP/EA, solvation + logP |
| Properties | pKa, solubility (ESOL + fastsolv), donor atoms |
| Representation | SMILES/name/formula/InChI conversion, 3D geometry |
| xTB / CREST | Charges, conformers, IR spectrum, thermochemistry |
| Solvation | Abraham LSER solvation, logP, 200+ solvents |
| Descriptors | All RDKit 2D descriptors, TPSA, Fsp3, fingerprints |
| Steric | Buried volume, Sterimol, cone angle, bite angle |
| Synthesis | Retrosynthesis planning, feasibility, stock check |
| Optimization | Bayesian optimization (GP/XGBoost), Pareto front |
| Spectroscopy | ¹³C NMR (CASCADE-2.0), IR, spectral elucidation |
| Kinetics | Arrhenius kinetics, reaction families (RMG) |
| Thermodynamics | Formation enthalpy, entropy, Benson groups (RMG) |
| Data Science | PCA, k-means, regression, classification |
| Filtering | Druglikeness, synthetic feasibility, purchasability |
| Safety | Hazard screening, patent check, CAS lookup |
| Generation | Analog enumeration, similarity/substructure search |
| Ranking | Multi-objective Pareto, active learning |
| Screening | ~99.7M PubChem3D similarity search |
| Calculations | Arithmetic, unit conversion, reaction energy |
| Polymer | Repeat-unit descriptors |
| Jobs | Status, result, cancel for long-running tools |

## Example Conversations

**Molecule characterization:**
> *"What are the pKa, logP, and druglikeness of ibuprofen?"*

**Retrosynthesis:**
> *"Plan a retrosynthetic route for oseltamivir and check if the precursors
> are commercially available."*

**Conformer analysis:**
> *"Generate the conformer ensemble of this SMILES and compute
> thermochemistry for the lowest-energy conformer."*

**Steric analysis for catalysis:**
> *"Compute the buried volume and Sterimol parameters for
> triphenylphosphine around the phosphorus atom."*

**Spectral elucidation:**
> *"I have a compound with formula C8H8O3 and ¹³C peaks at 56, 114, 122,
> 130, 148, 153, 191 ppm. Identify it."*

**Bayesian optimization:**
> *"Set up a Bayesian optimization to maximize solubility while minimizing
> synthetic difficulty for these 50 candidate molecules."*

## Authentication

This plugin connects to the Chembricks MCP server at
`https://mcp.cmbx.ai/mcp/` using a Bearer token. Set the
`CHEMBRICKS_TOKEN` environment variable with your `cmbx_live_*` key.

The token is never stored in the plugin — it is read from your environment
at runtime via the `${CHEMBRICKS_TOKEN}` placeholder in `.mcp.json`.

## Links

- **Website:** [cmbx.ai](https://cmbx.ai)
- **MCP Server:** `https://mcp.cmbx.ai/mcp/`
- **Server Health:** [mcp.cmbx.ai/health](https://mcp.cmbx.ai/health)
- **Connection Guide:** [docs/CONNECT.md](https://github.com/Chembricks/chembricks-mcp/blob/main/docs/CONNECT.md)

## License

[MIT](LICENSE)
