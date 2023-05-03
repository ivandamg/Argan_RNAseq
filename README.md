# Argane RNAseq Bioinformatic analysis
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
              
              srun --cpus-per-task=1 --mem-per-cpu=32000 --time=00:15:00 --pty bbduk.sh in=1A_1.fastq.gz in2=1A_2.fastq.gz out=../01_Cleanning/1A_1_clean.fastq.gz out2=../01_Cleanning/1A_2_clean.fastq.gz ref=adapters ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=r trimq=24
              
              #Create statistics table
              
## 2 . Identify mycorrhizal reads with MASH  
https://mash.readthedocs.io/en/latest/

a. Download Mycorrhizal RefSeq genomes 

b. Build reference index of mycorrhizal genomes
               
               mash sketch -o reference genome1.fna genome2.fna
               mash info reference.msh
               /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/Soft/mash-Linux64-v2.2/mash sketch -o References_glomero6 *.fna.gz
       
       
c. Screen genomes against MASH refSeq db
The fields ouptut are [identity, shared-hashes, median-multiplicity, p-value, query-ID, query-comment]:

       /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/Soft/mash-Linux64-v2.2/mash screen -w -p 8 RefSeq_fungi_1perSpecies_1721.msh ../../../BRA337_DAOM197198_1_SRX4991128_1.fastq.gz > ../Screen_BRA337_DAOM197198_1.tab

for i in $(ls *.fastq.gz); do echo $i; /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/Soft/mash-Linux64-v2.2/mash screen -w -p 8 /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/02_Co-inoc/ZZ_ISMEJ_REVISION/06_OtherSeqOrg/01_Reffungi1species/RefSeq_fungi_1perSpecies_1721.msh $i > ZZ_ISMEJ_REVISION/06_OtherSeqOrg/Screen_$(echo $i | cut -d'_' -f1,2,3).out ; done


# add filename in last column
 for f in Screen*; do sed -i "s/$/\t$f/" $f; done    

See more common

     for i in $(ls Screen_*.out); do echo $i; sort -gr $i | head ; done

       
       IN loop
       
      for i in $(ls *.fna.gz); do echo $i; /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/Soft/mash-Linux64-v2.2/mash screen -w -p 16 /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/10_Contaminants/00_GeneBank_Complete_Bacteria/Reference_GeneBank_2549_Bacteria_completeGenomes.msh $i > ../03_BacteriaContaminants_Glomeromycetes/Screen_RefSeqk21S1000_$(echo $i | cut -d'_' -f1,2,3).tab ; done 



## 3. transcript quantification with salmon
https://combine-lab.github.io/salmon/getting_started/


a. obtain salmon


b. build an index on the transcriptome


c. Quantify the samples


                for FILE in *clean1.fastq.gz; do echo ${FILE}; sbatch -o $(echo ${FILE} | cut -d'' -f1,2)bbmap.stdout.txt -J B$(echo ${FILE} | cut -d'' -f1,2) -e {FILE}| cut -d'_' -f1,2)_bbmap.stderr.txt ../0003_bbmap_curnagleB1_v2.sh ${FILE}; sleep 1; done

## 3. Quantification import with tximport


## 4. Differential expression with DEseq2


## Gene enrichement analysis







