# alignment-to-microreact

Build a Microreact project from an aligned FASTA file and a Newick tree.

The script:
- extracts variable positions from the alignment
- writes a CSV with one column per variable position
- creates a `.microreact` project file
- can upload the project directly to Microreact with an API key
- can optionally add a second linked metadata table
- can optionally number positions relative to a named reference sequence

By default, the script writes both local output files:
- `variant_positions.csv`
- `project.microreact`

If you do not pass `--skip-upload`, it then also tries to create the Microreact project via API.

## Files In This Repo

Example inputs:
- `SHV_protein_alignment.fas`
- `gene_SHV_aligned.tree`
- `SHV_ncbirefgenes_noBla.tsv`

Example reference/example outputs for inspection:
- `3panel_example.microreact`
- `SHV tree.microreact`

Main script:
- `build_microreact_from_alignment.py`

## Requirements

- Python 3
- a Microreact access key if you want to upload directly

## API Key Setup

If you want the script to create the Microreact project directly, export your key first:

```bash
export MICROREACT_ACCESS_KEY='your-key-here'
```

If you do not want to upload yet, add `--skip-upload` and the script will stop after writing the CSV and `.microreact` file locally.

## Basic Example

Create the variant CSV and `.microreact` file locally:

```bash
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "SHV variants" \
  --skip-upload
```

## Upload Directly To Microreact

```bash
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "SHV variants"
```

## Reference-Based Coordinates

Use a named sequence in the alignment as the reference. Alignment columns where the reference has a gap are removed, and remaining positions are numbered by reference coordinate.

For the included SHV example, use `SHV-1`:

```bash
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "SHV variants (SHV-1 reference)" \
  --reference-id SHV-1 \
  --skip-upload
```

If you want reference-gap columns to be retained by folding them into a nearby reference position, add `--compress-inserts`. Insert runs are merged into the preceding retained reference position, except for leading inserts before the first reference base, which are merged into the first retained position. For example, if the aligned values are `A,B,C,D,E,F` and positions `3` and `4` are insert columns relative to the reference, `--compress-inserts` produces `A,BCD,E,F` instead of `A,B,E,F`.

`--compress-inserts` only works together with `--reference-id`, because insert columns are defined relative to a named reference sequence.

```bash
# valid: compress inserts while numbering columns against SHV-1
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "SHV variants (compressed inserts)" \
  --reference-id SHV-1 \
  --compress-inserts \
  --skip-upload
```

```bash
# invalid: this exits with "--compress-inserts requires --reference-id"
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "Invalid compressed inserts example" \
  --compress-inserts \
  --skip-upload
```

## Add A Second Linked Metadata Table

The optional second table is linked to the main variant table. By default the primary field is `id` and the secondary field is `#Allele`.

Example with the included TSV:

```bash
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "SHV variants with metadata" \
  --secondary-table SHV_ncbirefgenes_noBla.tsv \
  --skip-upload
```

If your link columns are different, override them:

```bash
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "Custom linked metadata" \
  --secondary-table SHV_ncbirefgenes_noBla.tsv \
  --secondary-master-field id \
  --secondary-link-field '#Allele' \
  --skip-upload
```

## Full Example

Reference-based positions plus linked metadata, with direct upload:

```bash
python3 build_microreact_from_alignment.py \
  SHV_protein_alignment.fas \
  gene_SHV_aligned.tree \
  --project-name "SHV variants with metadata (SHV-1 reference)" \
  --reference-id SHV-1 \
  --secondary-table SHV_ncbirefgenes_noBla.tsv
```

## Help

```bash
python3 build_microreact_from_alignment.py --help
```
