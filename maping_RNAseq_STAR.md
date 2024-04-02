1. Map RNAseq reads to genome with STAR
 STAR https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf


a. Index reference

              sbatch --partition=pshort_el8 --job-name=StarIndex --time=0-01:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=StarIndex.out --error=StarIndex.error --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/02_STAR_MappingReads/01_STARindex/; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/02_STAR_MappingReads/01_STARindex --genomeFastaFiles hap1.fasta --sjdbGTFfile Argania_spinosa_DR.gff3 --sjdbOverhang 99 --genomeSAindexNbases 10"
              
              sbatch --partition=pshort_el8 --job-name=Star2Index --time=0-01:00:00 --mem-per-cpu=12G --ntasks=1 --cpus-per-task=1 --output=Star2Index.out --error=Star2Index.error --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/04_haplotype2/02_STAR_MappingReads/01_STARindex/; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/04_haplotype2/02_STAR_MappingReads/01_STARindex --genomeFastaFiles hap2.fasta --sjdbGTFfile Argania_spinosa_DR.gff3 --sjdbOverhang 99 --genomeSAindexNbases 10"

### b. Map reads

              for FILE in $(ls *_1_trimmed.fastq.gz ); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1)_STAR --time=0-09:00:00 --mem-per-cpu=128G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_STAR.out --error=$(echo $FILE | cut -d'_' -f1)_STAR.error --mail-type=END,FAIL --wrap "module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/02_TrimmedData; STAR --runThreadN 8 --genomeDir /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/03_haplotype1/02_STAR_MappingReads/01_STARindex --readFilesIn $FILE $(echo $FILE | cut -d'_' -f1)_2_trimmed.fastq.gz --readFilesCommand zcat --outFileNamePrefix STAR_Hap1_$(echo $FILE | cut -d'_' -f1)_Mapped --outSAMtype BAM SortedByCoordinate --limitBAMsortRAM 1065539232"; sleep  1; done

#transform to bam sort and Qfilter

     module load UHTS/Analysis/samtools/1.10
     samtools sort -m4G -@4 -o Z17Hifireads_RefSuperZ17p_sorted.bam Z17Hifireads_RefSuperZ17p.sam
     samtools view -bq 30 Z17Hifireads_RefSuperZ17p_sorted.bam > Z17Hifireads_RefSuperZ17p_sorted_Q30.bam
     samtools index Z17Hifireads_RefSuperZ17p_sorted.bam

