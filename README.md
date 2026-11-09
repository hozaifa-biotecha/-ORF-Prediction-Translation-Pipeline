# 🧬 ORF Prediction & Translation Pipeline

A bioinformatics workflow that starts from a **GENCODE transcript sequence**, predicts its **Open Reading Frames (ORFs)** with several independent tools, translates it in all six reading frames, and compares the results to identify the true protein-coding frame.

> 🎥 A screen-recorded walkthrough is available in the demo video: `ORF_Prediction___Translate.mp4` *(add the link or file here)*.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Workflow](#-workflow)
- [Tools & Data](#-tools--data)
- [Step-by-Step](#-step-by-step)
- [Results](#-results)
- [How to Reproduce](#-how-to-reproduce)
- [Limitations & Future Work](#-limitations--future-work)
- [Author](#-author)

---

## 🔎 Overview

An **Open Reading Frame (ORF)** is a stretch of DNA/RNA that starts with a start codon (usually `ATG`) and continues, codon by codon, until a stop codon (`TAA`, `TAG`, or `TGA`). Because a nucleotide sequence can be read in **six frames** (3 on the forward strand: +1, +2, +3, and 3 on the reverse strand: −1, −2, −3), finding the real protein-coding ORF requires scanning and translating all of them.

This project:

1. Extracts a transcript from the GENCODE protein-coding transcripts FASTA file.
2. Predicts ORFs with **NCBI ORFfinder**.
3. Translates the sequence with **ExPASy Translate** and **EMBOSS Transeq**.
4. Counts and visualizes ORFs in every frame with **EMBOSS Sixpack**.
5. Concludes which ORF/frame is the main one.

---

## 🔁 Workflow

```
GENCODE FASTA (protein-coding transcripts)
          │
          ▼
Terminal: select & view the target transcript
          │
          ├──► NCBI ORFfinder ─────── ORF prediction
          │
          ├──► ExPASy Translate ───── 6-frame translation, ORFs highlighted
          │
          ├──► EMBOSS Transeq ─────── protein sequence of a chosen frame
          │
          └──► EMBOSS Sixpack ─────── ORF counts per frame + 6-frame map
                      │
                      ▼
          Final answer: ORF 1, Frame +1
```

---

## 🛠 Tools & Data

| Category | Details |
|---|---|
| Environment | Ubuntu (WSL) terminal, Python virtual environment `bio_env` |
| Input data | GENCODE protein-coding transcripts FASTA (`gencode.v50.pc_transcripts.fa`) |
| ORF prediction | [NCBI ORFfinder](https://www.ncbi.nlm.nih.gov/orffinder/) |
| Translation | [ExPASy Translate](https://web.expasy.org/translate/), [EMBOSS Transeq](https://www.ebi.ac.uk/jdispatcher/st/emboss_transeq) |
| 6-frame analysis | [EMBOSS Sixpack](https://www.ebi.ac.uk/jdispatcher/st/emboss_sixpack) |

### Target transcript

The FASTA header of the analyzed transcript (GENCODE format, fields separated by `|`):

| Field | Value |
|---|---|
| Transcript ID | `ENST00000114009.1` |
| Gene ID | `ENSG00000187634.15` |
| Gene name | `SAMD11` (transcript `SAMD11-220`) |
| Transcript length | 2406 nt |
| 5′ UTR | 1–537 |
| CDS (annotated) | 538–1383 |
| 3′ UTR | 1384–2406 |

---

## 🧭 Step-by-Step

### 1. Prepare the environment and view the transcript

```bash
cd gencode_project/
source bio_env/bin/activate
```

The transcript's FASTA record was extracted from the GENCODE file and inspected in the terminal.

### 2. Predict ORFs with NCBI ORFfinder

The transcript sequence (FASTA format) was pasted into ORFfinder to list the candidate ORFs, their frames and positions.

### 3. Translate with ExPASy Translate

The sequence was translated with the **Standard genetic code**, on both strands (forward and reverse), using the *Compact* output format. Open reading frames are highlighted in the output so they can be compared across frames (5′3′ Frame 1 to 3 and 3′5′ Frame 1 to 3).

### 4. Get the protein sequence with EMBOSS Transeq

EMBOSS Transeq was run with **Frame = 1** and the **Standard code** to obtain the amino-acid sequence of the selected frame.

### 5. Map and count ORFs with EMBOSS Sixpack

EMBOSS Sixpack displays the DNA sequence together with the six translations and counts the ORFs in each frame (minimum ORF size set to 1).

---

## 📊 Results

### ORFs per frame (EMBOSS Sixpack, minimum ORF size = 1)

| Frame | Total ORFs |
|---|---|
| Frame 1 | 8 |
| Frame 2 | 26 |
| Frame 3 | 15 |
| Frame 4 | 7 |
| Frame 5 | 11 |
| Frame 6 | 14 |
| **Total** | **81** |

> With a minimum size of 1, even tiny stop-to-stop segments are counted, which is why the number of ORFs is high. In practice a minimum length (for example 75–300 nt) is used to filter out spurious ORFs.

### Conclusion

**Our ORF is: ORF 1, Frame +1.**

This agrees with the annotation in the header: the CDS starts at position 538, and since (538 − 1) is divisible by 3, the CDS begins in the first forward frame (+1).

---

## ▶️ How to Reproduce

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# 2. Create and activate a virtual environment
python3 -m venv bio_env
source bio_env/bin/activate

# 3. Get a transcript FASTA from GENCODE (https://www.gencodegenes.org/)
#    and save the record of interest as transcript.fasta
```

Then:

1. Paste `transcript.fasta` into [NCBI ORFfinder](https://www.ncbi.nlm.nih.gov/orffinder/) and run it.
2. Paste the sequence into [ExPASy Translate](https://web.expasy.org/translate/) (Standard code, forward and reverse strands).
3. Run [EMBOSS Transeq](https://www.ebi.ac.uk/jdispatcher/st/emboss_transeq) (Frame = 1, Standard code).
4. Run [EMBOSS Sixpack](https://www.ebi.ac.uk/jdispatcher/st/emboss_sixpack) to get the ORF count per frame.

---

## 🚧 Limitations & Future Work

- The analysis uses web tools; the same steps can be automated in Python with Biopython (`Seq.translate()` over 6 frames) or with the EMBOSS command-line tools.
- Add a minimum ORF length filter and report the longest ORF per frame.
- Validate the predicted protein with BLASTP or a domain search (for example Pfam/InterPro).
- Planned improvements:
  - [ ] Automate 6-frame translation and ORF detection in a Python script
  - [ ] Plot ORF lengths per frame
  - [ ] Support multiple transcripts in a single run

---

## 👤 Author

**<Hozaifa Alzohare>**
Biotechnology | Bioinformatics | Ai in Healthcare


---

## 📚 References

- GENCODE — https://www.gencodegenes.org/
- NCBI ORFfinder — https://www.ncbi.nlm.nih.gov/orffinder/
- ExPASy Translate — https://web.expasy.org/translate/
- EMBL-EBI Job Dispatcher (EMBOSS Transeq & Sixpack) — https://www.ebi.ac.uk/jdispatcher/
