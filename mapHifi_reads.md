# Script to map hifi reads to genome assemblies


1. mapping
format:    minimap2 -ax map-hifi genome.fasta hifi_reads.fastq > output.sam

   
        sbatch --partition=pibu_el8 --job-name=minimap1 --time=0-10:00:00 --mem-per-cpu=50G --ntasks=1 --cpus-per-task=1 --output=minimap1.out --error=minimap1.error --mail-type=END,FAIL --wrap "module load minimap2; cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/03_CleanReads_Haplotigs; minimap2 -ax map-hifi /data/projects/p782_RNA_seq_Argania_spinosa/00_ArganiaSpinosaRef/00_FinalAssembly/Aspinosa_hap1.fa /data/projects/p782_RNA_seq_Argania_spinosa/03_PreviousGenomeAssemblyAttemps/20_GenomeAssembly/01_Hifi/Combined_clean.fq > CombinedClean_Aspinosa_hap1.sam"
        sbatch --partition=pibu_el8 --job-name=minimap2 --time=0-10:00:00 --mem-per-cpu=50G --ntasks=1 --cpus-per-task=1 --output=minimap2.out --error=minimap2.error --mail-type=END,FAIL --wrap "module load minimap2; cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/03_CleanReads_Haplotigs; minimap2 -ax map-hifi /data/projects/p782_RNA_seq_Argania_spinosa/00_ArganiaSpinosaRef/00_FinalAssembly/Aspinosa_hap2.fa /data/projects/p782_RNA_seq_Argania_spinosa/03_PreviousGenomeAssemblyAttemps/20_GenomeAssembly/01_Hifi/Combined_clean.fq > CombinedClean_Aspinosa_hap2.sam"


2. Sort with samtool and index
samtools sort -o output.bam output.sam;samtools index output.bam

        sbatch --partition=pibu_el8 --job-name=Samtools1 --time=0-10:00:00 --mem-per-cpu=50G --ntasks=1 --cpus-per-task=1 --output=samtools1.out --error=samtools1.error --mail-type=END,FAIL --wrap "module load SAMtools; cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/03_CleanReads_Haplotigs; samtools sort -o CombinedClean_Aspinosa_hap1.bam CombinedClean_Aspinosa_hap1.sam; samtools index CombinedClean_Aspinosa_hap1.bam"
        sbatch --partition=pibu_el8 --job-name=Samtools2 --time=0-10:00:00 --mem-per-cpu=50G --ntasks=1 --cpus-per-task=1 --output=samtools2.out --error=samtools2.error --mail-type=END,FAIL --wrap "module load SAMtools; cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/03_CleanReads_Haplotigs; samtools sort -o CombinedClean_Aspinosa_hap2.bam CombinedClean_Aspinosa_hap2.sam; samtools index CombinedClean_Aspinosa_hap2.bam"




