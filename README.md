# GenMask

Generate novel molecule and protein analogues by masking part of a sequence and
unmasking it with a Hugging Face masked-language model.

Given a SMILES string or an amino-acid sequence, GenMask masks a fraction of its
tokens/residues, then fills them back in most-confident-position first, keeping every
candidate whose probability clears a cutoff (not just the single best guess). Repeating
this with different masking patterns (start, end, and several random draws) produces a
diverse set of plausible analogues in one run.

## Contents

- **Small molecules** — mask/unmask SMILES tokens with a BERT-style model, score
  results with QED (drug-likeness) and Tanimoto similarity to the parent molecule.
- **Proteins** — mask/unmask residues with an ESM2 model, score results with percent
  sequence identity to the parent. Sequences can come from a raw string or be pulled
  directly from a PDB entry and chain.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

export HF_TOKEN=your_huggingface_token   # required for gated/private HF models
```

## Usage

### Small molecules (SMILES)

```bash
python code/gen_mask_cli.py --smiles "CC(=O)Oc1ccccc1C(=O)O"
python code/gen_mask_cli.py --csv molecules.csv
```

The CSV mode looks for a `smiles`/`SMILES`/`Smiles` column (case-insensitive).

| Flag | Default | Description |
|---|---|---|
| `--smiles` | — | A single SMILES string |
| `--csv` | — | CSV file with a SMILES column |
| `--percent` | `0.10` | Fraction of tokens masked per pass |
| `--model` | `cafierom/bert-base-cased-ChemTok-ZN250K-V1` | HF model name |
| `--out` | `outputs/results.csv` | Output CSV path |
| `--no-image` | off | Skip saving a grid-image PNG per input molecule |
| `--no-tanimoto` | off | Skip the `tanimoto_to_parent` similarity column |

Output columns: `input_smiles`, `analogue_smiles`, `entropy`, `qed`, `tanimoto_to_parent`.

### Proteins

```bash
python code/protein_mask_cli.py --sequence "MKTAYIAKQRQISFVKSHFSRQLEERLGLIEV..."
python code/protein_mask_cli.py --pdb 4ZGM --chain B
python code/protein_mask_cli.py --csv sequences.csv
```

The CSV mode looks for a `sequence`/`Sequence`/`SEQUENCE` column (case-insensitive).

| Flag | Default | Description |
|---|---|---|
| `--sequence` | — | A single amino-acid sequence |
| `--csv` | — | CSV file with a sequence column |
| `--pdb` | — | PDB ID to fetch a chain's sequence from (use with `--chain`) |
| `--chain` | — | Chain letter to extract, e.g. `B` (required with `--pdb`) |
| `--percent` | `0.10` | Fraction of residues masked per pass |
| `--prob-cutoff` | `0.05` | Minimum candidate probability to keep a beam |
| `--model` | `facebook/esm2_t12_35M_UR50D` | HF ESM checkpoint |
| `--out` | `outputs/protein_results.csv` | Output CSV path |

Output columns: `input_sequence`, `generated_sequence`, `entropy`, `percent_identity_to_parent`.

## Repository layout

```
code/       library modules and CLI entry points
outputs/    generated CSVs and images (not tracked in git)
archive/    earlier exploratory notebooks
```

## Models

- **SMILES**: `cafierom/bert-base-cased-ChemTok-ZN250K-V1`, a BERT model trained on
  tokenized SMILES strings.
- **Proteins**: any Hugging Face ESM2 checkpoint (e.g. `facebook/esm2_t12_35M_UR50D`
  through `facebook/esm2_t33_650M_UR50D`) — larger checkpoints give better predictions
  at the cost of slower runs and bigger downloads.
