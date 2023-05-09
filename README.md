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
              
              for i in $(ls [3-9]*_1.fastq.gz); do echo $i; bbduk.sh in=$i in2=$(echo $i | cut -d'_' -f1)_2.fastq.gz out=../01_Cleanning/$(echo $i | cut -d'_' -f1)_1_clean.fastq.gz out2=../01_Cleanning/$(echo $i | cut -d'_' -f1)_2_clean.fastq.gz ref=adapters ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=r trimq=24 ; done
              
    on cluster test
              
              sbatch --partition=pall -w binfservas37 --job-name=bbtest --time=00:01:00 --mem-per-cpu=32G --cpus-per-task=8 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL 
              sbatch --partition=pall -w binfservas20 --job-name=bbtest --time=00:00:03 --mem-per-cpu=2G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL; echo "HELLO WORLD" ; sleep 40
              
              #Create statistics table
              echo "" > Reads_raw.tmp
              for i in $(ls *.fastq.gz); do zcat $i | grep ">" | wc -l >> Reads_raw.tmp; done  # with this commmand ex
              echo "" > Reads_cleaned.tmp
              for i in $(ls *.fastq.gz); do zcat $i | grep ">" | wc -l >> Reads_cleaned.tmp ; done  # with this commmand ex
              
              # in cluster
              sbatch --partition=pall --job-name=countraw --time=00:10:00 --mem-per-cpu=8G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "echo "" > Reads_raw.tmp; for i in $(ls *.fastq.gz); do zcat $i | grep ">" | wc -l ; done"  
              
## 2 . Identify mycorrhizal reads with MASH  
https://mash.readthedocs.io/en/latest/

a. Download Mycorrhizal GenBank genome assemblies

b. Build reference index of mycorrhizal genomes
               
               mash sketch -o AMF_GeneBank_ref */*.fna.gz
               mash info reference.msh
               
       
       
c. Screen genomes against MASH refSeq db
The fields ouptut are [identity, shared-hashes, median-multiplicity, p-value, query-ID, query-comment]:

                for i in $(ls *.fastq.gz); do echo $i; ~/00_Software/mash-Linux64-v2.3/mash screen -w -p 8 /data/projects/p782_RNA_seq_Argania_spinosa/11_Mycorrhizal_containement/AMF_GeneBank_ref.msh $i > /data/projects/p782_RNA_seq_Argania_spinosa/11_Mycorrhizal_containement/Screen_$(echo $i | cut -d'.' -f1).out ; done


                        # Sort and add filename in last column
                        for f in Screen*; do echo $f; sort -gr $f | sed -i "s/$/\t$f/" $f; done    
                        
                        Concatenate all files into a table

See more common

     for i in $(ls Screen_*.out); do echo $i; sort -gr $i | head ; done

       
       IN loop
       
      for i in $(ls *.fna.gz); do echo $i; /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/Soft/mash-Linux64-v2.2/mash screen -w -p 16 /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/10_Contaminants/00_GeneBank_Complete_Bacteria/Reference_GeneBank_2549_Bacteria_completeGenomes.msh $i > ../03_BacteriaContaminants_Glomeromycetes/Screen_RefSeqk21S1000_$(echo $i | cut -d'_' -f1,2,3).tab ; done 



## 3. transcript quantification with salmon
https://combine-lab.github.io/salmon/getting_started/


a. obtain salmon

                module load UHTS/Analysis/salmon/0.11.2

b. build an index on the transcriptome

                salmon index -t ../00_Genome/Argan2-publish-files/Argania-spinosa-argan2-v1.0.a1.6346630e4cbe5-publish.genes.fna -i argane_index

c. Quantify the samples

                # in local
                for i in $(ls 9*_1_clean.fastq.gz); do echo $i;salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/02_SalmonQuantification/argane_index -l A -1 $i -2 $(echo $i | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/02_SalmonQuantification/$i_quant ; done

                #in cluster      
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=bbtest --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/02_SalmonQuantification/argane_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/02_SalmonQuantification/$(echo $FILE | cut -d'_' -f1)_quant "   ; sleep 1; done



## 3. Quantification import with tximport


## 4. Differential expression with DEseq2


## Gene enrichement analysis







