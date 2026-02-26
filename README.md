# OpenNucFold — User Guide

**Open-source nucleic acid folding & design toolkit**
*NUPACK-like workflows via ViennaRNA — runs entirely offline*

---

## Quick Start

```bash
# 1. Install ViennaRNA (required)
#    See "Backend Installation" below

# 2. Install OpenNucFold
pip install PyQt6 matplotlib numpy scipy
cd opennucfold/
pip install -e .

# 3. Launch
opennucfold
# or: python -m opennucfold.app
```

---

## Backend Installation

### ViennaRNA (required — primary backend)

**Linux (Ubuntu/Debian):**
```bash
sudo apt install vienna-rna
# or from source:
wget https://www.tbi.univie.ac.at/RNA/download/sourcecode/2_6_x/ViennaRNA-2.6.4.tar.gz
tar xzf ViennaRNA-2.6.4.tar.gz && cd ViennaRNA-2.6.4
./configure && make -j$(nproc) && sudo make install
```

**macOS:**
```bash
brew install viennarna
```

**Windows:**
- Download installer from https://www.tbi.univie.ac.at/RNA/#download
- Add ViennaRNA `bin/` to your PATH environment variable
- Verify: open Command Prompt and run `RNAfold --version`

### UNAFold (optional — DNA Tm, enhanced duplex)

**Linux:**
```bash
# Download from http://www.unafold.org/
tar xzf unafold-3.9.tar.gz && cd unafold-3.9
./configure && make && sudo make install
```

**macOS:**
```bash
# Same as Linux; may need Xcode command line tools
```

**Windows:**
- Build from source with MinGW/MSYS2, or use WSL

### Verifying Installation

Launch OpenNucFold → **Tools → Backend Status** shows detected backends:
```
✓ ViennaRNA 2.6.4
✓ UNAFold 3.9
```
or
```
✓ ViennaRNA 2.6.4
✗ UNAFold  (not found)
```

---

## Tab 1 — Single Strand Folding

**Purpose:** Predict the minimum free energy (MFE) secondary structure and
compute the partition function for a single RNA or DNA strand.

**Inputs:**
- Sequence (paste raw or FASTA format)
- RNA / DNA toggle
- Temperature (°C), Na⁺ concentration, Mg²⁺ (if backend supports)
- Optional dot-bracket constraint

**Outputs:**
- MFE structure in dot-bracket notation
- ΔG (kcal/mol)
- Base-pair probability dot plot (interactive zoom/pan)
- Per-base pairing probability bar chart
- Ensemble free energy and diversity

**Backend:** ViennaRNA `RNAfold -p --MEA`

**Export:** JSON report, PNG/SVG plots

---

## Tab 2 — Duplex / Co-folding

**Purpose:** Predict heterodimer structure between two strands, compare
against self-folding, and estimate Tm.

**Inputs:**
- Strand A and Strand B sequences
- RNA / DNA toggle
- Temperature + salt conditions

**Outputs:**
- Heterodimer structure and ΔG (via RNAcofold)
- Self-fold of each strand (via RNAfold)
- Binding risk indicator: green if heterodimer is favorable, yellow if
  self-folding competes
- Tm estimate (UNAFold if available, otherwise simple formula)

**Backend:** ViennaRNA `RNAcofold` + `RNAfold`; UNAFold for DNA Tm

---

## Tab 3 — Multi-Strand Mixture Analysis

**Purpose:** Approximate equilibrium complex concentrations for a mixture
of strands — analogous to NUPACK's `complexes` / `concentrations` analysis.

**Inputs:**
- Up to 8 strands (name, sequence, concentration in µM)
- Maximum complex size (2–4)
- Temperature + salts

**Method:**
1. Enumerate all candidate complexes up to the specified size
2. Compute ΔG for each complex via ViennaRNA cofold
3. Estimate relative concentrations using Boltzmann-weighted mass-action

