# Identify orthologs between different haplotypes and species

1. OMA database conversion part

        sbatch --partition=pibu_el8 --job-name=OMA1 --time=0-01:00:00 --mem-per-cpu=2G --ntasks=1 --cpus-per-task=1 --output=logs/oma1-%J.log --error=logs/oma1-%J.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/01_plantSpecies/OMA.2.6.0; ./bin/oma -c"
   


2. All-against-all part, ortholog id. parallelized


        sbatch --array=1-500 --partition=pibu_el8 --job-name=OMA2 --time=0-04:00:00 --mem-per-cpu=2G --nodes=1 --ntasks=1 --cpus-per-task=1 --output=logs/oma1-%J.log --error=logs/oma1-%J.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/01_plantSpecies/OMA.2.6.0; export NR_PROCESSES=500; ./bin/oma -s -W 7000;if [[ "$?" == "99" ]] ;then scontrol requeue ${SLURM_ARRAY_JOB_ID}_${SLURM_ARRAY_TASK_ID};fi ;exit 0 "

3. Ortholog inference part

           sbatch --partition=pibu_el8 --job-name=OMA3 --time=0-06:00:00 --mem-per-cpu=50G --ntasks=1 --cpus-per-task=1 --output=logs/oma3-%J.log --error=logs/oma3-%J.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/01_plantSpecies/OMA.2.6.0; ./bin/oma"

