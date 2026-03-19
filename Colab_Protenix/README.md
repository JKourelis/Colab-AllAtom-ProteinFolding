# Protenix — Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1lGmcSPIFvhkfpJXUqjET-ikQzZWwi4D4)

Batch structure prediction with [Protenix](https://github.com/bytedance/Protenix) v1.0.4, ByteDance's open-source implementation inspired by AlphaFold3. Offers 8 model variants including Mini for lower VRAM.

## Features

- 8 model variants (Base/Mini x default/finetuned x v1.0.0/v0.0.1)
- Mini models run on T4 free tier (~6 GB VRAM for small complexes)
- Supports proteins, DNA, RNA, ligands (CCD + SMILES), ions, and modifications
- MSA via ColabFold server or `protenix mt` pipeline
- Pre-computed MSA support (from MSA Colab) — auto-downloads from Google Drive
- Full PAE (Predicted Aligned Error) matrices included in all predictions
- T4 auto-downgrades to fp32 when bf16 unavailable
- Built-in [ipSAE_batch](https://github.com/JKourelis/ipSAE_batch) interface analysis — ipSAE, ipTM, AlphaBridge, pDockQ scores + matrix plots, AlphaBridge-style ribbon diagrams, and interactive HTML comparisons. Also available as a [standalone Colab](https://colab.research.google.com/github/JKourelis/ipSAE_batch/blob/main/ipSAE_batch_colab.ipynb)

## GPU Requirements

| Model | Tokens | VRAM | Colab GPU |
|-------|--------|------|-----------|
| Mini | ~500 | ~6 GB | T4 (free) |
| Base | ~500 | ~6 GB | T4 (free) |
| Base | ~1000 | ~18 GB | L4/A100 |
| Base | ~2000 | ~67 GB | A100 80GB |

## Quick Start

1. Read Cell 0 for overview and CSV format
2. Run Cell 1: Restart kernel (installs PyTorch 2.7.1 + NumPy 2.4.1)
3. Run Cell 2: Install Protenix and dependencies
4. Run Cell 3: Initialize CSV processor
5. Run Cell 4: Upload CSV + connect Google Drive + resolve pre-computed MSAs
6. Run Cell 5: Configure prediction settings (MSA, model, precision)
7. Run Cell 6: Run predictions + download results

See the [main README](../README.md) for CSV format documentation and examples.

## Pre-computed MSAs

To use pre-computed MSAs from the MSA Colab notebook:

1. Run MSA Colab first (CPU runtime, no GPU needed)
2. In Cell 4 of this notebook, set `msa_folder` to the MSA output folder name (e.g., `MSA_Results`)
3. The notebook auto-downloads and unzips from Google Drive if needed
4. All-or-nothing per job: either all protein chains have pre-computed MSAs, or the entire job uses online search

When `msa_folder` is empty, Cell 5's online MSA settings (`use_msa`, `msa_server_mode`) control everything as usual.

## PAE Output

All predictions include full Predicted Aligned Error (PAE) matrices in `*_full_data_sample_*.json` files. These contain per-residue-pair confidence scores showing how well different parts of the structure are positioned relative to each other. PAE files are automatically included in the results ZIP alongside structure files (.cif).

## Performance Tips

**Parallel scheduling**: Jobs are automatically sorted largest-first for optimal calibration. The largest job calibrates VRAM usage, giving the most accurate per-token rate for parallel scheduling of remaining jobs.

## Known Limitations

- CCD ligand codes require `CCD_` prefix in the input JSON (e.g., `CCD_ATP` not `ATP`) — the CSV processor handles this automatically
- Requires PyTorch 2.7.1 and NumPy 2.4.1 (kernel restart handles this automatically)

## Citation

If you use this notebook, please cite:

**This notebook:**
```
Kourelis, J. (2026). Accessible batch biomolecular structure prediction
via Google Colab. [Manuscript in preparation]
```

**Protenix:**
```
Zhang, Y., Gong, C., Zhang, H., et al. (2026). Protenix-v1: Toward
High-Accuracy Open-Source Biomolecular Structure Prediction. bioRxiv.
https://doi.org/10.64898/2026.02.05.703733
```

**Foundational work:**
```
Abramson, J., et al. (2024). Accurate structure prediction of
biomolecular interactions with AlphaFold 3. Nature, 630, 493-500.
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