**⚠ Important Limitation:**
This is an *approximate* method. Unlike NUPACK, it does NOT perform full
partition-function enumeration over all secondary structures of all complexes.
Results are useful for **screening** and **qualitative comparison**, but
should not be treated as quantitative thermodynamic predictions. For
publication-grade thermodynamics, use NUPACK directly.

**Outputs:**
- Table: complex stoichiometry, ΔG, estimated fraction, concentration
- Adjustable filter threshold
- Warnings for potential cross-talk

---

## Tab 4 — Sequence Design

**Purpose:** Propose nucleotide sequences that fold into a target secondary
structure, subject to constraints.

**Inputs:**
- Target structure (dot-bracket notation)
- RNA / DNA toggle
- GC content range
- Forbidden motifs (comma-separated)
- Max homopolymer length
- Optimizer settings (SA steps, restarts, top K)

**Method:**
Simulated annealing with a composite score function:
- Structure match (heavy weight) — does the sequence fold into the target?
- GC content penalty
- Homopolymer and forbidden-motif penalties
- Multiple independent restarts for diversity

**⚠ Limitation vs NUPACK Design:**
This is a heuristic optimizer, not a formal inverse-folding algorithm.
It works well for simple structures (hairpins, stem-loops, small motifs)
but may struggle with complex multi-domain structures. For mission-critical
design, validate candidates experimentally or cross-check with NUPACK.

**Outputs:**
- Top K candidate sequences with scores
- For each: predicted structure, ΔG, structure match %, GC%
- Export FASTA + JSON report

---

## Reproducibility

Every analysis can be exported as a JSON report containing:
- All input parameters
- Tool name and version
- Operating system and Python version
- Complete results
- Timestamp

This ensures any result can be reproduced exactly.

---

## Comparison with NUPACK

| Feature | NUPACK | OpenNucFold |
|---|---|---|
| Single-strand MFE | ✓ | ✓ (ViennaRNA) |
| Partition function | ✓ (full) | ✓ (ViennaRNA) |
| Base-pair probabilities | ✓ | ✓ |
| Multi-strand complexes | ✓ (exact) | ≈ (approximate) |
| Concentration prediction | ✓ (exact) | ≈ (Boltzmann approx.) |
| Sequence design | ✓ (advanced) | ✓ (heuristic SA) |
| DNA parameters | ✓ | ✓ (ViennaRNA + UNAFold) |
| Tm estimation | ✓ | ✓ (UNAFold or formula) |
| Runs offline | ✗ (web) / ✓ (local) | ✓ (always) |
| Open source | ✗ (academic license) | ✓ (MIT) |
| GUI | Web | Native desktop (PyQt6) |

**Bottom line:** OpenNucFold covers the most common day-to-day workflows
(folding, duplex checks, quick mixture screening, basic design) using
fully open-source tools. For rigorous multi-strand thermodynamics at
equilibrium, NUPACK remains the gold standard.

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| Ctrl+Q | Quit |

---

## Troubleshooting

**"No backends found"**
→ Install ViennaRNA and ensure `RNAfold` is on your PATH.

**"Tool not found: RNAfold"**
→ ViennaRNA is not installed or not on PATH. Try `which RNAfold` (Linux/Mac)
  or `where RNAfold` (Windows).

**Dot plot is empty**
→ The partition function run may not have generated a `.ps` file.
  Ensure ViennaRNA ≥ 2.4 is installed.

**DNA folding gives unexpected results**
→ ViennaRNA DNA parameters are less extensively validated than RNA.
  Consider using UNAFold for DNA-specific calculations.

---
## License
This project is licensed under a Non-Commercial Open Source License. This license allows you to use, modify, and distribute the software for non-commercial purposes only. Commercial use is prohibited without explicit permission from the copyright holder. Additionally, you may not patent or claim intellectual property rights over the software or any derivative works.

© Sharma, Deepanshu (2026), Germany.
---
*OpenNucFold v1.0.0*
