## Map long reads with minimap2


# 1. Download the reads

        sbatch --partition=pibu_el8 --job-name=VparadoxaSRA --time=0-21:00:00 --mem-per-cpu=16G --ntasks=48 --cpus-per-task=1 --output=VparadoxaSRA.log --error=VparadoxaSRA.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/02_Vparadoxa/00_RawLongReads; module load SRA-Toolkit; prefetch SRR15663250 ; prefetch SRR15663251 ; prefetch SRR15663252 ; prefetch SRR15663253 ; prefetch SRR15663254 ; prefetch SRR15663255 ; prefetch SRR15663256 ; prefetch SRR15663257 ; prefetch SRR15663258 ; prefetch SRR15663259 ; prefetch SRR15663260 ; prefetch SRR15663261 ; prefetch SRR15663262 ; prefetch SRR15663263 ; prefetch SRR15663264 ; prefetch SRR15663265 ; prefetch SRR15663266 ; prefetch SRR15663267 ; prefetch SRR15663268 ; prefetch SRR15663269 ; prefetch SRR15665707 ; prefetch SRR15665708 ; prefetch SRR15665709 ; prefetch SRR15665720 ; prefetch SRR15665722 "

                sbatch --partition=pibu_el8 --job-name=VparadoxaSRA --time=0-21:00:00 --mem-per-cpu=16G --ntasks=48 --cpus-per-task=1 --output=VparadoxaSRA.log --error=VparadoxaSRA.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/02_Vparadoxa/00_RawLongReads; module load SRA-Toolkit; fasterq-dump --split-files SRR15663250; fasterq-dump --split-files SRR15663251; fasterq-dump --split-files SRR15663252; fasterq-dump --split-files SRR15663253; fasterq-dump --split-files SRR15663254; fasterq-dump --split-files SRR15663255; fasterq-dump --split-files SRR15663256; fasterq-dump --split-files SRR15663257; fasterq-dump --split-files SRR15663258; fasterq-dump --split-files SRR15663259; fasterq-dump --split-files SRR15663260; fasterq-dump --split-files SRR15663261; fasterq-dump --split-files SRR15663262; fasterq-dump --split-files SRR15663263; fasterq-dump --split-files SRR15663264; fasterq-dump --split-files SRR15663265; fasterq-dump --split-files SRR15663266; fasterq-dump --split-files SRR15663267; fasterq-dump --split-files SRR15663268; fasterq-dump --split-files SRR15663269; fasterq-dump --split-files SRR15665707; fasterq-dump --split-files SRR15665708; fasterq-dump --split-files SRR15665709; fasterq-dump --split-files SRR15665720; fasterq-dump --split-files SRR15665722"
        

# 2. Map the reads


           sbatch --partition=pibu_el8 --job-name=VparadoxaSRA --time=0-21:00:00 --mem-per-cpu=16G --ntasks=48 --cpus-per-task=1 --output=VparadoxaSRA.log --error=VparadoxaSRA.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/02_Vparadoxa; module load minimap2; minimap2 -ax map-pb /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/02_BUSCO/05_vitellaria/VitellariaHiCP0_only20.fasta 00_RawLongReads/SRR15663250.fastq > Vparadoxa.sam"

                sbatch --partition=pibu_el8 --job-name=minimapHap1 --time=0-21:00:00 --mem-per-cpu=16G --ntasks=48 --cpus-per-task=1 --output=minimapHap1.log --error=minimapHap1.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/22_MapingLongReads/01_AspinosaHap1; module load minimap2; minimap2 -ax map-pb /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/02_BUSCO/01_hap1/Aspinosa_hap1.fa /data/projects/p782_RNA_seq_Argania_spinosa/20_GenomeAssembly/01_Hifi/Combined_clean.fq > Hap1.sam"

