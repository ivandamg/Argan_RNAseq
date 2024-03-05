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
