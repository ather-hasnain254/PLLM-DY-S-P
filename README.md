# PLLM-DY: A Probabilistic Dolev-Yao Model for Multi-Agent LLM Systems

## Reproducible Implementation

This repository contains the complete, executable codebase for reproducing all results
from the paper *"PLLM-DY: A Probabilistic Dolev-Yao Attacker Model for Multi-Agent LLM Systems"*.

---

## Table of Contents

1. [Overview](#overview)
2. [Paper-to-Code Mapping](#paper-to-code-mapping)
3. [Installation](#installation)
4. [Dataset Requirements](#dataset-requirements)
5. [Running Experiments](#running-experiments)
6. [Expected Outputs](#expected-outputs)
7. [Folder Structure](#folder-structure)
8. [ProVerif Verification](#proverif-verification)
9. [Assumptions and Limitations](#assumptions-and-limitations)

---

## Overview

PLLM-DY extends the classical Dolev-Yao attacker model with:

- **Probabilistic derivation rules** (R1–R5) capturing LLM-specific attack vectors
- **Tool-output corruption primitive** modeling indirect prompt injection
- **ε-feasible derivability** replacing binary DY derivability
- **Formal verification** of centralized and decentralized agent communication protocols
- **ε-bounded attacker success analysis** with quantitative security guarantees

This codebase implements:

| Component | Description | Paper Section |
|-----------|-------------|---------------|
| Probabilistic Term Algebra | Term generation, derivation rules R1–R5 | §4.1 |
| Tool-Output Corruption | Two-stage IPI primitive | §4.2 |
| ε-Capability Set | Probabilistic capability computation | §4.3 |
| Taxonomy Mapping | OWASP/Ferrag → PLLM-DY rules | §5 |
| ProVerif Models | Protocol 1 (centralized) and Protocol 2 (DID) | §6 |
| Security Analysis | Equivalence proofs, trade-off analysis | §7.1 |
| ε-Bounded Analysis | Case studies 1 & 2 with parameter sweeps | §7.2 |

---

## Paper-to-Code Mapping

| Paper Element | File(s) | Notes |
|--------------|---------|-------|
| Definition 1 (Term Algebra) | `src/term_algebra.py` | Classes for all term types |
| Definition 2 (Prob. Derivation) | `src/derivation_rules.py` | Functions f_DPI, f_IPI, f_CP, f_AC, f_SE |
| Definition 3 (Tool Corruption) | `src/derivation_rules.py` → `ToolOutputCorruption` | Two-stage rule |
| Definition 4 (ε-Feasibility) | `src/epsilon_feasibility.py` | BFS/DFS over derivation graph |
| Definition 5 (Capability Set) | `src/capability_set.py` | Full capability computation |
| Theorem 1 (Expressiveness) | `tests/test_theorems.py` | Automated verification |
| Theorem 2 (Monotonicity) | `tests/test_theorems.py` | Property-based tests |
| Theorem 3 (Completeness) | `experiments/taxonomy_mapping.py` | Exhaustive mapping |
| Theorems 4–6 (Verification) | `proverif/protocol_*.pv` | ProVerif models |
| Theorem 7 (Equivalence) | `experiments/security_equivalence.py` | Simulation |
| Table 1 (Attack Mapping) | `experiments/taxonomy_mapping.py` | Generates table |
| Table 2 (Trade-offs) | `experiments/security_equivalence.py` | Generates table |
| Case Study 1 (IPI) | `experiments/case_study_ipi.py` | Parameter sweeps |
| Case Study 2 (Cascade) | `experiments/case_study_cascade.py` | Graph analysis |
| All Figures | `scripts/generate_all_figures.py` | Reproduces all plots |

---

## Installation

### Prerequisites

- Python 3.9+
- ProVerif 2.04+ (optional, for formal verification)

### Setup

```bash
# Clone or copy this repository
cd pllm-dy

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows

# Install dependencies
pip install -r requirements.txt

# Verify installation
python -m pytest tests/ -v
```

### ProVerif (Optional)

```bash
# Ubuntu/Debian
sudo apt-get install proverif

# macOS (Homebrew)
brew install proverif

# Or download from: https://bblanche.gitlabpages.inria.fr/proverif/
```

---

## Dataset Requirements

This paper is primarily theoretical/formal, so **no large external datasets are required**.
However, we use empirical parameters from published studies to calibrate our models:

### Required Data (auto-generated)

The simulation uses parameters derived from published empirical studies:

| Parameter | Source | Value Range |
|-----------|--------|-------------|
| GPT-4 jailbreak success rate | Shen et al. [35], CCS 2024 | 0.10–0.30 |
| Tool injection success rate | InjecAgent [38] | ~0.24 |
| Multi-agent cascade rates | He et al. [10], ACL 2025 | 0.05–0.80 |
| Temperature sensitivity | Zou et al. [33] | τ ∈ [0.0, 2.0] |

These are encoded in `config/empirical_parameters.yaml`.

### Optional: Real LLM Attack Data

For extended experiments with real attack data, you can download:

1. **JailbreakBench** (jailbreak prompts):
   - Source: https://github.com/JailbreakBench/jailbreakbench
   - Usage: Calibrate f_DPI parameters
   - Place in: `data/jailbreakbench/`

2. **InjecAgent** (indirect prompt injection):
   - Source: https://github.com/uiuc-kang-lab/InjecAgent
   - Usage: Calibrate f_IPI parameters
   - Place in: `data/injecagent/`

3. **AgentDojo** (agent security benchmark):
   - Source: https://github.com/ethz-spylab/agentdojo
   - Usage: Validate ε-bounds against real tool-use attacks
   - Place in: `data/agentdojo/`

4. **DAN Jailbreak Dataset** (in-the-wild prompts):
   - Source: Collected by Shen et al. (CCS 2024)
   - Usage: Calibrate R5 semantic equivalence parameters
   - Place in: `data/dan_jailbreaks/`

Run `python scripts/download_datasets.py --list` to see all options.

---

## Running Experiments

### Quick Start: Run Everything

```bash
# Run all experiments and generate all figures
python scripts/run_all.py

# Results will be saved in results/ and figures/
```

### Individual Experiments

```bash
# 1. Probabilistic term algebra demonstration (§4.1)
python experiments/term_algebra_demo.py

# 2. Tool-output corruption analysis (§4.2)
python experiments/tool_corruption_demo.py

# 3. ε-feasibility computation (§4.3)
python experiments/epsilon_feasibility_demo.py

# 4. Taxonomy mapping verification (§5, Table 1)
python experiments/taxonomy_mapping.py

# 5. Security equivalence analysis (§7.1, Table 2)
python experiments/security_equivalence.py

# 6. Case Study 1: Indirect Prompt Injection (§7.2)
python experiments/case_study_ipi.py

# 7. Case Study 2: Multi-Agent Cascade (§7.2)
python experiments/case_study_cascade.py

# 8. Generate all figures
python scripts/generate_all_figures.py
```

### ProVerif Verification (§6)

```bash
# Protocol 1: Centralized Authority
proverif proverif/protocol_centralized.pv

# Protocol 2: Decentralized DID-Based
proverif proverif/protocol_did.pv

# Protocol 2 with PLLM-DY attacker extensions
proverif proverif/protocol_did_pllmdy.pv
```

---

## Expected Outputs

After running `python scripts/run_all.py`, you should see:

### results/
- `taxonomy_mapping.json` — Complete attack class → rule mapping (Table 1)
- `security_equivalence.json` — Protocol comparison data (Table 2)
- `case_study_ipi.json` — IPI parameter sweep results
- `case_study_cascade.json` — Cascade analysis results
- `epsilon_analysis.json` — ε-bounded success probabilities
- `theorem_verification.json` — Automated theorem check results

### figures/
- `fig1_epsilon_vs_temperature.png` — ε-feasibility as function of τ
- `fig2_cascade_probability.png` — Cascade attack probability vs. chain length
- `fig3_attack_surface_heatmap.png` — α(θ) heatmap over (τ, φ) space
- `fig4_protocol_comparison.png` — Security equivalence visualization
- `fig5_taxonomy_mapping.png` — Attack taxonomy → rule mapping diagram
- `fig6_capability_set_growth.png` — |Cap_ε(S)| vs. ε threshold

---

## Folder Structure

```
pllm-dy/
├── README.md                          # This file
├── requirements.txt                   # Python dependencies
├── config/
│   └── empirical_parameters.yaml      # Empirical params from literature
├── src/
│   ├── __init__.py
│   ├── term_algebra.py                # Def 1: PLLM-DY term algebra
│   ├── derivation_rules.py            # Def 2-3: Probabilistic rules R1-R5
│   ├── epsilon_feasibility.py         # Def 4: ε-feasible derivability
│   ├── capability_set.py              # Def 5: Probabilistic capability set
│   ├── attack_graph.py                # Attack graph construction & analysis
│   ├── agent_system.py                # Multi-agent LLM system simulator
│   └── utils.py                       # Shared utilities
├── models/
│   ├── __init__.py
│   ├── llm_agent.py                   # Simulated LLM agent model
│   ├── attacker.py                    # PLLM-DY attacker implementation
│   └── protocol.py                    # Protocol 1 & 2 implementations
├── experiments/
│   ├── __init__.py
│   ├── term_algebra_demo.py           # §4.1 demonstration
│   ├── tool_corruption_demo.py        # §4.2 demonstration
│   ├── epsilon_feasibility_demo.py    # §4.3 demonstration
│   ├── taxonomy_mapping.py            # §5 complete mapping + Theorem 3
│   ├── security_equivalence.py        # §7.1 equivalence analysis
│   ├── case_study_ipi.py             # §7.2 Case Study 1
│   └── case_study_cascade.py         # §7.2 Case Study 2
├── proverif/
│   ├── protocol_centralized.pv        # §6: Protocol 1 model
│   ├── protocol_did.pv                # §6: Protocol 2 model
│   └── protocol_did_pllmdy.pv         # §6: Protocol 2 + PLLM-DY extensions
├── scripts/
│   ├── run_all.py                     # Master script
│   ├── generate_all_figures.py        # All paper figures
│   └── download_datasets.py           # Optional dataset downloader
├── tests/
│   ├── __init__.py
│   ├── test_term_algebra.py           # Unit tests for term algebra
│   ├── test_derivation_rules.py       # Unit tests for rules
│   ├── test_theorems.py               # Automated theorem verification
│   └── test_integration.py            # End-to-end tests
├── data/                              # Datasets (auto-generated or downloaded)
├── results/                           # Experiment outputs
└── figures/                           # Generated figures
```

---

## Assumptions and Limitations

### Documented Assumptions

1. **Derivation success functions** (f_DPI, f_IPI, etc.) are modeled as sigmoid-based
   functions calibrated to empirical data from published studies. The exact functional
   forms are assumptions since the paper defines them abstractly (§4.1).

2. **Semantic equivalence** (~_sem in R5) is approximated using cosine similarity in
   a simulated embedding space. A production implementation would require an actual
   embedding model (§4.1, R5).

3. **ProVerif encoding** uses the deterministic skeleton of PLLM-DY. Probabilistic
   reasoning is handled separately in the Python simulation code (§6.2).

4. **LLM agent behavior** is simulated using parameterized stochastic models rather
   than actual LLM inference, consistent with the paper's theoretical framework (§3).

5. **Empirical parameters** (attack success rates, temperature sensitivity) are drawn
   from published results cited in the paper, not from new experiments.

### Known Limitations

- The ProVerif models verify the deterministic protocol skeleton; full probabilistic
  model checking would require tools like PRISM or Storm (noted in §8.2).
- Semantic equivalence simulation is approximate; a full implementation would require
  access to an embedding model API.
- The codebase focuses on reproducing the theoretical results and parameter analysis
  from the paper, not on attacking real LLM systems.

---

## Citation

If you use this code, please cite the paper:

```bibtex
@inproceedings{pllmdy2026,
  title={PLLM-DY: A Probabilistic Dolev-Yao Model 
         for Multi-Agent LLM Systems},
  author={[Redacted for Review]},
  booktitle={[Under Submission]},
  year={2026}
}
```

---

## License

This implementation is provided for academic research and reproducibility purposes.
