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
* Each sample must have exactly **two files**, one for forward reads and one for reverse reads.
* Files should follow a naming convention such as:

```
SRR15836018_S1_L001_R1_001.fastq   # Forward (R1)
SRR15836018_S1_L001_R2_001.fastq   # Reverse (R2)
```

* Both files must be present and correctly named for each sample to be processed.

---

## 💼 Output

* Processed OTU tables
* Taxonomic assignment results
* Quality control plots
* Summary statistics

---

For questions or issues, refer to the Docker Hub or GitHub repository of the image owner.
