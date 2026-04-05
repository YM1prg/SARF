# SARFM: Symbolic Arabic Root Fingerprint Morphology

# 📘 SARFM: Symbolic Arabic Root Fingerprint Morphology

> **A deterministic, rule-based morphological engine for Classical Arabic.**  
> Zero statistics. Zero neural networks. 100% traceable. Bidirectional encode/decode symmetry.

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![Deterministic](https://img.shields.io/badge/Architecture-Deterministic-orange.svg)](#)

---

## 🌍 Overview

**SARFM** (Symbolic Arabic Root Fingerprint Morphology) replaces statistical embeddings, corpus frequency bias, and probabilistic path selection with a **fully symbolic, mathematically transparent morphology pipeline**. 

It maps Arabic roots to fixed 40-dimensional semantic fingerprints extracted from classical lexicons using a formal decomposition protocol. These fingerprints modulate Finite-State Transducer (FST) transition weights via deterministic dot products, enforce hard grammatical constraints through context/pattern routers, and guarantee identical outputs across runs. Designed for academic rigor, linguistic auditability, and production reliability.

---

## ✨ Core Principles

| Principle | Implementation |
|-----------|----------------|
| **Determinism** | Same input + same dataset version → identical output. No `softmax`, no sampling, no weight drift. |
| **Lexicon-Grounded** | Fingerprints derived from classical dictionaries via a rule-based atomic lexicon protocol. |
| **Encode/Decode Symmetry** | Identical vector space and FST graph serve both directions; only traversal order and I/O mapping change. |
| **Hard Constraints** | Context tags & pattern matches route FST subgraphs deterministically. No fuzzy thresholds. |
| **Full Traceability** | Every decision logs exact lexical provenance, activated dimensions, and routing rationale. |

---

## 🏗️ Architecture

```
[Input: Root / Intent] 
   → Root Normalizer (canonicalization)
   → Semantic Index (O(1) hash lookup)
   → Fingerprint Retrieval (40-dim vector)
   → Transition Mask Modulation (vector • mask)
   → Context Router (pattern/tag → FST subgraph gating)
   → Quality Controller (PARTIAL/MULTI_SENSE fallbacks)
   → FST Traversal (weighted path selection)
   → [Output: Surface Form / Semantic Vector] + Trace Log
```

### 📦 Core Modules
| Module | Responsibility |
|--------|----------------|
| `RootNormalizer` | Deterministic canonicalization (strip tashkeel, unify alef/hamza, standardize weak finals) |
| `SemanticIndex` | O(1) hash-index mapping normalized roots → memory offsets & records |
| `TransitionMaskRegistry` | Static 40D sensitivity vectors pre-assigned to FST transitions |
| `ContextRouter` | Hard-wires `pattern_matches` & `context_tags` to FST activation/deactivation rules |
| `QualityController` | Applies fixed penalties for `PARTIAL` records; resolves `MULTI_SENSE` via precedence rules |
| `FST_Engine` | Weighted traversal with structural validation & tie-breaking |

---

## 🚀 Installation

```bash
# Clone repository
git clone https://github.com/yourorg/sarfm.git
cd sarfm

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install numpy  # Optional but recommended for vector operations
```

Place your compiled lexicon dataset at `data/sarfm_lexicon.json` (schema documented below).

---

## 💻 Quick Start

### 🔹 Encode Preparation (Python API)
```python
from sarfm.encoder import encode_prepare

# Prepare deterministic encode directives for root "تبل" in agricultural context
directives = encode_prepare(
    root="تبل",
    json_dataset_path="data/sarfm_lexicon.json",
    target_context="ENTITY_PLANT"
)

print(directives["status"])              # → "READY"
print(directives["active_fingerprint"])  # → 40-dim vector aligned with plant semantics
print(directives["routing_directive"])   # → activated transitions, constraints, offsets
print(directives["quality"]["trace_msg"])# → "MULTI_SENSE_FILTERED_BY_ENTITY_PLANT"
```

### 🔹 Command Line Interface
```bash
# Encode a root with explicit semantic context
python sarfm_cli.py encode --root "تاج" --context "ENTITY_OBJECT" --output directives.json

# Decode a surface word to semantic fingerprint
python sarfm_cli.py decode --word "التِّيجَانُ" --output fingerprint.json
```

---

## 📊 Dataset Specification

SARFM expects a JSON array of root records matching this schema:

```json
{
  "root": "تبل",
  "fingerprint_vector": [0.0, 0.33, 0.67, ..., 0.5],
  "validation_flags": ["MULTI_SENSE"],
  "sense_records": [
    {
      "sense_id": "ROOT_TBL_SENSE_PLANT",
      "primary_category": "ENTITY_PLANT",
      "vector_slice": [0.0, 1.0, 0.0, ..., 0.5],
      "confidence": "FULL",
      "provenance": "matched 'النّخْل'"
    }
  ],
  "pattern_matches": [{"pattern": "تَفَعَّلَ", "function": "CAUSATIVE_INTENSIVE"}],
  "context_tags": [{"context": "CONTEXT_AGRICULTURAL", "source_phrase": "نَخْلٌ مُتَبَتِّل"}]
}
```

### 🔑 Key Fields
| Field | Purpose |
|-------|---------|
| `fingerprint_vector` | 40-dimensional normalized semantic vector |
| `validation_flags` | `FULL`, `PARTIAL`, or `MULTI_SENSE` |
| `sense_records` | Disambiguated semantic branches with `vector_slice` |
| `pattern_matches` | Explicit morphological patterns triggering FST subgraphs |
| `context_tags` | Pragmatic/domain tags applying priority offsets |

---

## 🔐 Determinism & Auditing

SARFM guarantees mathematical transparency through:

1. **Immutable Vectors:** Fingerprints never change at runtime. Updates require versioned dataset releases.
2. **Fixed Arithmetic:** All weight modulation uses `base + (vector • mask)`. No dynamic learning, no thresholds beyond documented gates.
3. **Provenance Logging:** Every output includes:
   ```json
   "trace": {
     "selected_sense": "ROOT_TBL_SENSE_PLANT",
     "activated_dimensions": [1, 35, 39],
     "routing_reason": "MULTI_SENSE_FILTERED_BY_ENTITY_PLANT",
     "mask_modifiers": {"ROOT_SLOT_2": 0.82, "SUFFIX_NOUN_PL": 0.65}
   }
   ```
4. **Version Control:** Dataset, mask registry, and routing tables are version-tagged. Reproduction requires identical version hashes.

---

## 🗺️ Roadmap

| Phase | Milestone | Status |
|-------|-----------|--------|
| **v0.1** | Semantic Index, Mask Registry, Context Router, Quality Controller | ✅ Complete |
| **v0.2** | FST Traversal Engine (encode direction) |  In Development |
| **v0.3** | Decoder Module (surface → fingerprint reconstruction) | 📋 Planned |
| **v0.4** | Mask Population Toolkit (linguist UI for weight authoring) | 📋 Planned |
| **v1.0** | Full bidirectional pipeline, performance benchmarks, academic paper | 📋 Planned |

---

## 🤝 Contributing

SARFM is designed for linguistic researchers, computational morphologists, and NLP engineers who value transparency over black-box performance.

**How to contribute:**
- Add root records following the `protocol_v1.0` extraction guidelines
- Author transition sensitivity masks with classical references
- Submit routing rule extensions with documented precedence logic
- Improve normalization edge cases (dialectal variants, orthographic shifts)

All contributions must include deterministic test cases and provenance documentation.

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for details.

---

## 📚 Citation

If you use SARFM in academic research, please cite:
```bibtex
@misc{sarfm2026,
  author = {SARFM Contributors},
  title = {SARFM: Symbolic Arabic Root Fingerprint Morphology},
  year = {2026},
  howpublished = {\url{https://github.com/yourorg/sarfm}},
  note = {Deterministic morphological encoding/decoding for Classical Arabic}
}
```

---

> **SARFM does not guess. It computes.**  
> Built on centuries of Arabic linguistic scholarship. Engineered for mathematical transparency. Ready for the next generation of transparent NLP.
