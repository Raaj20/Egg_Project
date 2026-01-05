# Egg_Project

# Install Homebrew 
git clone https://github.com/Homebrew/brew homebrew
eval "$(homebrew/bin/brew shellenv)"
brew update --force --quiet
chmod -R go-w "$(brew --prefix)/share/zsh"

# Create an environment in Conda
brew install miniforge
conda update -n base -c conda-forge conda

 conda --version
conda 25.11.1

conda create -n mirna_pipeline python=3.10

echo $SHELL
/bin/zsh

conda init zsh

# Restart Terminal

conda activate mirna_pipeline

# Install required packages
conda install -c bioconda fastqc
conda install -c bioconda bbmap
conda install -c bioconda bowtie
conda install -c bioconda samtools
conda install -c bioconda multiqc


# Version
fastqc --version 
FastQC v0.12.1

bowtie --version
/usr/local/Caskroom/miniforge/base/envs/mirna_pipeline/bin/bowtie-align-s version 1.3.1
64-bit
Built on Mac-1733801072041.local
2024-12-10T05:02:18

 samtools --version
samtools 1.23

multiqc --version            
multiqc, version 1.33

# Indexing and setting up Directories
cd Egg_Project/Index 
head mature.fa 
grep '^>' mature.fa | head -n 20

grep '^>' mature.fa | sed 's/>//' | cut -d'-' -f1 | sort -u

### Double-checking if there are any Rattus norvegicus miRNAs to align
grep -i 'Rattus norvegicus' mature.fa | head

awk '/^>/{p = ($0 ~ /^>rno/)} p{print}' mature.fa > mature_rno.fa

grep -c '^>' mature_rno.fa
764

### Convert Us to Ts and doublecheck number of miRNAs before and after conversion
awk '/^>/{print; next} { gsub(/[Uu]/, "T"); print }' mature_rno.fa > mature_rnoTs.fa

head mature_rnoTs.fa

### Indexing the Rhesus macaque + U-->T converted .fa files
bowtie-build mature_rnoTs.fa mature_rnoTs
Settings:
  Output files: "mature_rnoTs.*.ebwt"

## Unzip Files
gunzip -k /Users/viraajv/Egg_Project/Zipped_Files/*.fastq.gz

## Make Directories
INPUTDIR="/Users/viraajv/Egg_Project/Unzipped_Files"
OUTPUTDIR="/Users/viraajv/Egg_Project/Analysis"
INDEX="/Users/viraajv/Egg_Project/Index"

## Pre-trimmed QC
for file in "$INPUTDIR"/*.fastq; do   
    echo "Running FastQC on: $file"
    fastqc -o "$QCREPORT" -t 16 "$file"
done

### Multi QC
cd /Users/viraajv/Egg_Project/Unzipped_Files/PreQCReports/MultiQC_Report
multiqc . -o . 


## Adapter Trimming
INPUTDIR="/Users/viraajv/Egg_Project/Unzipped_Files"
OUTPUTDIR="/Users/viraajv/Egg_Project/Analysis/trimmed_reads"

mkdir -p "$OUTPUTDIR"

for file in "$INPUTDIR"/*.fastq; do
    i=$(basename "$file" .fastq)

    echo "Trimming adapter from $i..."

    bbduk.sh -Xmx27g \
        in="$file" \
        out="$OUTPUTDIR/${i}-Trimmed.fastq" \
        literal=TGGAATTCTCGGGTGCCAAGGAACTCCAGTCAC \
        ktrim=r k=21 mink=11 hdist=2
done



## Bowtie Alignment
5' – TGGAATTCTCGGGTGCCAAGGAACTCCAGTCAC – 3' 

OUTPUTDIR="/Users/viraajv/Egg_Project/Analysis/trimmed_reads"
INDEX="/Users/viraajv/Egg_Project/Index/mature_rnoTs"
COUNTDIR="$OUTPUTDIR/maturemiRNAcounts"

# Create output folder for counts
mkdir -p "$COUNTDIR"

# --- Loop through all trimmed FASTQ files ---
for file in "$OUTPUTDIR"/*-Trimmed.fastq; do
    i=$(basename "$file" -Trimmed.fastq)

    echo "Running Bowtie on $i..."

    bowtie -v 1 -k 1 -m 1 --best --strata --threads 16 -S\
        "$INDEX" \
        -q "$file" \
        --un "$OUTPUTDIR/${i}-unaligned.fastq" \
        -S "$OUTPUTDIR/${i}.sam" \
        2> "$OUTPUTDIR/${i}.log"

    echo "Converting SAM to BAM and indexing..."

    samtools sort "$OUTPUTDIR/${i}.sam" -o "$OUTPUTDIR/${i}.bam"
    samtools index "$OUTPUTDIR/${i}.bam"
    rm "$OUTPUTDIR/${i}.sam"

    echo "Counting mapped reads for $i..."

    samtools idxstats "$OUTPUTDIR/${i}.bam" | cut -f1,3 | \
        sed "1s/^/miRNA\t${i}-miRNAcount\n/" > "$COUNTDIR/${i}-counts.txt"

    echo "Done with $i."
done


## Post-trimmed QC
for file in "$OUTPUTDIR"/*Trimmed.fastq; do   
    echo "Running FastQC on: $file"
    fastqc -o "$QCREPORT" -t 16 "$file"
done

cd /Users/viraajv/Egg_Project/Analysis/trimmed_reads/MultiQC_Report
multiqc . -o .

