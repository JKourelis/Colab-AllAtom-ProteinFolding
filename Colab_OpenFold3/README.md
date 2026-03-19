# OpenFold3 — Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1HeoAwO4XQStRK7Qq6dcUAstyULblukL1)

Batch structure prediction with [OpenFold3](https://github.com/aqlaboratory/openfold-3) v0.3.1 (preview), the OpenFold Consortium's open-source reimplementation of AlphaFold3.

## Features

- Open-source AF3 reimplementation with full training code
- Supports proteins, DNA, RNA, ligands (CCD + SMILES), and ions
- Configuration via runner YAML for advanced control (seeds, precision, kernels)
- MSA via ColabFold server
- cuEquivariance kernel acceleration (auto-detected)
- Built-in [ipSAE_batch](https://github.com/JKourelis/ipSAE_batch) interface analysis — ipSAE, ipTM, AlphaBridge, pDockQ scores + matrix plots, AlphaBridge-style ribbon diagrams, and interactive HTML comparisons. Also available as a [standalone Colab](https://colab.research.google.com/github/JKourelis/ipSAE_batch/blob/main/ipSAE_batch_colab.ipynb)

## GPU Requirements

**OpenFold3 requires minimum 32 GB VRAM. A100 is the minimum viable Colab GPU.**

| Colab GPU | VRAM | Compatible | Notes |
|-----------|------|------------|-------|
| T4 (free) | 15 GB | **No** | Insufficient VRAM |
| L4 (paid) | 24 GB | **No** | Insufficient VRAM |
| A100 (Pro) | 40 GB | Yes | Short/medium sequences |
| A100 80GB | 80 GB | Yes | Full capability |

## Quick Start

1. Read Cell 0 for overview and CSV format
2. Run Cell 1: Restart kernel (if needed)
3. Run Cell 2: Install OpenFold3 and dependencies
4. Run Cell 3: Initialize CSV processor
5. Run Cell 4: Upload CSV + connect Google Drive
6. Run Cell 5: Configure prediction settings
7. Run Cell 6: Run predictions + download results

See the [main README](../README.md) for CSV format documentation and examples.

## Performance Tips

**Parallel scheduling**: Jobs are automatically sorted largest-first for optimal calibration. The largest job calibrates VRAM usage, giving the most accurate per-token rate for parallel scheduling of remaining jobs.

## Known Limitations

- **Preview release** (v0.3.1) — active development, expect rough edges
- DeepSpeed evo attention disabled (requires CUTLASS source tree for JIT compilation)
- Template search disabled by default (IndexError bug, issue #101)
- Parallel jobs: MSA cache cleanup managed at batch level to prevent file collisions
- Requires `kalign2` and `hmmer` system packages (installed automatically)
- **Empty-MSA multimer crash** (see below)

## Known Bug: Empty-MSA Multimer Crash (UNRESOLVED)

Multimer jobs where one chain has **no MSA homologs** (e.g., pathogen effectors like Avr2 with no sequence database hits) can crash during MSA processing. The root cause is a column-count mismatch between the main and paired MSA arrays for that chain.

**When it happens:** ColabFold returns a query-only MSA for the no-homolog chain. The paired a3m and main a3m files can have different effective alignment widths after parsing. This mismatch propagates to two crash sites:

1. `crop_vstack_msa_arrays()` — `ValueError: Cannot concatenate along axis=0: number of columns must match`
2. `create_main()` — `TypeError: cannot compare unstructured voids of different length`

**When it does NOT happen:** If the complex has 3+ chains and at least 2 chains have homologs, the pairing pipeline has enough data to produce correctly-sized gap-filled MSAs for all chains (including the no-homolog chain). Example: `cf2_rcr3pim_avr2` (3 chains, 2 with homologs) succeeds while `rcr3_avr2` (2 chains, 1 with homologs) fails despite sharing the same Avr2 chain.

**Status:** No fix currently applied. A proper fix requires upstream patches to normalize paired MSA column counts after parsing. This is an OpenFold3 bug (no GitHub issue filed yet as of Feb 2026).

## Citation

If you use this notebook, please cite:

**This notebook:**
```
Kourelis, J. (2026). Accessible batch biomolecular structure prediction
via Google Colab. [Manuscript in preparation]
```

**OpenFold3:**
```
The OpenFold3 Team (2025). OpenFold3-preview (v0.2.0). Zenodo.
https://doi.org/10.5281/zenodo.17485510
```

**Foundational work:**
```
Abramson, J., et al. (2024). Accurate structure prediction of
biomolecular interactions with AlphaFold 3. Nature, 630, 493-500.

Ahdritz, G., Bouatta, N., et al. (2024). OpenFold: Retraining
AlphaFold2 yields new insights into its learning mechanisms and
capacity for generalization. Nature Methods, 21(8), 1514-1524.
```

**Interface analysis (ipSAE_batch):**
```
Dunbrack, R. L. (2025). ipSAE: A score for confident satisfactory
protein-protein interactions predicted by AlphaFold2 and AlphaFold3.
bioRxiv. https://doi.org/10.1101/2025.02.10.637595

Alvarez-Salmoral, D., et al. (2024). AlphaBridge: tools for the
analysis of predicted biomolecular complexes. bioRxiv.
https://doi.org/10.1101/2024.10.23.619601
```

## License

Apache License 2.0 — see [LICENSE](../LICENSE).
