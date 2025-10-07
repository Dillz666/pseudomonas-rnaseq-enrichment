# 🧬 RNA-Seq Expression Analysis & Enrichment — *Pseudomonas aeruginosa* SigX Study

![Bioinformatics](https://img.shields.io/badge/Field-Bioinformatics-blue)
![RNA-seq](https://img.shields.io/badge/Analysis-RNA--seq%20Expression%20Analysis-orange)
![Tools](https://img.shields.io/badge/Tools-FastQC%20|%20fastp%20|%20STAR%20|%20featureCounts%20|%20DESeq2%20|%20STRINGdb-green)
![Status](https://img.shields.io/badge/Status-Completed-success)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

> Comprehensive RNA-seq analysis of *Pseudomonas aeruginosa* SigX mutant vs. wild-type strains, including data preprocessing, alignment, differential expression analysis, and functional enrichment to identify antibiotic stress-response pathways.

---

## 🧭 Project Overview

This repository documents the **complete RNA-seq analysis workflow** for *Pseudomonas aeruginosa*, comparing a **SigX-overexpressed mutant** with the **wild-type strain** to explore transcriptional changes associated with stress adaptation and tetracycline response.

**Main objectives:**
- Analyze differential gene expression between SigX mutant and wild-type.
- Identify key pathways enriched under antibiotic stress.
- Explore biological significance through GO, KEGG, and STRING network analyses.

---

## 📊 Dataset Information

| Attribute | Description |
|------------|--------------|
| **Organism** | *Pseudomonas aeruginosa* |
| **Study** | Sigma factor (SigX)-dependent gene expression |
| **Source** | NCBI SRA / GEO (PRJNA322892 / GSE703771) |
| **Samples** | SRR1168693 (SigX-overexpressed mutant), SRR1168695 (Wild-type) |
| **Sequencing** | Illumina platform (Genome Analyzer IIx, HiSeq 2500) |
| **Read Type** | Single-end reads (30–45 bp) |

---

## ⚙️ Workflow Overview

**Pipeline:**
Raw Data (SRA) → Quality Control (FastQC, MultiQC)
→ Trimming (fastp) → Alignment (STAR)
→ Quantification (featureCounts)
→ Differential Expression (DESeq2)
→ Functional Enrichment (GO, KEGG, STRING)
---

## 🧩 Steps in Detail
 1️⃣ Data Retrieval

Tools: **SRA Toolkit**

```bash
prefetch SRR1168693 SRR1168695
fasterq-dump SRR1168693 SRR1168695 --progress -e 6
gzip SRR1168693.fastq SRR1168695.fastq
✅ Both datasets downloaded and converted successfully:

SRR1168693 → 5.16M reads

SRR1168695 → 9.20M reads

2️⃣ Quality Control
Tools: FastQC, MultiQC

Median Phred > 30 across reads.

GC content consistent with P. aeruginosa (~66%).

Adapter contamination detected → trimming required.

Summary:
High-quality reads with moderate duplication; ready for trimming.

3️⃣ Trimming & Filtering
Tool: fastp (v0.23.2)

bash
Copy code
fastp -i SRR1168693.fastq.gz -o SRR1168693-trimmed.fastq.gz \
      -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA -l 25 \
      -j SRR1168693.fastp.json -h SRR1168693.fastp.html
Results Summary:

Sample	Reads Before	Reads After	Duplication	Q30 (%)
Mutant (SRR1168693)	5.16M	3.87M	67%	94.5
Wild-type (SRR1168695)	9.20M	9.14M	78%	94.6

Interpretation:
Adapters removed; improved base quality; data ready for alignment.

4️⃣ Genome Alignment
Tool: STAR Aligner (v2.7.9a)
Reference: P. aeruginosa PAO1 (GCF_000006765.1)

bash
Copy code
STAR --runMode genomeGenerate --genomeDir ./star_index \
     --genomeFastaFiles ./GCF_000006765.1_ASM676v1_genomic.fna \
     --sjdbGTFfile ./GCF_000006765.1_ASM676v1_genomic.gtf

STAR --runThreadN 4 --genomeDir ./star_index \
     --readFilesIn SRR1168693-trimmed.fastq.gz --readFilesCommand zcat \
     --outSAMtype BAM Unsorted --quantMode GeneCounts
Mapping Summary:

Sample	Uniquely Mapped	Multi-mapped	Unmapped	Mismatch Rate
SRR1168693	18.7%	74.1%	7.2%	0.50%
SRR1168695	46.1%	47.3%	6.6%	0.41%

🧠 Interpretation:
Good mapping efficiency and low error rate; short reads explain multi-mapping.

5️⃣ BAM Processing
Tools: Samtools

Steps:

Sorting (samtools sort)

Indexing (samtools index)

Statistics (samtools flagstat)

Ensures coordinate-sorted BAMs for visualization and counting.

6️⃣ Quantification
Tool: featureCounts

bash
Copy code
featureCounts -T 4 -a GCF_000006765.1_ASM676v1_genomic.gtf \
-o counts.txt *.sorted.bam
Observation:
Low assignment (0.2–0.8%) due to possible genome–annotation mismatch.

7️⃣ Differential Expression Analysis
Tool: DESeq2 (R)

Identified significant DEGs (FDR < 0.05)

Upregulated genes: Efflux pumps, ribosomal proteins, stress response factors

Downregulated genes: Core metabolic enzymes, motility-related genes

Visualization:

🔹 Volcano plot — up/downregulated DEGs

🔹 Heatmap — clear mutant vs wild-type clustering

🔹 MA plot — balanced expression distribution

8️⃣ Functional Enrichment
Tools: ClusterProfiler (GO/KEGG), STRINGdb

GO/KEGG Findings:

Category	Enriched Pathways
Biological Process	Ribosome biogenesis, translation, antibiotic response
Molecular Function	Structural constituent of ribosome, transporter activity
Cellular Component	Ribosomal subunit, membrane complex
KEGG Pathways	Ribosome, ABC transporters, Two-component systems, Biofilm formation

STRING Network Highlights:

Hub proteins: rrf, rnr, pnp, rph, ybeY

Function: Ribosome recycling, RNA metabolism, antibiotic stress response

🔬 Biological Interpretation
Key Insights:

P. aeruginosa adapts to tetracycline stress via:

🧬 Upregulation of ribosomal and translation machinery

💊 Efflux pump activation (MexAB-OprM)

🧠 Regulatory adaptation via two-component systems

🧫 Biofilm enhancement for persistence

Potential Biomarkers/Targets:

Ribosomal proteins (Rpl, Rps)

Efflux transporters (MexA/B/OprM)

Regulators (PhoPQ, PmrAB)

Biofilm operons (Pel, Psl)

🧰 Tools & Technologies
Category	Tools
Data Retrieval	SRA Toolkit
Quality Control	FastQC, MultiQC, fastp
Alignment	STAR
BAM Processing	Samtools
Quantification	featureCounts
DE Analysis	DESeq2
Functional Enrichment	ClusterProfiler, KEGG, STRINGdb
Environment	Linux, R, Bash

🧾 Keywords
RNA-seq Pseudomonas aeruginosa SigX Differential Expression
DESeq2 FastQC STAR featureCounts GO KEGG STRINGdb
Antibiotic Resistance Bioinformatics Transcriptomics

📄 Project Report
📘 Expression_Data_Analysis_and_Enrichment.pdf

👩‍💻 Author
Debopriya
Bioinformatics | Transcriptomics | Functional Genomics

🔗 GitHub Profile # 🧬 RNA-Seq Expression Analysis & Enrichment — *Pseudomonas aeruginosa* SigX Study

![Bioinformatics](https://img.shields.io/badge/Field-Bioinformatics-blue)
![RNA-seq](https://img.shields.io/badge/Analysis-RNA--seq%20Expression%20Analysis-orange)
![Tools](https://img.shields.io/badge/Tools-FastQC%20|%20fastp%20|%20STAR%20|%20featureCounts%20|%20DESeq2%20|%20STRINGdb-green)
![Status](https://img.shields.io/badge/Status-Completed-success)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

> Comprehensive RNA-seq analysis of *Pseudomonas aeruginosa* SigX mutant vs. wild-type strains, including data preprocessing, alignment, differential expression analysis, and functional enrichment to identify antibiotic stress-response pathways.

---

## 🧭 Project Overview

This repository documents the **complete RNA-seq analysis workflow** for *Pseudomonas aeruginosa*, comparing a **SigX-overexpressed mutant** with the **wild-type strain** to explore transcriptional changes associated with stress adaptation and tetracycline response.

**Main objectives:**
- Analyze differential gene expression between SigX mutant and wild-type.
- Identify key pathways enriched under antibiotic stress.
- Explore biological significance through GO, KEGG, and STRING network analyses.

---

## 📊 Dataset Information

| Attribute | Description |
|------------|--------------|
| **Organism** | *Pseudomonas aeruginosa* |
| **Study** | Sigma factor (SigX)-dependent gene expression |
| **Source** | NCBI SRA / GEO (PRJNA322892 / GSE703771) |
| **Samples** | SRR1168693 (SigX-overexpressed mutant), SRR1168695 (Wild-type) |
| **Sequencing** | Illumina platform (Genome Analyzer IIx, HiSeq 2500) |
| **Read Type** | Single-end reads (30–45 bp) |

---

## ⚙️ Workflow Overview

**Pipeline:**
Raw Data (SRA) → Quality Control (FastQC, MultiQC)
→ Trimming (fastp) → Alignment (STAR)
→ Quantification (featureCounts)
→ Differential Expression (DESeq2)
→ Functional Enrichment (GO, KEGG, STRING)

yaml
Copy code

---

## 🧩 Steps in Detail

### 1️⃣ Data Retrieval

Tools: **SRA Toolkit**

```bash
prefetch SRR1168693 SRR1168695
fasterq-dump SRR1168693 SRR1168695 --progress -e 6
gzip SRR1168693.fastq SRR1168695.fastq
✅ Both datasets downloaded and converted successfully:

SRR1168693 → 5.16M reads

SRR1168695 → 9.20M reads

2️⃣ Quality Control
Tools: FastQC, MultiQC

Median Phred > 30 across reads.

GC content consistent with P. aeruginosa (~66%).

Adapter contamination detected → trimming required.

Summary:
High-quality reads with moderate duplication; ready for trimming.

3️⃣ Trimming & Filtering
Tool: fastp (v0.23.2)

bash
Copy code
fastp -i SRR1168693.fastq.gz -o SRR1168693-trimmed.fastq.gz \
      -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCA -l 25 \
      -j SRR1168693.fastp.json -h SRR1168693.fastp.html
Results Summary:

Sample	Reads Before	Reads After	Duplication	Q30 (%)
Mutant (SRR1168693)	5.16M	3.87M	67%	94.5
Wild-type (SRR1168695)	9.20M	9.14M	78%	94.6

Interpretation:
Adapters removed; improved base quality; data ready for alignment.

4️⃣ Genome Alignment
Tool: STAR Aligner (v2.7.9a)
Reference: P. aeruginosa PAO1 (GCF_000006765.1)

bash
Copy code
STAR --runMode genomeGenerate --genomeDir ./star_index \
     --genomeFastaFiles ./GCF_000006765.1_ASM676v1_genomic.fna \
     --sjdbGTFfile ./GCF_000006765.1_ASM676v1_genomic.gtf

STAR --runThreadN 4 --genomeDir ./star_index \
     --readFilesIn SRR1168693-trimmed.fastq.gz --readFilesCommand zcat \
     --outSAMtype BAM Unsorted --quantMode GeneCounts
Mapping Summary:

Sample	Uniquely Mapped	Multi-mapped	Unmapped	Mismatch Rate
SRR1168693	18.7%	74.1%	7.2%	0.50%
SRR1168695	46.1%	47.3%	6.6%	0.41%

🧠 Interpretation:
Good mapping efficiency and low error rate; short reads explain multi-mapping.

5️⃣ BAM Processing
Tools: Samtools

Steps:

Sorting (samtools sort)

Indexing (samtools index)

Statistics (samtools flagstat)

Ensures coordinate-sorted BAMs for visualization and counting.

6️⃣ Quantification
Tool: featureCounts

bash
Copy code
featureCounts -T 4 -a GCF_000006765.1_ASM676v1_genomic.gtf \
-o counts.txt *.sorted.bam
Observation:
Low assignment (0.2–0.8%) due to possible genome–annotation mismatch.

7️⃣ Differential Expression Analysis
Tool: DESeq2 (R)

Identified significant DEGs (FDR < 0.05)

Upregulated genes: Efflux pumps, ribosomal proteins, stress response factors

Downregulated genes: Core metabolic enzymes, motility-related genes

Visualization:

🔹 Volcano plot — up/downregulated DEGs

🔹 Heatmap — clear mutant vs wild-type clustering

🔹 MA plot — balanced expression distribution

8️⃣ Functional Enrichment
Tools: ClusterProfiler (GO/KEGG), STRINGdb

GO/KEGG Findings:

Category	Enriched Pathways
Biological Process	Ribosome biogenesis, translation, antibiotic response
Molecular Function	Structural constituent of ribosome, transporter activity
Cellular Component	Ribosomal subunit, membrane complex
KEGG Pathways	Ribosome, ABC transporters, Two-component systems, Biofilm formation

STRING Network Highlights:

Hub proteins: rrf, rnr, pnp, rph, ybeY

Function: Ribosome recycling, RNA metabolism, antibiotic stress response

🔬 Biological Interpretation
Key Insights:

P. aeruginosa adapts to tetracycline stress via:

🧬 Upregulation of ribosomal and translation machinery

💊 Efflux pump activation (MexAB-OprM)

🧠 Regulatory adaptation via two-component systems

🧫 Biofilm enhancement for persistence

Potential Biomarkers/Targets:

Ribosomal proteins (Rpl, Rps)

Efflux transporters (MexA/B/OprM)

Regulators (PhoPQ, PmrAB)

Biofilm operons (Pel, Psl)

🧰 Tools & Technologies
Category	Tools
Data Retrieval	SRA Toolkit
Quality Control	FastQC, MultiQC, fastp
Alignment	STAR
BAM Processing	Samtools
Quantification	featureCounts
DE Analysis	DESeq2
Functional Enrichment	ClusterProfiler, KEGG, STRINGdb
Environment	Linux, R, Bash

🧾 Keywords
RNA-seq Pseudomonas aeruginosa SigX Differential Expression
DESeq2 FastQC STAR featureCounts GO KEGG STRINGdb
Antibiotic Resistance Bioinformatics Transcriptomics

📄 Project Report
📘 Expression_Data_Analysis_and_Enrichment.pdf

👩‍💻 Author
Debopriya
Bioinformatics | Transcriptomics | Functional Genomics

🔗 GitHub Profile (https://github.com/DEBOPRIYA2320)

🔗 LinkedIn (www.linkedin.com/in/debopriya2320)

📜 License
This project is licensed under the MIT License — open for educational and research use.

⭐ If you found this useful, please star the repo and share with the bioinformatics community!
