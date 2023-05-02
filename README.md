# Argan RNAseq Bioinformatic analysis
v1
Argan transcriptomic analysis on drought and AMF conditions





## Experimental design

The plants were raised on growth chambers on different conditions. The conditions are composed of two different variables ( Mycorrhizal and Drought) with three replicates each. 

## Genomic ressources

The genome asm_hifi_Both.p_ctg was assembled on 1002 contigs and annotated. There were found 22349 gene models 

        cat asm_hifi_Both.p_ctg.fa | grep ">" | wc -l #For contig counts
        cat asm_hifi_Both.p_ctg.gtf | grep "gene[[:blank:]]" | wc -l #for gene model counts


## 1. Reads adapter trimming and quality filtering
BBduk documentation : https://jgi.doe.gov/data-and-tools/software-tools/bbtools/bb-tools-user-guide/bbduk-guide/
parameters used as  in documentation examples


              mkdir 01_Cleanning
              module load UHTS/Analysis/BBMap/38.91  
              
              srun --cpus-per-task=1 --mem-per-cpu=24000 --time=00:10:00 --pty bbduk.sh in=1A_1.fastq.gz in2=1A_2.fastq.gz out=../01_Cleanning/1A_1_clean.fastq.gz out2=../01_Cleanning/1A_2_clean.fastq.gz ref=adapters ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=r trimq=24
              
              #Create statistics table
              

## 2. transcript quantification with salmon
https://combine-lab.github.io/salmon/getting_started/


a. obtain salmon


b. build an index on the transcriptome


c. Quantify the samples


                for FILE in *clean1.fastq.gz; do echo ${FILE}; sbatch -o $(echo ${FILE} | cut -d'' -f1,2)bbmap.stdout.txt -J B$(echo ${FILE} | cut -d'' -f1,2) -e {FILE}| cut -d'_' -f1,2)_bbmap.stderr.txt ../0003_bbmap_curnagleB1_v2.sh ${FILE}; sleep 1; done

## 3. Quantification import with tximport


## 4. Differential expression with DEseq2


## Gene enrichement analysis







