# IntelliFold — Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1VNPsmVu45b1xf5jvAE0N4c38HZb0w74_)

Batch structure prediction with [IntelliFold](https://github.com/IntelliGen-AI/IntelliFold) v2.0.0, a controllable foundation model achieving AF3-level accuracy on biomolecular complexes.

## Features

- v2 and v2-flash model variants (flash trades accuracy for speed)
- Supports proteins, DNA, RNA, ligands, ions, and modifications (PTMs, glycans)
- Pocket restraints and covalent bond constraints
- Fast LayerNorm option for reduced VRAM
- MSA via ColabFold server or single-sequence mode
- fp16, bf16, and fp32 precision options
- Built-in [ipSAE_batch](https://github.com/JKourelis/ipSAE_batch) interface analysis — ipSAE, ipTM, AlphaBridge, pDockQ scores + matrix plots, AlphaBridge-style ribbon diagrams, and interactive HTML comparisons. Also available as a [standalone Colab](https://colab.research.google.com/github/JKourelis/ipSAE_batch/blob/main/ipSAE_batch_colab.ipynb)

## GPU Requirements

| Colab GPU | VRAM | Compatible |
|-----------|------|------------|
| T4 (free) | 15 GB | Yes |
| L4 (paid) | 24 GB | Yes |
| A100 (Pro) | 40 GB | Yes |

## Quick Start

1. Read Cell 0 for overview and CSV format
2. Run Cell 1: Restart kernel (installs NumPy 1.x + PyTorch 2.6)
3. Run Cell 2: Install IntelliFold and dependencies
4. Run Cell 3: Initialize CSV processor
5. Run Cell 4: Upload CSV + connect Google Drive
6. Run Cell 5: Configure prediction settings
7. Run Cell 6: Run predictions + download results

See the [main README](../README.md) for CSV format documentation and examples.

## Performance Tips

**Parallel scheduling**: Jobs are automatically sorted largest-first for optimal calibration. The largest job calibrates VRAM usage, giving the most accurate per-token rate for parallel scheduling of remaining jobs.

## Known Limitations

- v2.0.0 has an rdkit CCD parsing bug (`Boost.Python.ArgumentError`) with some ligand codes — use SMILES as a workaround
- Requires NumPy 1.x and PyTorch 2.6.0 (kernel restart handles this automatically)

## Citation

If you use this notebook, please cite:

**This notebook:**
```
Kourelis, J. (2026). Accessible batch biomolecular structure prediction
via Google Colab. [Manuscript in preparation]
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
