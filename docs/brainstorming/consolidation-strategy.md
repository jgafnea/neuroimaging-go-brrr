# Consolidation Strategy: neuroimaging-go-brrrr

> **Status**: Draft / Open for Discussion
> **Date**: 2024-12-05
> **Contributors**: Open to all

## Overview

This document outlines how multiple related neuroimaging repositories could be consolidated under `neuroimaging-go-brrrr` to reduce fragmentation and enable better collaboration within the HuggingFace Science community.

## Current Ecosystem

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     neuroimaging-go-brrrr                                │
│                     ═══════════════════════                              │
│                     CONSOLIDATION HUB                                    │
│                                                                          │
│  Current contents:                                                       │
│  ├── scripts/push_to_hub_ds004884_full.py  ← Uses native BIDS loader    │
│  ├── tools/bids-neuroimaging/              ← HF Space visualization     │
│  └── README.md                             ← Links to resources         │
└─────────────────────────────────────────────────────────────────────────┘
                                    ▲
                                    │ (potential consolidation)
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐
│ arc-aphasia-bids │  │ stroke-deepisles │  │ huggingface/datasets     │
│                  │  │ -demo            │  │ (BIDS loader PR)         │
│                  │  │                  │  │                          │
│ • Upload ARC     │  │ • DeepISLES      │  │ • NATIVE BIDS LOADER     │
│   to HF Hub      │  │   inference      │  │   load_dataset('bids')   │
│ • BIDS→HF        │  │ • Gradio UI      │  │ • Nifti() feature type   │
│   conversion     │  │ • Full test      │  │ • NiiVue integration     │
│ • Validation     │  │   suite          │  │                          │
└──────────────────┘  └──────────────────┘  └──────────────────────────┘
                                │
                                ▼
                      ┌──────────────────┐
                      │ DeepIsles        │
                      │ (External Ref)   │
                      │                  │
                      │ • Nature paper   │
                      │ • SEALS+NVAUTO   │
                      │   +FACTORIZER    │
                      │ • Docker image   │
                      └──────────────────┘
```

## Related Repositories

| Repository | Maintainer | Purpose | Link |
|------------|------------|---------|------|
| neuroimaging-go-brrrr | @CloseChoice | Consolidation hub for neuroimaging tools | [GitHub](https://github.com/CloseChoice/neuroimaging-go-brrrr) |
| arc-aphasia-bids | @The-Obstacle-Is-The-Way | Upload ARC dataset (ds004884) to HuggingFace Hub | [GitHub](https://github.com/The-Obstacle-Is-The-Way/arc-aphasia-bids) |
| stroke-deepisles-demo | @The-Obstacle-Is-The-Way | DeepISLES inference demo with Gradio UI | [GitHub](https://github.com/The-Obstacle-Is-The-Way/stroke-deepisles-demo) |
| datasets (BIDS loader PR) | @CloseChoice | Native BIDS loader for HuggingFace datasets library | [GitHub](https://github.com/CloseChoice/datasets/tree/feat/bids-loader-streaming-upload-fix) |
| DeepIsles | @ezequieldlrosa | State-of-the-art stroke lesion segmentation (Nature 2025) | [GitHub](https://github.com/ezequieldlrosa/DeepIsles) |

## HuggingFace Resources

| Resource | Description | Link |
|----------|-------------|------|
| arc-aphasia-bids dataset | ARC dataset on HF Hub (293 GB, 902 sessions) | [HF Hub](https://huggingface.co/datasets/hugging-science/arc-aphasia-bids) |
| bids-neuroimaging Space | NiiVue visualization demo | [HF Spaces](https://huggingface.co/spaces/TobiasPitters/bids-neuroimaging) |

## Key Technical Components

### 1. Native BIDS Loader (datasets library PR)

The `feat/bids-loader-streaming-upload-fix` branch adds native BIDS support to HuggingFace's `datasets` library:

```python
from datasets import load_dataset

# Load any BIDS dataset with one line
ds = load_dataset('bids', data_dir="path/to/bids/dataset", streaming=True)

