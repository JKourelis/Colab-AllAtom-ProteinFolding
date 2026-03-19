# Protein Structure Prediction — Colab Notebooks

> **Manuscript in preparation** — Kourelis J. Accessible batch biomolecular structure prediction via Google Colab. 2026.

Google Colab notebooks for batch biomolecular structure prediction. All tools share a unified CSV input format with calibration-based parallel GPU scheduling and automatic Google Drive upload.

## Notebooks

### MSA Pre-computation (optional)

Pre-computes MSAs once for all unique protein sequences and produces pre-paired alignments that every folding tool can reuse. **No GPU required** (CPU-only).

**This step is optional.** Without pre-computed MSAs, each folding notebook queries the [ColabFold](https://github.com/sokrypton/ColabFold) MMseqs2 server on-the-fly during prediction (~30–120 s per protein chain). Pre-computing is recommended when:
- **Running the same sequences through multiple tools** — avoids redundant server queries
- **Running large batches** — ColabFold rate-limits parallel requests; pre-computing lets you search sequentially, then fold in parallel
- **Custom cross-species MSA pairing** — e.g., host–pathogen complexes where standard TaxID-based pairing produces no paired rows

| Notebook | Colab Link | GPU | Description |
|----------|-----------|-----|-------------|
| **MSA Pre-computation** | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1kUEp55gbRm_9zFBQN4FQUaeaxdFh04BK) | CPU | Pre-compute paired MSAs once; reuse across all 5 folding tools |

### Folding Notebooks

