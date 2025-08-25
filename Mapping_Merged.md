# Pipeline v2. 


# 1. Clean and trim reads with fastp


         mkdir ../02_TrimmedData/
         for FILE in $(ls *1.fastq.gz); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1)fastp --time=0-08:00:00 --mem-per-cpu=24G --ntasks=1 --cpus-per-task=4 --output=$(echo $FILE | cut -d'_' -f1)_fastp.out --error=$(echo $FILE | cut -d'_' -f1)_fastp.error --mail-type=END,FAIL --wrap " cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/01_RawReads ; module load FastQC; module load fastp/0.23.4-GCC-10.3.0; fastp --in1 $FILE --in2 $(echo $FILE | cut -d'_' -f1)_2.fastq.gz --out1 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_1_trimmed.fastq.gz --out2 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -h ../02_TrimmedData/$(echo $FILE | cut -d',' -f1)_fastp.html --detect_adapter_for_pe --trim_poly_g --thread 4; fastqc -t 4 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_1_trimmed.fastq.gz; fastqc -t 4 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz"; sleep  1; done



# 2. Index transcriptome Hap1 and Hap2 independently
copy Annotation into 00_References/hap folder


         mkdir ../00_References/Merged
         #Genome Assembly
         cat ../Hap1/S_spinosum_hap1.fa | sed 's/>/>Hap1_/g' > Merged_Sspinosum.fa
         cat ../Hap2/S_spinosum_hap2.fa | sed 's/>/>Hap2_/g' >> Merged_Sspinosum.fa
         # Annotation gtf
         cat ../Hap1/Sspinosum_hap1.gtf | sed 's/Chr/Hap1_Chr/g'| sed 's/scaf/Hap1_scaf/g' > Merged_Sspinosum.gtf 
         cat ../Hap2/Sspinosum_hap2.gtf | sed 's/Chr/Hap2_Chr/g'| sed 's/scaf/Hap2_scaf/g' >> Merged_Sspinosum.gtf
             # Index
             sbatch --partition=pshort_el8 --job-name=IndexHap1 --time=0-01:00:00 --mem-per-cpu=64G --ntasks=1 --cpus-per-task=1 --output=Hap1Index.out --error=Hap1Index.error --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Merged; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Merged --genomeFastaFiles Merged_Sspinosum.fa --sjdbGTFfile Merged_Sspinosum.gtf --sjdbGTFfeatureExon CDS --sjdbOverhang 99"



# 3. Map reads 


## 1. Hap1

        mkdir ../13_MappedMixed/
       
       for FILE in $(ls *_1_trimmed.fastq.gz ); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_1STAR --time=1-12:00:00 --mem-per-cpu=128G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_STAR.out --error=$(echo $FILE | cut -d'_' -f1)_STAR.error --mail-type=END,FAIL --wrap "module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/02_TrimmedData; STAR --runThreadN 8 --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap1 --readFilesIn $FILE $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz --readFilesCommand zcat --outFileNamePrefix $(echo $FILE | cut -d'_' -f1)_Merged_ --outSAMtype BAM SortedByCoordinate --outReadsUnmapped Fastx"; sleep  1; done



# 3. Create the SJ.out.tab by filtering and merging SJ.out.tab files from all runs

    
    mkdir Merged_2ndPass #(do not create this folder in the folder "RNAseq")
    echo "" > Merged_2ndPass/Global.SJ.out.tab.Pass2.sjdb
    for i in $(ls *Merged_SJ.out.tab); do echo $i; awk 'BEGIN {OFS="\t"; strChar[0]="."; strChar[1]="+"; strChar[2]="-";} {if($5>0){print $1,$2,$3,strChar[$4]}}' $i >> /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Merged/Merged_2ndPass/Global.SJ.out.tab.Pass2.sjdb ; done

# 4. Remove bam and unmapped reads

                  rm *Aligned.sortedByCoord.out.bam
                  rm *Unmapped.out.mate1
                  rm *Unmapped.out.mate2


# 4. Index the genome with STAR for 2nd-pass mapping
copy the genome and gtf file into 2nd pass folder


## 1. Hap1
    
             sbatch --partition=pshort_el8 --job-name=IndexHap1 --time=0-01:00:00 --mem-per-cpu=64G --ntasks=1 --cpus-per-task=1 --output=Hap1Index.out --error=Hap1Index.error --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap1/2ndPass; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap1/2ndPass --genomeFastaFiles S_spinosum_hap1.fa --sjdbGTFfile Sspinosum_hap1.gtf --sjdbGTFfeatureExon CDS --sjdbOverhang 99 --sjdbFileChrStartEnd Global.SJ.out.tab.Pass2.sjdb"

