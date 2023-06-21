# This script shows how to screen different bacterial genomes with mash on the samples



1. Select reference database.

We will use as bacterial reference, the RefSeq representative dataset of bacteria. the RedfSeq representative dataset is "A comprehensive, integrated, non-redundant, well-annotated set of reference sequences"

In total we will use 4385 different bacterial genomes that should be representative of the bacterial kingdom.

To download all the bacterial genome, can be done in the ncbi web site. by searching: Bacteria, and selecting the filters: complete genome and RefSeq representative.


2. Index the reference.

mash sketch -o reference genome1.fna genome2.fna
mash info reference.msh
mash dist reference.msh genome3.fna


3. Screen genomes against MASH refSeq db The fields ouptut are [identity, shared-hashes, median-multiplicity, p-value, query-ID, query-comment]:

            for i in $(ls *.fastq.gz); do echo $i; ~/00_Software/mash-Linux64-v2.3/mash screen -w -p 8 /data/projects/p782_RNA_seq_Argania_spinosa/11_Mycorrhizal_containement/AMF_GeneBank_ref.msh $i > /data/projects/p782_RNA_seq_Argania_spinosa/11_Mycorrhizal_containement/Screen_$(echo $i | cut -d'.' -f1).out ; done


            # Sort and add filename in last column
            for f in Screen*; do echo $f; sort -gr $f | sed -i "s/$/\t$f/" $f; done    
                    
4. Concatenate all files into a table

5. See more common

 for i in $(ls Screen_*.out); do echo $i; sort -gr $i | head ; done

   
   IN loop
   
  for i in $(ls *.fna.gz); do echo $i; /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/Soft/mash-Linux64-v2.2/mash screen -w -p 16 /scratch/wally/FAC/FBM/DEE/isanders/popgen_to_var/IM/10_Contaminants/00_GeneBank_Complete_Bacteria/Reference_GeneBank_2549_Bacteria_completeGenomes.msh $i > ../03_BacteriaContaminants_Glomeromycetes/Screen_RefSeqk21S1000_$(echo $i | cut -d'_' -f1,2,3).tab ; done 
