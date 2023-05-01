# Argan_RNAseq Bioinformatic analysis
v1
Argan transcriptomic analysis on drought and AMF conditions





## Experimental design

The plants were raised on growth chambers on different conditions. The conditions are composed of two different variables ( Mycorrhizal and Drought) with three replicates each. 

## Genomic ressources

The genome asm_hifi_Both.p_ctg was assembled on 1002 contigs and annotated. There were found 22349 gene models 

        cat asm_hifi_Both.p_ctg.fa | grep ">" | wc -l #For contig counts
        cat asm_hifi_Both.p_ctg.gtf | grep "gene[[:blank:]]" | wc -l #for gene model counts


## Reads adapter trimming and quality filtering

              mkdir 01_trimmed
              module load gcc bbmap/38.6  
           bbduk.sh in=B1_1_L4_R1_001_2avKbOqY7ITX.fastq.gz in2=B1_1_L4_R2_001_bdo0VZLZah6p.fastq.gz out=01_trimmed/B1_1_clean1.fastq.gz out2=01_trimmed/B1_1_clean2.fastq.gz ref=/work/FAC/FBM/DEE/isanders/popgen_to_var/IM/ZZ_Soft/bbmap/resources/adapters.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=r trimq=24


## Mapping


## Gene Counts


## Differential expression


## Gene enrichement analysis







