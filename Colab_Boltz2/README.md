# Boltz-2 — Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1maLULdT_lyid0-tRvrdldrSsZaiXpIuL)

Batch structure and binding affinity prediction with [Boltz-2](https://github.com/jwohlwend/boltz) v2.2.1. Predicts protein, DNA, RNA, and ligand complexes with optional Kd/Ki affinity estimation.

## Features

- Structure prediction + binding affinity (Kd/Ki) in one model
- Supports proteins, DNA, RNA, ligands, ions, and modifications (PTMs, glycans)
- Pocket restraints and covalent bond constraints
- cuEquivariance kernel acceleration (auto-detected, fallback if unavailable)
- MSA via ColabFold server with configurable pairing strategy
- PAE/PDE confidence matrices (optional)
- Built-in [ipSAE_batch](https://github.com/JKourelis/ipSAE_batch) interface analysis — ipSAE, ipTM, AlphaBridge, pDockQ scores + matrix plots, AlphaBridge-style ribbon diagrams, and interactive HTML comparisons. Also available as a [standalone Colab](https://colab.research.google.com/github/JKourelis/ipSAE_batch/blob/main/ipSAE_batch_colab.ipynb)

## GPU Requirements

| Colab GPU | VRAM | Compatible |
|-----------|------|------------|
| T4 (free) | 15 GB | Yes |
| L4 (paid) | 24 GB | Yes |
| A100 (Pro) | 40 GB | Yes |

## Quick Start

1. Read Cell 0 for overview and CSV format
2. Run Cell 1: Restart kernel (installs NumPy 1.x)
3. Run Cell 2: Install Boltz-2 and dependencies
4. Run Cell 3: Initialize CSV processor
5. Run Cell 4: Upload CSV + connect Google Drive
6. Run Cell 5: Configure prediction settings
7. Run Cell 6: Run predictions + download results

See the [main README](../README.md) for CSV format documentation and examples.

## Performance Tips

**Parallel scheduling**: Jobs are automatically sorted largest-first for optimal calibration. The largest job calibrates VRAM usage, giving the most accurate per-token rate for parallel scheduling of remaining jobs.

## Known Limitations

- cuEquivariance kernels may fail on some GPU configurations (auto-detected with `--no_kernels` fallback)
- Requires NumPy <2.0 (kernel restart handles this automatically)

## Citation

If you use this notebook, please cite:

**This notebook:**
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
