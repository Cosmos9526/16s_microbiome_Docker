# 👫 16S Microbiome Docker Pipeline

**16s\_microbiome\_Docker** is a streamlined, containerized pipeline for analyzing 16S rRNA sequencing data using Snakemake and Python. It supports reproducible and scalable microbiome analysis with built-in quality control, taxonomic profiling, and customizable threading.

---

## 🐳 Docker Image

```bash
docker pull cosmos9526/microbiom16s:latest
```

This Docker image comes pre-configured with:

* ✅ Snakemake workflow engine
* ✅ 16S rRNA processing pipeline (QIIME2-style)
* ✅ Multi-threading support
* ✅ Fully isolated Conda environment

---

## 🚀 Run the Pipeline

```bash
docker run --rm \
    -v /home/ubuntu/00Milad/Dockermicrobiome:/data \
    cosmos9526/microbiom16s:latest \
    -c "source /opt/conda/envs/snakemake/bin/activate snakemake && \
        python run.py -i /data/16s -o /data/result -m 16s -n 20"
```

---

## 🔧 Parameters

| Parameter              | Description                                     |
| ---------------------- | ----------------------------------------------- |
| `--rm`                 | Automatically remove container after completion |
| `-v /local/path:/data` | Mount your local directory to the container     |
| `-i`                   | Input directory (must contain raw 16S data)     |
| `-o`                   | Output directory for results                    |
| `-m 16s`               | Mode set to `16s` for 16S rRNA analysis         |
| `-n 20`                | Number of threads for parallel processing       |

---

## 📚 Input File Requirements

* Input directory should contain paired-end FASTQ files.
* Each sample must include exactly **two files**, one for forward reads (***R1***) and one for reverse reads (***R2***).
* Files must follow this flexible naming pattern where filenames contain `*R1*` and `*R2*` respectively:

📝 **Example:**

```
SRR15836018_S1_L001_R1_001.fastq   # *R1* = forward
SRR15836018_S1_L001_R2_001.fastq   # *R2* = reverse
```

* The pipeline will automatically pair files using the shared prefix **before** `*R1*` / `*R2*`.
* Do **not** rename or split files outside this convention.

---

## 💼 Output

* Processed OTU tables
* Taxonomic assignment results
* Quality control plots
* Summary statistics

---

For questions or issues, refer to the Docker Hub or GitHub repository of the image owner.



# 16S rRNA Amplicon Sequencing Pipeline - README

This README describes a step-by-step overview of the 16S rRNA preprocessing pipeline using QIIME2, including all modules used, the configuration setup, file paths, and expected inputs and outputs for each stage.

---

## 🧾 Configuration Summary

The pipeline uses a config file `config.cfg` located at the root of the repository. This file contains necessary parameters such as:

```ini
[paths]
input_dir = /data/0pipe/input_Example
output_dir = /data/results
cohort_name = AB
threads = 10
```

---

## 📂 Folder Structure

* `input_dir`: contains paired-end FASTQ files (e.g., R1 and R2)
* `output_dir`: all output results are saved here
* Manifest, QZA artifacts, and stats files are placed under:

  ```
  ```

output\_dir/cohort\_name/

```

---

## 🔧 Pipeline Steps

### 1. **Download Database Files**
- **Function**: `download_db_files()`
- **Purpose**: Downloads taxonomic mapping files needed for later analysis.
- **Inputs**: none
- **Outputs**: `names.json`, `taxonpath.json` saved in a local `db/` folder
- ✅ Status is logged if files already exist or downloaded

---

### 2. **Run FASTQC**
- **Function**: wrapper using `run_external_command()`
- **Purpose**: Performs quality check of FASTQ files
- **Inputs**: All `.fastq.gz` files in `input_dir`
- **Outputs**: FASTQC HTML reports in `output_dir/quality/fastqc/`

---

### 3. **Create Manifest File**
- **Function**: `create_manifest(input_dir, manifest_path)`
- **Purpose**: Maps sample IDs to their corresponding R1 and R2 FASTQ paths
- **Inputs**: Paired-end FASTQ files in `input_dir`
- **Outputs**: `manifest.tsv` file with sample-id, forward, reverse filepaths

---

### 4. **Import Data to QIIME2**
- **Command**: `qiime tools import`
- **Purpose**: Converts FASTQ + manifest to `.qza` artifact for further processing
- **Inputs**: `manifest.tsv`
- **Outputs**: `cohort_name.qza`

---

### 5. **Calculate Trimming Length (Dynamic)**
- **Function**: `calculate_len_from_fastq()`
- **Purpose**: Reads ~1000 reads and checks average quality per base
- **Logic**:
  - Trims front positions with low average quality
  - If fails, fallback to `trim_front = 4`
- **Inputs**: First FASTQ file (from manifest)
- **Outputs**: Two integers (`len1`, `len2`) representing trimming length

---

### 6. **Run Fastp Trimming**
- **Function**: `fastp_trim(fastp_artifact, len1, len2)`
- **Purpose**: Applies quality trimming from front for both R1 and R2
- **Inputs**: `.qza` from import step, `len1`, `len2`
- **Outputs**: `+fp-f-r.qza` trimmed reads

---

### 7. **Run BBDuk Trimming**
- **Function**: `bbduk_trim(bbduk_artifact, threshold)`
- **Purpose**: Quality filtering using a threshold score (Q=22 by default)
- **Inputs**: same `.qza` as fastp input, or can be fastp output
- **Outputs**: `+bb-t22.qza`

---

### 8. **Denoising with DADA2**
- **Function**: `dada2_denoise()`
- **Purpose**: Applies denoising, chimera filtering, and dereplication
- **Inputs**: Preprocessed `.qza` file (fastp/bbduk output)
- **Outputs**:
  - `+dd_table.qza` → Feature table
  - `+dd_seq.qza` → Representative sequences
  - `+dd_stats.qza` → DADA2 run statistics

---

## ✅ Final Outputs

All outputs are stored under:
```

/data/results/{cohort\_name}/

```
With filenames like:
```

AB+fp-f-r.qza
AB+bb-t22.qza
AB+dd\_table.qza
AB+dd\_seq.qza
AB+dd\_stats.qza

```

---

## 📌 Notes
- **Logging**: `log_status()` is used throughout to provide status of each step
- **Fallbacks**: Dynamic functions like `calculate_len()` and `calculate_threshold()` include safe defaults in case of failure
- **Primers used**: V3-V4 primers (e.g., `CCTACGGGNGGCWGCAG`, `GACTACHVGGGTATCTAATCC`)

---

For questions or customizations (e.g., change region, trimming rules, taxonomy assignments), feel free to extend each function. This pipeline is designed to be modular and fault-tolerant.

---

Happy microbiome analysis! 🧬

```