## 2. Hap2
    
             sbatch --partition=pshort_el8 --job-name=IndexHap2 --time=0-01:00:00 --mem-per-cpu=64G --ntasks=1 --cpus-per-task=1 --output=Hap2Index.out --error=Hap2Index.error --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap2/2ndPass; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap2/2ndPass --genomeFastaFiles S_spinosum_hap2.fa --sjdbGTFfile Sspinosum_hap2.gtf --sjdbGTFfeatureExon CDS --sjdbOverhang 99 --sjdbFileChrStartEnd Global.SJ.out.tab.Pass2.sjdb"



# 3. Map reads 


## 1. Hap1

       mkdir ../03_Mapped/
       
       for FILE in $(ls *_1_trimmed.fastq.gz ); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_1STAR --time=1-12:00:00 --mem-per-cpu=128G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_STAR.out --error=$(echo $FILE | cut -d'_' -f1)_STAR.error --mail-type=END,FAIL --wrap "module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/02_TrimmedData; STAR --runThreadN 8 --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap1/2ndPass --readFilesIn $FILE $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz --readFilesCommand zcat --outFileNamePrefix $(echo $FILE | cut -d'_' -f1)_Hap1_ --outSAMtype BAM SortedByCoordinate --outReadsUnmapped Fastx"; sleep  1; done

## 1. Hap2
       
       for FILE in $(ls *_1_trimmed.fastq.gz ); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_1STAR --time=1-12:00:00 --mem-per-cpu=128G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_STAR.out --error=$(echo $FILE | cut -d'_' -f1)_STAR.error --mail-type=END,FAIL --wrap "module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; cd /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/02_TrimmedData; STAR --runThreadN 8 --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap2/2ndPass --readFilesIn $FILE $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz --readFilesCommand zcat --outFileNamePrefix $(echo $FILE | cut -d'_' -f1)_Hap2_ --outSAMtype BAM SortedByCoordinate --outReadsUnmapped Fastx"; sleep  1; done



# 4. Count features with featureCounts

## 1. Hap1

          sbatch --partition=pshort_el8 --job-name=FC_Hap1 --time=0-02:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=FC_Hap1.out --error=FC_Hap1.error --mail-type=END,FAIL --wrap "module load Subread/2.0.3-GCC-10.3.0; featureCounts -p --countReadPairs -t CDS -g gene_id -a /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap1/Sspinosum_hap1.gtf  -o FC_Hap1_ALL.txt 1A_Hap1_Aligned.sortedByCoord.out.bam 2A_Hap1_Aligned.sortedByCoord.out.bam 3A_Hap1_Aligned.sortedByCoord.out.bam 4A_Hap1_Aligned.sortedByCoord.out.bam 5A_Hap1_Aligned.sortedByCoord.out.bam 6A_Hap1_Aligned.sortedByCoord.out.bam 7A_Hap1_Aligned.sortedByCoord.out.bam 8A_Hap1_Aligned.sortedByCoord.out.bam 9A_Hap1_Aligned.sortedByCoord.out.bam 10A_Hap1_Aligned.sortedByCoord.out.bam 11A_Hap1_Aligned.sortedByCoord.out.bam 12A_Hap1_Aligned.sortedByCoord.out.bam -T 8"


         sbatch --partition=pshort_el8 --job-name=FC_Hap1 --time=0-02:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=FC_Hap1.out --error=FC_Hap1.error --mail-type=END,FAIL --wrap "module load Subread/2.0.3-GCC-10.3.0; featureCounts -p --countReadPairs -t exon -g transcript_id  -a /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap1/Sspinosum_hap1.gtf  -o FC_Hap1_ALL.txt 1A_Hap1_Aligned.sortedByCoord.out.bam 2A_Hap1_Aligned.sortedByCoord.out.bam 3A_Hap1_Aligned.sortedByCoord.out.bam 4A_Hap1_Aligned.sortedByCoord.out.bam 5A_Hap1_Aligned.sortedByCoord.out.bam 6A_Hap1_Aligned.sortedByCoord.out.bam 7A_Hap1_Aligned.sortedByCoord.out.bam 8A_Hap1_Aligned.sortedByCoord.out.bam 9A_Hap1_Aligned.sortedByCoord.out.bam 10A_Hap1_Aligned.sortedByCoord.out.bam 11A_Hap1_Aligned.sortedByCoord.out.bam 12A_Hap1_Aligned.sortedByCoord.out.bam -T 8"

