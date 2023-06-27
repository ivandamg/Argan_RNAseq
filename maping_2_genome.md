1. Map RNAseq reads to genome with bbmap



module load UHTS/Analysis/BBMap/38.91 

# index reference

        bbmap.sh ref=/data/projects/p782_RNA_seq_Argania_spinosa/00_Genome/Argan2-publish-files/asm_hifi_Both.p_ctg.fa

 bbmap.sh in1=C3-4_Rirregularis_1_val_1.fq in2=C3-4_Rirregularis_2_val_2.fq out=C3-4_mappedSuperZ17p.sam ref=../00_GenomeAssemblies/SuperAssembly_Z17ALL_primary_ctg.fasta
 
# Modify file names to NAME_Clean.1.fq.gz NAME_Clean.2.fq.gz
 
# map to genome assembly
 
                for FILE in $(ls *A_clean.1.fq.gz); do echo $FILE; sbatch --partition=pall --job-name=$(echo $FILE | cut -d'_' -f1)_bbmap --time=16:00:00 --mem-per-cpu=64G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1)_bbmap.out --error=$(echo $FILE | cut -d'_' -f1)_bbmap.error --mail-type=END,FAIL --wrap "module load UHTS/Analysis/BBMap/38.91;cd /data/projects/p782_RNA_seq_Argania_spinosa/05_MapReadsGenome/;  bbmap.sh in1=/data/projects/p782_RNA_seq_Argania_spinosa/05_MapReadsGenome/$FILE in2=/data/projects/p782_RNA_seq_Argania_spinosa/05_MapReadsGenome/$(echo $FILE | cut -d'.' -f1).2.fq out=$(echo $FILE | cut -d'.' -f1)_mapped_Argan.sam ref=/data/projects/p782_RNA_seq_Argania_spinosa/05_MapReadsGenome/asm_hifi_Both.p_ctg.fa vslow k=8 maxindel=200 minratio=0.1"   ; sleep 1; done

#transform to bam sort and Qfilter

     module load UHTS/Analysis/samtools/1.10
     samtools sort -m4G -@4 -o Z17Hifireads_RefSuperZ17p_sorted.bam Z17Hifireads_RefSuperZ17p.sam
     samtools view -bq 30 Z17Hifireads_RefSuperZ17p_sorted.bam > Z17Hifireads_RefSuperZ17p_sorted_Q30.bam
     samtools index Z17Hifireads_RefSuperZ17p_sorted.bam