| Tool | Colab Link | GPU | Description |
|------|-----------|-----|-------------|
| [**Boltz-2**](https://github.com/jwohlwend/boltz) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1maLULdT_lyid0-tRvrdldrSsZaiXpIuL) | T4+ | Structure + binding affinity, MIT license |
| [**Chai-1**](https://github.com/chaidiscovery/chai-lab) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1vkzrziFd8t4PIf2HFzpiDCnUu9ytU9PP) | L4/A100 | Requires bf16, strong single-sequence performance |
| [**IntelliFold**](https://github.com/IntelliGen-AI/IntelliFold) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1VNPsmVu45b1xf5jvAE0N4c38HZb0w74_) | T4+ | AF3-level accuracy, v2/v2-flash models |
| [**OpenFold3**](https://github.com/aqlaboratory/openfold-3) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1HeoAwO4XQStRK7Qq6dcUAstyULblukL1) | A100 | 32GB+ VRAM, open-source AF3 reimplementation |
| [**Protenix**](https://github.com/bytedance/Protenix) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1lGmcSPIFvhkfpJXUqjET-ikQzZWwi4D4) | T4+ (Mini) / A100 (Base) | ByteDance AF3 implementation, 8 model variants |

## GPU Requirements

| Colab GPU | VRAM | IntelliFold | Boltz-2 | Protenix | Chai-1 | OpenFold3 |
|-----------|------|-------------|---------|----------|--------|-----------|
| T4 (free) | 15 GB | Yes | Yes | Mini only | **No** (needs bf16) | **No** (needs 32GB+) |
| L4 (paid) | 24 GB | Yes | Yes | Mini/Base (small) | Yes | **No** |
| A100 (Pro) | 40 GB | Yes | Yes | Yes | Yes | Yes (short seqs) |
| A100 80 GB | 80 GB | Yes | Yes | Yes | Yes | Yes |
| H100 | 80 GB | Yes | Yes | Yes | Yes | Yes |
| G4 (Blackwell)\* | 96 GB | Yes | Yes | Yes | Untested | Untested |

\*G4 = NVIDIA RTX PRO 6000 Blackwell Server Edition (sm\_120). Automatic compatibility patches are applied during install. IntelliFold runs at full speed (native sm\_120 fast\_layernorm). Boltz-2 and Protenix use cuEquivariance CUDA kernels for triangle\_attention (full speed) but fall back to PyTorch native ops for triangle\_multiplicative (~20% overhead on that operation) due to a Triton kernel incompatibility with sm\_120. The 96 GB VRAM more than compensates — enables larger complexes and more parallel jobs than any other Colab GPU.

**Recommended:** A100 80 GB or H100 for general use across all tools. The free T4 handles small monomers with IntelliFold, Boltz-2, or Protenix Mini. For large complexes (>2000 tokens, many chains) or parallel batch runs, A100 80 GB, H100, or G4 provide the VRAM headroom needed.

## Unified CSV Format

All notebooks accept the same CSV. See [`csv_template.csv`](csv_template.csv) for the empty template.

### Columns

| Column | Required | Description |
|--------|----------|-------------|
| `jobname` | Yes | Unique job identifier |
| `seq{N}_name` | Yes (seq1) | Chain/entity name |
| `seq{N}_type` | Yes (seq1) | `protein`, `dna`, `rna`, or `ligand` |
| `seq{N}` | Yes (seq1) | Sequence or ligand code (CCD/SMILES) |
| `seq{N}_copies` | No | Number of copies (default: 1) |
| `seq{N}_mods` | No | Modifications: `CCD_CODE:POSITION;...` |
| `pocket_binder` | No | Binder chain for pocket constraint |
| `pocket_contacts` | No | Contact residues: `CHAIN:RES;CHAIN:RES;...` |
| `covalent_bonds` | No | Bonds: `CHAIN:RES:ATOM:CHAIN:RES:ATOM;...` |

Supports up to 10 sequences per job (seq1 through seq10). One row = one prediction job.

### Sequence Types

- **protein** — amino acid sequence (ACDEFGHIKLMNPQRSTVWY)
- **dna** — DNA bases (ATCG)
- **rna** — RNA bases (AUCG)
- **ligand** — CCD code (e.g., `ATP`, `MG`) or SMILES string

## CSV Examples

**Monomer:**
```csv
jobname,seq1_name,seq1_type,seq1,seq1_copies
MyProtein,A,protein,MKTAYIAKQRQISFVKS,1
```

**Homodimer (via copies):**
```csv
jobname,seq1_name,seq1_type,seq1,seq1_copies
Homodimer,A,protein,MKTAYIAK,2
```

**Heterodimer:**
```csv
jobname,seq1_name,seq1_type,seq1,seq1_copies,seq2_name,seq2_type,seq2,seq2_copies
Heterodimer,A,protein,MKTAYIAK,1,B,protein,MLKFSG,1
```

**Protein-ligand:**
```csv
jobname,seq1_name,seq1_type,seq1,seq1_copies,seq2_name,seq2_type,seq2,seq2_copies
ProteinATP,A,protein,MKTAYIAK,1,B,ligand,ATP,1
```

**With phosphorylation (PTM):**
```csv
jobname,seq1_name,seq1_type,seq1,seq1_copies,seq1_mods
WithPTM,A,protein,MKTAYIAK,1,CCD_SEP:8
```
*CCD_SEP = phosphoserine at position 8*

**With pocket restraints:**
```csv
jobname,seq1_name,seq1_type,seq1,seq1_copies,seq2_name,seq2_type,seq2,seq2_copies,pocket_binder,pocket_contacts
WithPocket,A,protein,MKTAYIAK,1,B,ligand,ATP,1,B,A:10;A:15;A:20
```
*Ligand B binds at protein A residues 10, 15, 20*

## Embedded Reference Data

All notebooks include built-in support for common modifications (upload custom reference files if needed):

- **PTMs (15)**: SEP, TPO, PTR, MLY, ALY, HYP, M3L, MLZ, CSD, CSO, CGU, FME, NEP, HIC, CAS
- **Ligands (24)**: ATP, ADP, AMP, GTP, GDP, GMP, CTP, CDP, UTP, UDP, NAD, NAP, FAD, FMN, HEM, SAH, SAM, COA, ACO, PLP, TPP, BTN, MTA, THM
- **Ions (11)**: MG, ZN, CA, FE, FE2, MN, CU, NI, CO, K, NA, CL
- **Glycans (10)**: NAG, BMA, MAN, FUC, GAL, GLC, SIA, FUL, XYS, ARA
- **DNA mods (8)**: 5CM, 6OG, 8OG, 5HC, 5FC, 5CA, 6MA, 4MC
- **RNA mods (10)**: PSU, 5MC, M2G, M7G, 1MA, 5MU, OMC, OMG, 2MA, I

## How It Works

Each notebook follows a 7-cell structure: header, kernel restart, install, CSV processor, upload + GDrive, settings, run + download. The run cell uses calibration-based parallel scheduling — jobs are automatically sorted largest-first, the largest job runs alone to measure VRAM, then remaining jobs are scheduled in parallel when GPU memory allows. Results are zipped and uploaded to Google Drive after each job.

## Interface Analysis (ipSAE_batch)

All folding notebooks include built-in [ipSAE_batch](https://github.com/JKourelis/ipSAE_batch) interface analysis. After each prediction, ipSAE_batch automatically scores interface quality and generates visualizations. A final batch comparison across all jobs produces combined CSVs and interactive HTML plots.

**Scores**: ipSAE, ipTM, AlphaBridge, contact probability, PAE, PMC, pDockQ, pDockQ2, local interaction scores.

**Visualizations**: PMC/contact-probability matrix plots, AlphaBridge-style circular ribbon diagrams with colored arcs for confident interface contacts, PAE matrices, and interactive HTML comparisons (scatter plots, jitter plots, correlation matrices with threshold filtering).

<table>
<tr>
<td width="50%"><b>A) Joint Matrix Plot</b></td>
<td width="50%"><b>B) Ribbon Diagram</b></td>
</tr>
<tr>
<td><img src="https://github.com/JKourelis/ipSAE_batch/raw/main/images/example_model_matrix.png" width="100%"/></td>
<td><img src="https://github.com/JKourelis/ipSAE_batch/raw/main/images/example_model_ribbon.png" width="100%"/></td>
</tr>
<tr>
<td>Upper triangle: PMC (Predicted Merged Confidence) combining pLDDT and PAE into a single confidence metric. Lower triangle: Contact probability derived from PAE and inter-residue distance. Chain boundaries shown with gaps; ticks indicate residue positions with chain labels (\u25bc) marking N-terminus direction.</td>
<td>Circular representation of protein chains with interface regions. Colored arcs indicate confident interface contacts; grey arcs show contacts below the PAE threshold. Interface regions and contact lines use matching colors.</td>
</tr>
</table>

Enable/disable via the `enable_ipsae` toggle in each notebook's Settings cell.

For standalone use outside these notebooks, see the [ipSAE_batch Colab](https://colab.research.google.com/github/JKourelis/ipSAE_batch/blob/main/ipSAE_batch_colab.ipynb).

## Pre-computed MSAs

The MSA Pre-computation notebook runs MSA searches once for all unique protein sequences and performs cross-chain pairing. The paired MSAs can then be used by any folding notebook, skipping redundant MSA searches at folding time.

**Workflow:**
1. Upload the same CSV to the MSA notebook
2. Run — it searches MSAs per protein and pairs them per job
3. Note the output folder path (local or GDrive)
4. In any folding notebook's Settings cell, paste the `paired/` path into `msa_folder`
5. Folding runs use pre-computed MSAs instead of searching online

## Repository Structure

```
Protein_folding/
├── README.md                    # This file
├── csv_template.csv             # Unified CSV template (all tools)
├── Colab_IntelliFold/
│   ├── IntelliFold.ipynb
│   ├── README.md
│   └── KNOWLEDGE/
├── Colab_Boltz2/
│   ├── Boltz2.ipynb
│   └── README.md
├── Colab_Protenix/
│   ├── Protenix.ipynb
│   └── README.md
├── Colab_Chai1/
│   ├── Chai1.ipynb
│   └── README.md
├── Colab_OpenFold3/
│   ├── OpenFold3.ipynb
│   └── README.md
└── Colab_MSA/
    ├── MSA.ipynb
    └── README.md
```

## Citation

If you use these notebooks, please cite them and the respective tools:

**These notebooks:**
```
Kourelis, J. (2026). Accessible batch biomolecular structure prediction
via Google Colab. [Manuscript in preparation]
```

**Boltz-2:**
```
Passaro, S., Corso, G., Wohlwend, J., et al. (2025). Boltz-2: Towards
Accurate and Efficient Binding Affinity Prediction. bioRxiv.
https://doi.org/10.1101/2025.06.14.659707

Wohlwend, J., Corso, G., Passaro, S., et al. (2024). Boltz-1:
Democratizing Biomolecular Interaction Modeling. bioRxiv.
https://doi.org/10.1101/2024.11.19.624167
```

**Chai-1:**
```
Chai Discovery (2025). Zero-shot antibody design in a 24-well plate.
bioRxiv. https://doi.org/10.1101/2025.07.05.663018
```

**IntelliFold:**
```
Qiao, L., Yan, H., Liu, G., Guo, G. & Sun, S. (2026). IntelliFold-2:
Surpassing AlphaFold 3 via Architectural Refinement and Structural
Consistency. bioRxiv.

The IntFold Team, Qiao, L., et al. (2025). IntFold: A Controllable
Foundation Model for General and Specialized Biomolecular Structure
Prediction. arXiv. https://arxiv.org/abs/2507.02025
```

**OpenFold3:**
```
The OpenFold3 Team (2025). OpenFold3-preview (v0.2.0). Zenodo.
https://doi.org/10.5281/zenodo.17485510
```

**Protenix:**
```
Zhang, Y., Gong, C., Zhang, H., et al. (2026). Protenix-v1: Toward
High-Accuracy Open-Source Biomolecular Structure Prediction. bioRxiv.
https://doi.org/10.64898/2026.02.05.703733
```

**Foundational work (OpenFold3/Protenix build on AF3):**
```
Abramson, J., et al. (2024). Accurate structure prediction of
biomolecular interactions with AlphaFold 3. Nature, 630, 493-500.
```

**Interface analysis (ipSAE_batch, included in all notebooks):**
```
Dunbrack, R. L. (2025). ipSAE: A score for confident satisfactory
protein-protein interactions predicted by AlphaFold2 and AlphaFold3.
bioRxiv. https://doi.org/10.1101/2025.02.10.637595

Alvarez-Salmoral, D., et al. (2024). AlphaBridge: tools for the
analysis of predicted biomolecular complexes. bioRxiv.
https://doi.org/10.1101/2024.10.23.619601
```

**MSA search (ColabFold, used by all notebooks):**
```
Mirdita, M., Schutze, K., Moriwaki, Y., Heo, L., Ovchinnikov, S. &
Steinegger, M. (2022). ColabFold: making protein folding accessible
to all. Nature Methods, 19(6), 679-682.
```

## License

Apache License 2.0 — see [LICENSE](LICENSE).

Individual tools have their own licenses:
- IntelliFold: Apache 2.0
- Boltz-2: MIT
- Protenix: Apache 2.0
- Chai-1: Apache 2.0
- OpenFold3: Apache 2.0