## 2. Hap2
         
          sbatch --partition=pshort_el8 --job-name=FC_Hap2 --time=0-02:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=FC_Hap2.out --error=FC_Hap2.error --mail-type=END,FAIL --wrap "module load Subread/2.0.3-GCC-10.3.0; featureCounts -p --countReadPairs -t CDS -g gene_id -a /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap2/Sspinosum_hap2.gtf  -o FC_Hap2_ALL.txt 1A_Hap2_Aligned.sortedByCoord.out.bam 2A_Hap2_Aligned.sortedByCoord.out.bam 3A_Hap2_Aligned.sortedByCoord.out.bam 4A_Hap2_Aligned.sortedByCoord.out.bam 5A_Hap2_Aligned.sortedByCoord.out.bam 6A_Hap2_Aligned.sortedByCoord.out.bam 7A_Hap2_Aligned.sortedByCoord.out.bam 8A_Hap2_Aligned.sortedByCoord.out.bam 9A_Hap2_Aligned.sortedByCoord.out.bam 10A_Hap2_Aligned.sortedByCoord.out.bam 11A_Hap2_Aligned.sortedByCoord.out.bam 12A_Hap2_Aligned.sortedByCoord.out.bam -T 8"


          sbatch --partition=pshort_el8 --job-name=FC_Hap2 --time=0-02:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=FC_Hap2.out --error=FC_Hap2.error --mail-type=END,FAIL --wrap "module load Subread/2.0.3-GCC-10.3.0; featureCounts -p --countReadPairs -t exon -g transcript_id -a /data/projects/p782_RNA_seq_Argania_spinosa/500_RNAseqV3/00_References/Hap2/Sspinosum_hap2.gtf  -o FC_Hap2_ALL.txt 1A_Hap2_Aligned.sortedByCoord.out.bam 2A_Hap2_Aligned.sortedByCoord.out.bam 3A_Hap2_Aligned.sortedByCoord.out.bam 4A_Hap2_Aligned.sortedByCoord.out.bam 5A_Hap2_Aligned.sortedByCoord.out.bam 6A_Hap2_Aligned.sortedByCoord.out.bam 7A_Hap2_Aligned.sortedByCoord.out.bam 8A_Hap2_Aligned.sortedByCoord.out.bam 9A_Hap2_Aligned.sortedByCoord.out.bam 10A_Hap2_Aligned.sortedByCoord.out.bam 11A_Hap2_Aligned.sortedByCoord.out.bam 12A_Hap2_Aligned.sortedByCoord.out.bam -T 8"



#-----------------------

2. transcript quantification with salmon

https://combine-lab.github.io/salmon/getting_started/

a. obtain salmon

            module load UHTS/Analysis/salmon/0.11.2

b. build an index on the transcriptome

            
            sbatch --partition=pshort_el8 --job-name=SalmonIndex --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=SalmonIndexHap1.out --error=SalmonIndexHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/01_Hap1; salmon index -t Aspinosa_hap1_cds.fasta -i Argan_hap1_index"

         sbatch --partition=pshort_el8 --job-name=2SalmonIndex --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=SalmonIndexHap2.out --error=SalmonIndexHap2.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/02_Hap2; salmon index -t Aspinosa_hap2_cds.fasta -i Argan_hap2_index"

c. Quantify the samples

         
            #on haplotype 1  
            for FILE in $(ls *1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_Hap1 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.out --error=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/00_trimmedReads ; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/01_Hap1/Argan_hap1_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/01_Hap1/01_SalmonQuant/$(echo $FILE | cut -d'_' -f1)_quant"; sleep  1; done

            #on haplotype 2  
            for FILE in $(ls *1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_Hap2 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_SalmonHap2.out --error=$(echo $FILE | cut -d'_' -f1)_SalmonHap2.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/00_trimmedReads ; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/02_Hap2/Argan_hap2_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/02_Hap2/01_SalmonQuant/$(echo $FILE | cut -d'_' -f1)_quant"; sleep  1; done


d. mix the transcriptome together and build an index on the transcriptome
Change name of reads for hap2

                  cat ../01_Hap1/Aspinosa_hap1_aa.fasta | sed 's/>/>hap1_/' > Aspinosa_MixHap1Hap2_aa.fasta
                  cat ../02_Hap2/Aspinosa_hap2_aa.fasta | sed 's/>/>hap2_/' >> Aspinosa_MixHap1Hap2_aa.fasta
            
            sbatch --partition=pshort_el8 --job-name=SalmonIndex3 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=SalmonIndexHapMIX.out --error=SalmonIndexHapMIX.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/03_MixHap1Hap2; salmon index -t Aspinosa_MixHap1Hap2_cds.fasta -i Argan_hapMix_index"

e. Quantify the samples

         
            #on haplotype Mix  
            for FILE in $(ls *1_trimmed.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_Hap1 --time=0-02:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.out --error=$(echo $FILE | cut -d'_' -f1)_SalmonHap1.error --mail-type=END,FAIL --wrap "module load Salmon; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/00_trimmedReads ; salmon quant -i /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/03_MixHap1Hap2/Argan_hapMix_index -l A -1 $FILE -2 $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz -p 8 --validateMappings -o /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/10_RNAseq/03_MixHap1Hap2/01_SalmonQuant/$(echo $FILE | cut -d'_' -f1)_quant"; sleep  1; done


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
