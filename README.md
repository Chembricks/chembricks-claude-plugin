# Chembricks AI — Claude Code Plugin

Computational chemistry tools as native AI tool calls, powered by the
[Model Context Protocol](https://modelcontextprotocol.io/).

Predict pKa, solubility, and logP. Plan retrosynthetic routes. Run xTB/CREST
simulations. Optimize molecular libraries with Bayesian optimization. Search
99.7 million PubChem compounds by similarity. All from inside Claude Code,
with a bundled skill that keeps molecular identity, method compatibility,
provenance, failures, and requested result counts explicit.

## Quick Start

### 1. Get a token

Register at [cmbx.ai](https://cmbx.ai) and create an API key in the
self-service panel. Your key starts with `cmbx_live_` and works for both
the prediction API and the MCP server.

### 2. Set the environment variable (Claude Code)

```bash
export CHEMBRICKS_TOKEN="cmbx_live_<your-token>"
```

Add this to your `~/.zshrc` or `~/.bashrc` to persist it across sessions. The
Claude Desktop extension does not need this variable — it asks for the key in
its own settings window and keeps it in your operating system's secure storage.

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

## Install in Claude Desktop

Claude Desktop installs MCP servers as extensions. Download
`chembricks-<version>.mcpb` from the
[latest release](https://github.com/Chembricks/chembricks-claude-plugin/releases/latest)
and open it in any of these ways:

- double-click the file;
- drag it into the Claude Desktop window;
- **Settings > Extensions > Advanced settings > Install Extension…**.

The installation window asks for your `cmbx_live_*` key. Leave the server URL
field as it is unless you were given a different endpoint.

The extension starts a small local process on Claude Desktop's own Node
runtime, so nothing has to be installed first. That process computes nothing:
it forwards MCP messages to `https://mcp.cmbx.ai/mcp/`, which is where the
chemistry runs, and it holds the connection open through long idle periods.

## Where Each Install Works

The plugin reaches the server over HTTPS; the extension starts a local process
that does the same. Which one works depends on the surface:

| | Chat (web, desktop, mobile) | Cowork | Claude Code |
|---|---|---|---|
| Plugin (remote server in `.mcp.json`) | Yes, after you connect it on the plugin's **Connectors** tab | Yes | Yes |
| Extension (`.mcpb`) | No — chat does not run local servers | Yes, when the session runs on your computer | Yes |

Install one of the two. With both, Claude sees the same 150 tools twice and
spends context on the duplicate.

## How the Skill Works

The `chembricks-computational-chemistry` skill treats the live MCP schemas as
the source of truth and chooses the smallest scientifically adequate tool
chain for each question. It:

- keeps ordinary arithmetic, unit conversion, planning, memory, and code
  generation native to Claude;
- retains `reaction_energy` for atom, charge, stoichiometry, and compatible
  energy-provenance validation;
- completes every requested strategy and replaces failed candidates instead
  of silently counting them as successes;
- reconciles queued work with the correct job-family result tool; and
- reports molecular identity, method, conditions, units, limitations, and
  decision-driving provenance.

## What You Get

The live server advertises the complete, up-to-date tool set and schemas.
Below are the major scientific capabilities. Generic calculator and
data-science helpers may be exposed by the server, but the bundled skill does
not route to them when Claude can handle that work natively.

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
| Filtering | Druglikeness, synthetic feasibility, purchasability |
| Safety | Hazard screening, patent check, CAS lookup |
| Generation | Analog enumeration, similarity/substructure search |
| Ranking | Multi-objective Pareto, active learning |
| Screening | ~99.7M PubChem3D similarity search |
| Reaction validation | Atom/charge-balanced reaction energies with provenance |
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

Both installs reach `https://mcp.cmbx.ai/mcp/` with the same `cmbx_live_*` key
as a Bearer token, and neither keeps the key in this repository:

- **Plugin:** the key is read from your environment at runtime, through the
  `${CHEMBRICKS_TOKEN}` placeholder in `.mcp.json`.
- **Extension:** Claude Desktop asks for the key during installation, keeps it
  in your operating system's secure storage, and passes it to the local
  process as an environment variable, so it never appears in the process list.

## Links

- **Website:** [cmbx.ai](https://cmbx.ai)
- **MCP Server:** `https://mcp.cmbx.ai/mcp/`
- **Server Health:** [mcp.cmbx.ai/health](https://mcp.cmbx.ai/health)
- **Connection Guide:** [docs/CONNECT.md](https://github.com/Chembricks/chembricks-mcp/blob/main/docs/CONNECT.md)

## License

[MIT](LICENSE)
