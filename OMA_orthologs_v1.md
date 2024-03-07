# Identify orthologs between different haplotypes and species

1. OMA database conversion part

        sbatch --partition=pibu_el8 --job-name=OMApart1 --time=2-00:00:00 --mem-per-cpu=2G --ntasks=1 --cpus-per-task=1 --output=logs/oma1-%J.log --error=logs/oma1-%J.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/OMA.2.6.0; ./bin/oma -c"
   


2. This is the all-against-all part, ortholog id.


        sbatch --partition=pibu_el8 --job-name=OMApart1 --time=2-00:00:00 --mem-per-cpu=2G --ntasks=1 --cpus-per-task=1 --output=logs/oma1-%J.log --error=logs/oma1-%J.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/OMA.2.6.0; ./bin/oma"
