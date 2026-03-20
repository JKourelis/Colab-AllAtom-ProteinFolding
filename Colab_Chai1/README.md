# Chai-1 — Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1vkzrziFd8t4PIf2HFzpiDCnUu9ytU9PP)

Batch structure prediction with [Chai-1](https://github.com/chaidiscovery/chai-lab) v0.6.1 (Chai Discovery), a model with strong single-sequence performance and ESM embedding support.

## Features

- Strong single-sequence mode (no MSA needed for good results)
- ESM protein language model embeddings
- Supports proteins, DNA, RNA, ligands (SMILES), ions, and glycans
- MSA via ColabFold server (optional, improves multi-chain quality)
- Automatic CCD-to-SMILES conversion for ligands
- Built-in [ipSAE_batch](https://github.com/JKourelis/ipSAE_batch) interface analysis — ipSAE, ipTM, AlphaBridge, pDockQ scores + matrix plots, AlphaBridge-style ribbon diagrams, and interactive HTML comparisons. Also available as a [standalone Colab](https://colab.research.google.com/github/JKourelis/ipSAE_batch/blob/main/ipSAE_batch_colab.ipynb)

## GPU Requirements

**Chai-1 requires bfloat16 (compute capability >= 8.0). T4 GPUs cannot run this model.**

| GPU | VRAM | Compatible | Notes |
|-----|------|------------|-------|
| T4 (free) | 15 GB | **No** | Compute capability 7.5, no bf16 |
| L4 (paid) | 24 GB | Yes | Small complexes only |
| A100 (Pro) | 40 GB | Yes | Recommended |
| H100 | 80 GB | Yes | |
| Blackwell (G4) | 24–192 GB | Yes | torch 2.7.1+cu128 (auto-installed) |

## Quick Start

1. Read Cell 0 for overview and CSV format
2. Run Cell 1: Restart kernel (installs NumPy 1.x + PyTorch 2.7.x)
3. Run Cell 2: Install Chai-1 and dependencies
4. Run Cell 3: Initialize CSV processor
5. Run Cell 4: Upload CSV + connect Google Drive
6. Run Cell 5: Configure prediction settings
7. Run Cell 6: Run predictions + download results

See the [main README](../README.md) for CSV format documentation and examples.

## Performance Tips

**Parallel scheduling**: Jobs are automatically sorted largest-first for optimal calibration. The largest job calibrates VRAM usage, giving the most accurate per-token rate for parallel scheduling of remaining jobs.

## Known Limitations

- **bf16 only** — will not run on T4 GPUs (detected and aborted automatically)
- Always outputs mmCIF format (no PDB option)
- Max 2048 tokens per complex
- Model weights (~6 GB) download on first run

## Citation

If you use this notebook, please cite:

**This notebook:**
```
Kourelis, J. (2026). Accessible batch biomolecular structure prediction
via Google Colab. [Manuscript in preparation]
```

**Chai-1:**
```
Chai Discovery (2025). Zero-shot antibody design in a 24-well plate.
bioRxiv. https://doi.org/10.1101/2025.07.05.663018
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