# Push to Hub
ds.push_to_hub("org/dataset-name", num_shards={"train": 500})
```

Features:
- Uses `pybids` for BIDS-compliant file discovery
- `Nifti()` feature type for NIfTI file handling
- `Nifti1ImageWrapper._repr_html_()` for NiiVue visualization in notebooks
- Streaming support for large datasets

### 2. arc-aphasia-bids

Custom pipeline for the Aphasia Recovery Cohort (ARC) dataset:
- Converts OpenNeuro ds004884 (BIDS) to HuggingFace Dataset format
- Validation against the Scientific Data paper (230 subjects, 902 sessions)
- Session-level sharding for memory efficiency

### 3. stroke-deepisles-demo

End-to-end inference demo:
- Loads BIDS/HF neuroimaging data
- Runs DeepISLES Docker container for stroke lesion segmentation
- Gradio UI for interactive visualization
- Comprehensive test suite (96 tests, 82% coverage)

## Consolidation Options

### Option A: Modular Monorepo

Migrate all code into `neuroimaging-go-brrrr` as packages:

```
neuroimaging-go-brrrr/
├── packages/
│   ├── arc-bids/              ← arc-aphasia-bids code
│   ├── stroke-demo/           ← stroke-deepisles-demo code
│   └── bids-loader-examples/  ← Examples using native BIDS loader
├── scripts/
├── tools/
├── docs/
└── pyproject.toml             ← Workspace configuration
```

**Pros:**
- Single source of truth
- Easier cross-package development
- Unified CI/CD

**Cons:**
- Larger repository size
- More complex release management
- Migration effort required

### Option B: Hub with Linked Repositories

Keep repositories separate, use `neuroimaging-go-brrrr` as documentation/coordination hub:

```
neuroimaging-go-brrrr/
├── docs/
│   ├── architecture.md        ← How repos relate
│   ├── getting-started.md     ← Quick start guide
│   └── contributing.md        ← Contribution guidelines
├── scripts/                   ← Shared utility scripts
├── tools/                     ← Shared tools
└── README.md                  ← Links to all repos
```

**Pros:**
- Minimal migration effort
- Repos remain independently maintainable
- Clear separation of concerns

**Cons:**
- Coordination overhead
- Potential for divergence
- Cross-repo changes require multiple PRs

### Option C: Wait for Upstream Merge

Wait for the BIDS loader PR to merge into `huggingface/datasets`, then reassess:

- Once merged, `arc-aphasia-bids` simplifies to example usage
- Focus consolidation on inference/demo tooling
- Reduce duplication with upstream library

**Pros:**
- Leverages official HuggingFace support
- Reduces maintenance burden
- Community benefits from upstream contribution

**Cons:**
- Dependent on upstream merge timeline
- May need interim solutions

## Open Questions

1. **Monorepo vs. linked repos?** What structure best serves the community?

2. **BIDS loader PR timeline?** When is the `feat/bids-loader-streaming-upload-fix` expected to merge to `huggingface/datasets`?

3. **Demo deployment?** Should `stroke-deepisles-demo` be deployed as a HuggingFace Space alongside `bids-neuroimaging`?

4. **Shared infrastructure?** What CI/CD, testing, or documentation infrastructure should be shared?

5. **Additional datasets?** What other OpenNeuro datasets should be prioritized for HuggingFace upload?
   - ds004889 (acute stroke) is mentioned in README.md

## Next Steps

- [ ] Discuss consolidation approach in GitHub Issues
- [ ] Decide on monorepo vs. linked repos structure
- [ ] Create issues for specific integration tasks
- [ ] Document contribution workflow for new collaborators

## References

- [OpenNeuro ds004884 (ARC)](https://openneuro.org/datasets/ds004884) - Aphasia Recovery Cohort
- [OpenNeuro ds004889](https://openneuro.org/datasets/ds004889) - Acute stroke dataset
- [DeepISLES Nature Paper](https://www.nature.com/articles/s41467-025-62373-x) - Stroke lesion segmentation
- [ISLES'22 Challenge](https://isles22.grand-challenge.org/) - Ischemic Stroke Lesion Segmentation
- [HuggingFace Datasets](https://huggingface.co/docs/datasets/) - Dataset library documentation
- [BIDS Specification](https://bids-specification.readthedocs.io/) - Brain Imaging Data Structure
