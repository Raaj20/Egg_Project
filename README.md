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

grep -i 'Rattus norvegicus' mature.fa | head

awk '/^>/{p = ($0 ~ /^>rno/)} p{print}' mature.fa > mature_rno.fa

grep -c '^>' mature_rno.fa
764

awk '/^>/{print; next} { gsub(/[Uu]/, "T"); print }' mature_rno.fa > mature_rnoTs.fa

head mature_rnoTs.fa

bowtie-build mature_rnoTs.fa mature_rnoTs
Settings:
  Output files: "mature_rnoTs.*.ebwt"

