# Pipeline v2. 


1. Clean reads with fastp

         for FILE in $(ls *1.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)fastp --time=0-02:00:00 --mem-per-cpu=24G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_fastp.out --error=$(echo $FILE | cut -d'_' -f1)_fastp.error --mail-type=END,FAIL --wrap " cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/00_rawreads ; module load FastQC; ~/00_Software/fastp --in1 $FILE --in2 $(echo $FILE | cut -d'_' -f1)_2.fastq.gz --out1 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_1_trimmed.fastq.gz --out2 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -h ../02_TrimmedData/$(echo $FILE | cut -d',' -f1)_fastp.html --thread 4; fastqc -t 4 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_1_trimmed.fastq.gz; fastqc -t 4 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz"; sleep  1; done


2. transcript quantification with salmon

https://combine-lab.github.io/salmon/getting_started/

a. obtain salmon

            module load UHTS/Analysis/salmon/0.11.2

b. build an index on the transcriptome

            
            sbatch --partition=pshort_el8 --job-name=SalmonIndex --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=SalmonIndexHap1.out --error=SalmonIndexHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/00_ref; salmon index -t Argania_spinosa_DR.mrna-transcripts.fa.gz -i Argan_hap1_index"

c. Quantify the samples

         
            #on haplotype 1  
            for FILE in $(ls *1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_Hap1 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.out --error=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/02_TrimmedData ; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/00_ref/Argan_hap1_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/01_SalmonQuant/$(echo $FILE | cut -d'_' -f1)_quant"; sleep  1; done

            #on haplotype 2  
            for FILE in $(ls *1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_Hap1 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.out --error=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/02_TrimmedData ; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/04_haplotype2/00_ref/Argan_hap2_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/04_haplotype2/01_SalmonQuant/$(echo $FILE | cut -d'_' -f1)_quant"; sleep  1; done


d. mix the transcriptome together and build an index on the transcriptome
Change name of reads for hap2
            
            sbatch --partition=pshort_el8 --job-name=SalmonIndex --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=SalmonIndexHap1.out --error=SalmonIndexHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/00_ref; salmon index -t Argania_spinosa_DR.mrna-transcripts.fa.gz -i Argan_hap1_index"

e. Quantify the samples

         
            #on haplotype 1  
            for FILE in $(ls *1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_Hap1 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.out --error=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/02_TrimmedData ; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/00_ref/Argan_hap1_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/01_SalmonQuant/$(echo $FILE | cut -d'_' -f1)_quant"; sleep  1; done


            # Count nb. of genes with more than zero reads 
            for i in $(ls *_quant/*.sf); do cat $i | awk '$5 >0' | wc -l; done
            # Change name of quant file for all files
            cp 7A_quant/quant.sf 7A_quant.sf
            cp 8A_quant/quant.sf 8A_quant.sf
            cp 9A_quant/quant.sf 9A_quant.sf
            gzip *.sf


3. Similarity between haplotypes.

   a. Make db on protein
            module load BLAST+
            makeblastdb -dbtype prot -in Hap1_A_spinosa.fa -parse_seqids -out db_aa/Hap1_db
            makeblastdb -dbtype prot -in Hap2_A_spinosa.fa -parse_seqids -out db_aa/Hap2_db

   b. Select each of the 1:1 orthologs and blast each id to all proteins

   The 1:1 orthologs between the two haplotypes can be found at the OrhtologousGroups.txt file. This file contains the Orthologoup group assigned, the gene id of the first (hap1), second (hap2) and third sample (Mtruncatula).
   In our case we are only interested on the Orthologous groups showing a match between hap1 and hap2. so we can identify them by exporting the table to excel and filter the columns and selecting only the ones that have an hap1 and hap2 ortholog.

   c. Create script to make a blast for each orthologous group

   The fasta sequencees of each orthologous genes are found in the folder OrthologousGroupsFasta.
   We can blast the OG file against itself and this will give us the similarity value between hap1 and hap2.

   The result of the blast should be:

   Hap1 vs Hap1
   Hap1 vs Hap2
   Hap2 vs Hap1
   Hap2 vs Hap2

   We are interested in the comparison between hap1 and hap2.

            a. creat db for all the files on orthodb

                              for i in $(ls OG6*.fa); do echo $i; /home/imateusgonzalez/00_Software/ncbi-blast-2.15.0+/bin/makeblastdb -dbtype prot -in $i -parse_seqids -out /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/03_Similarity_btw_haplotypes/db_OG/$(echo $i |cut -d'.' -f1)_db; done

            b. We will blast each OMA orthogroup file to the hap2 aa db. Now create a file that contains the instructions for making the blast between each one of the orthologous groups between hap1 et hap2 1:1 agaisnt the hap2 db. Then execute in cluster.
   
                              sbatch --partition=pibu_el8 --job-name=Blast --time=0-10:00:00 --mem-per-cpu=2G --ntasks=1 --cpus-per-task=1 --output=blast.out --error=blast.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/03_Similarity_btw_haplotypes/01_BlastedOG;  bash Command_blast_OG.txt"

            c. Select only the blast result of the OMA ortholog

                     for i in $(ls Blast_*.xml); do echo $i; awk '{ if (($1 ~ /'$(echo $i |cut -d'_' -f4)'/) && ($2 ~ /'$(echo $i |cut -d'_' -f6|cut -d'.' -f1)'/)) { print } }' $i > FinalBlast_$(echo $i |cut -d'.' -f1).xml; done

            d. Sumarize the results in a single table

                  echo '' > ALL_OG_Blast_hap1hap2.xml # Create an empty file
                  for i in $(ls FinalBlast*.xml);do cat $i |head -1 | sed "s/$/,$(echo $i | cut -d'_' -f2)/" | sed 's/[0-9]*.,//' >> ALL_OG_Blast_hap1hap2.xml; done

   d. Plot the results with R.
