# Identify orthologs between different haplotypes and species

1. OMA database conversion part

        sbatch --partition=pibu_el8 --job-name=OMApart1 --time=2-00:00:00 --mem-per-cpu=2G --ntasks=1 --cpus-per-task=1 --output=logs/oma1-%J.log --error=logs/oma1-%J.err --mail-type=END,FAIL --wrap "cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/06_Orthologs/OMA.2.6.0; ./bin/oma -c"
   


#!/bin/bash
#SBATCH --partition=<your_partition>
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=2GB
#SBATCH --job-name=oma1
#SBATCH --output=logs/oma1-%J.log
#SBATCH --export=None
#SBATCH --error=logs/oma1-%J.err
cd <full_path_to_OMA.latest>
./bin/oma -c


2. This is the all-against-all part, it is split into 500 parallelized jobs

    #!/bin/bash
#SBATCH --array=1-500
#SBATCH --partition=<your_partition>
#SBATCH --time=2:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=2GB
#SBATCH --job-name=oma2
#SBATCH --output=logs/oma2-%A.%a.log
#SBATCH --export=None
#SBATCH --error=logs/oma2-%A.%a.err
cd <full_path_to_OMA.latest>
export NR_PROCESSES=500
./bin/oma -s -W 7000
if [[ "$?" == "99" ]] ; then
scontrol requeue \
${SLURM_ARRAY_JOB_ID}_${SLURM_ARRAY_TASK_ID}
fi
exit 0

3. This the orthology inference

   #!/bin/bash
#SBATCH --partition=<your_partition>
#SBATCH --time=1:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=50GB
#SBATCH --job-name=oma3
#SBATCH --output=logs/oma3-%J.log
#SBATCH --export=None
#SBATCH --error=logs/oma3-%J.err
cd <full_path_to_OMA.latest>
./bin/oma

