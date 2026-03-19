# MSA Colab - Pre-computed MSA Search, AFDB Templates & PDB Hits

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1kUEp55gbRm_9zFBQN4FQUaeaxdFh04BK)

**STATUS: IMPLEMENTED (Mar 2026) / PRIVATE**

Standalone Colab notebook for pre-computing Multiple Sequence Alignments (MSAs) using the ColabFold MMseqs2 API, plus optional AFDB structural template extraction and best PDB hit downloading. Outputs `.a3m` files that are directly consumed by folding tool notebooks (IntelliFold, Boltz-2, Protenix, Chai-1, OpenFold3).

**No GPU required** -- this notebook runs on CPU. All computation is MSA search (network I/O), template/PDB download, and file processing (CPU-only).

## Why

1. **Speed**: MSA search takes 30-120s per protein. Pre-compute once, reuse across all tools and re-runs.
2. **AFDB Templates**: Automatically extract high-quality AlphaFold DB structural templates for each protein chain.
3. **PDB Hits**: Download the best experimental PDB structure (>30% identity) for each protein chain.
4. **Consistency**: All 5 folding tools get identical MSAs and structural references, enabling fair comparison.

## Notebook Structure (6 cells)

| Cell | Title | Content |
|------|-------|---------|
| 0 | Markdown header | Overview, workflow, CSV format, output structure |
| 1 | Install Dependencies | `biopython==1.85`, `pydrive2==1.21.3` (no kernel restart) |
| 2 | MSA Processor Setup | `MSAProcessor` class: CSV parser, ColabFold API client, AFDB templates, PDB search |
| 3 | Upload CSV + GDrive | File upload, CSV processing, Google Drive connection |
| 4 | Settings | MSA search params, template/PDB extraction toggle |
| 5 | Run + Download | Phase 1 (MSA search) + Phase 1.5 (AFDB templates) + Phase 1.6 (PDB hits) + Phase 2 (paired) + GDrive upload |

## Workflow

1. Upload CSV (same unified format as all folding tools)
2. Configure MSA search settings and template/PDB extraction
3. Run: searches MSAs, extracts AFDB templates, downloads PDB hits, writes output
4. Download results from Google Drive

## CSV Input Format

Same columns as all folding notebooks. Only `protein` type sequences are used for MSA search (DNA, RNA, ligands are ignored).

```
jobname, seq1_name, seq1_type, seq1, seq1_copies, seq1_mods, seq2_name, seq2_type, seq2, ...
```

## Output Structure

```
MSA_Results/
  individual/              # Per-protein full MSAs (reusable across jobs)
    proteinA.a3m
    proteinB.a3m
  paired/                  # Per-job query-only MSAs (for pairedMsaPath)
    job1_proteinA_paired.a3m     # Query sequence only (2 lines)
    job1_proteinB_paired.a3m     # Query sequence only (2 lines)
  templates/               # Per-chain structural data (optional)
    proteinA/
      AFDB/
        P12345.cif               # AFDB predicted structure (original name)
        proteinA_templates.a3m   # Template alignment file
      PDB/
        1ABC.cif                 # Best PDB hit (original name)
        2DEF.cif                 # Second hit (if available)
    proteinB/
      AFDB/
        Q67890.cif
        proteinB_templates.a3m
      PDB/
        3GHI.cif
  metadata.json            # Search stats, template/PDB decisions, timing
```

### paired/ directory

The `paired/` files contain **query sequence only** (2 lines: header + sequence). This matches the local pipeline behavior. Folding tools use:
- `pairedMsaPath` -> `paired/` file (query-only, structural context)
- `unpairedMsaPath` -> `individual/` file (full MSA, evolutionary context)

## AFDB Templates

When `extract_templates` is enabled:

1. Parses UniProt accessions from MSA headers (ColabFold UniRef100 IDs)
2. Downloads candidate AFDB structures (cached for reuse)
3. Scores by pLDDT in the aligned region
4. Selects the best template using tiered selection:
   - **Tier 1**: 100% identity + pLDDT >= 85 (instant accept)
   - **Tier 2**: >95% identity, highest pLDDT (>= 70)
   - **Tier 3**: Best overall pLDDT among all qualifying hits (>= 30% identity)
5. Writes Protenix-format template `.a3m` files to `templates/AFDB/`

**All-or-nothing**: AFDB templates are used for a job only if ALL protein chains have a template. Per-job decisions are recorded in `metadata.json`.

## PDB Hits

When `extract_templates` is enabled, the notebook also searches RCSB PDB for the best experimental structure matching each protein (>30% identity):

1. Queries RCSB PDB Search API (MMseqs2 backend) for each protein sequence
2. Downloads the best-scoring hit in mmCIF format
3. Fetches metadata (resolution, title, release date) from RCSB Data API
4. Saves structures to `templates/PDB/`

PDB hits complement AFDB templates: AFDB provides predicted structures for any protein with a UniProt entry, while PDB provides experimentally determined structures (X-ray, cryo-EM, NMR).

## Settings Reference

| Setting | Options | Effect |
|---------|---------|--------|
| `msa_server_url` | URL (default: `https://api.colabfold.com`) | ColabFold API endpoint |
| `msa_database` | `UniRef+EnvDB` / `UniRef only` | Broader (recommended) vs faster search |
| `pair_mode` | `unpaired_paired` / `paired` / `unpaired` | What ColabFold server returns |
| `max_msa` | `auto` / `512:1024` / etc. | Truncate MSA depth to save memory in folding |
| `rate_limit_delay` | 1-30s (default 5) | Delay between API requests (rate limiting) |
| `extract_templates` | True / False | Download AFDB templates and best PDB hits |

## Using Results with Folding Notebooks

1. Run this notebook -- results upload to Google Drive automatically
2. In any folding notebook's **Cell 4** (Upload CSV), set `msa_folder` to the output folder
   name (e.g., `MSA_Results`). The notebook auto-downloads and unzips from Google Drive.
3. The folding notebook matches files by name: `{jobname}_{seq_name}_paired.a3m`
   - Name matching is case-insensitive and hyphen/underscore-tolerant
   - All-or-nothing per job: either all protein chains have MSAs, or the whole job uses online search

**Example**: If your CSV has `jobname=Rx-ADP, seq1_name=Rx, seq2_name=ADP`,
the paired MSA files will be `Rx-ADP_Rx_paired.a3m` (protein chains only -- ligands are skipped).

## Per-Tool Consumption

| Tool | MSA Mechanism | Template/PDB Mechanism |
|------|--------------|----------------------|
| IntelliFold | `msa:` field in YAML per chain | N/A |
| Boltz-2 | `msa:` field / `.csv` format | N/A |
| Protenix | `pairedMsaPath` + `unpairedMsaPath` in JSON | MSA only (templates disabled) |
| Chai-1 | `--msa-directory` CLI flag | N/A |
| OpenFold3 | `paired_msa_file_paths` in JSON | N/A |

## File Naming

- By protein name from CSV (e.g., `receptorX.a3m`)
- No name provided: `SHA256(sequence.upper()).a3m`
- Duplicate names for different sequences: `name_HASH[:8].a3m`

## Citation

If you use this notebook, please cite:

**This notebook:**
```
Kourelis, J. (2026). Accessible batch biomolecular structure prediction
via Google Colab. [Manuscript in preparation]
```

**ColabFold MSA search:**
```
Mirdita, M., Schutze, K., Moriwaki, Y., Heo, L., Ovchinnikov, S. &
Steinegger, M. (2022). ColabFold: making protein folding accessible
to all. Nature Methods, 19(6), 679-682.
```

## License

Private / not for public release.
