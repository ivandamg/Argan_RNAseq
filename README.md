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

## 3. transcript quantification of AMF with salmon
https://combine-lab.github.io/salmon/getting_started/

Quantify how many transcript map to different AMF genomes: 
1. Claroideoglomus_candidum
2. Diversispora_eburnea
3. Diversispora_epigaea
4. Rhizophagus_irregularis
5. Scutellospora_calospora
6. Funneliformis_mosseae
7. Rhizophagus_clarus
8. Gigaspora_rosea
9. Glomus_cerebriforme




a. obtain salmon

                module load UHTS/Analysis/salmon/0.11.2

b. build an index on the transcriptome

                salmon index -t GCA_910592215.1_Claroideoglomus_candidum_CCK_pot_B_6-9_cds_from_genomic.fna -i Claroideoglomus_candidum_index
                salmon index -t GCA_910592015.1_Diversispora_eburnea_AZ414A_cds_from_genomic.fna -i Diversispora_eburnea_index
                salmon index -t GCA_003547095.1_Glomus_Soap2_v3.1.nr.fa_cds_from_genomic.fna -i Diversispora_epigaea_index
                salmon index -t GCA_000439145.3_ASM43914v3_cds_from_genomic.fna.gz -i Rhizophagus_irregularis_index
                salmon index -t GCA_910592325.1_Scutellospora_calospora_AU212A_cds_from_genomic.fna -i Scutellospora_calospora_index
                salmon index -t GCA_910592005.1_Funneliformis_mosseae_87-6_pot_B_2015_cds_from_genomic.fna -i Funneliformis_mosseae_index
                salmon index -t GCA_015698045.1_RCL21_cds_from_genomic.fna -i Rhizophagus_clarus_index
                salmon index -t GCA_003550325.1_ASM355032v1_cds_from_genomic.fna -i Gigaspora_rosea_index
                salmon index -t GCA_003550305.1_ASM355030v1_cds_from_genomic.fna -i Glomus_cerebriforme_index

                
                
                
c. Quantify the samples

                #in cluster      
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Claroideoglomus_candidum_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Claroideoglomus_candidum_quant "   ; sleep 1; done
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=DE$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Diversispora_eburnea_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Diversispora_eburnea_quant "   ; sleep 1; done
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=DE$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Diversispora_epigaea_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Diversispora_epigaea_quant "   ; sleep 1; done
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=Ri$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Rhizophagus_irregularis_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Rhizophagus_irregularis_quant "   ; sleep 1; done
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=Sc$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Scutellospora_calospora_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Scutellospora_calospora_quant "   ; sleep 1; done
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=Fm$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Funneliformis_mosseae_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Funneliformis_mosseae_quant "   ; sleep 1; done
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=Fm$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Rhizophagus_clarus_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Rhizophagus_clarus_quant "   ; sleep 1; done
                
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=Gr$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Gigaspora_rosea_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Gigaspora_rosea_quant "   ; sleep 1; done
                
                for FILE in $(ls *_1_clean.fastq.gz); do echo $FILE; sbatch --partition=pall --job-name=Gc$(echo $FILE | cut -d'_' -f1) --time=02:00:00 --mem-per-cpu=12G --cpus-per-task=1 --output=bbtest.out --error=bbtest.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/salmon/0.11.2; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/Glomus_cerebriforme_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_clean.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/03_SalmonQuantificationAMF/$(echo $FILE | cut -d'_' -f1)_Glomus_cerebriforme_quant "   ; sleep 1; done
                

                # Count nb. of genes with more than zero reads 
                for i in $(ls *_quant/*.sf); do cat $i | awk '$5 >0' | wc -l; done
                # Change name of quant file for all files
                cp 7A_quant/quant.sf 7A_quant.sf
                cp 8A_quant/quant.sf 8A_quant.sf
                cp 9A_quant/quant.sf 9A_quant.sf
                gzip *.sf




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

                # Count nb. of genes with more than zero reads 
                for i in $(ls *_quant/*.sf); do cat $i | awk '$5 >0' | wc -l; done
                # Change name of quant file for all files
                cp 7A_quant/quant.sf 7A_quant.sf
                cp 8A_quant/quant.sf 8A_quant.sf
                cp 9A_quant/quant.sf 9A_quant.sf
                gzip *.sf

## 3. Quantification import with tximport in R
http://bioconductor.org/packages/release/bioc/vignettes/tximport/inst/doc/tximport.html





## 4. Differential expression with DESeq2 in R
https://www.bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#differential-expression-analysis



## Gene enrichement analysis







