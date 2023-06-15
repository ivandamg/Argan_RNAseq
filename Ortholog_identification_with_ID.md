 # This script allows to identify orthologs based on a list of genes.
 
 The analysis will consist on reciprocal blasts betweem the gene of interest and the target genome.
 
 
 
 1. Identification of the gene homologs on a related assembly.

    1. Make protein db with the transcriptomes

   makeblastdb -dbtype prot -in Medicago_truncatula.fa -parse_seqids -out db_Mtrunactula_aa/Mtruncatula_db
   makeblastdb -dbtype prot -in Argania_spinosa_v1.fa -parse_seqids -out db_Aspinosa_aa/Aspinosa_db

    2. Blastp REFERENCE proteins TO new assemblies proteins.

    a=0;for i in $(ls *.faa); do echo $(echo $i | cut -d'_' -f1,2,3) ;blastp -db db_Prokka_protein/$(echo $i | cut -d'_' -f1,2,3)_db -outfmt 6 -evalue 1e-8 -show_gis -num_alignments 1 -max_hsps 20 -num_threads 30 -out db_Prokka_protein/blast_$(echo $i | cut -d'_' -f1,2,3).xml -query REF_*/*.faa ; done
    blastp -db db_Mtrunactula_aa/Mtruncatula_db -outfmt 6 -evalue 1e-8 -show_gis -num_alignments 1 -max_hsps 20 -num_threads 30 -out blast_PT4_Mtruncatula.xml -query PT4_Mtruncatula_AAM76743.fasta

    3. Extract target matched sequence from transcriptome file 
    
    awk '/^>/{f=0} /argan2.00g034710/{f=1} f' Argania_spinosa_v1.fa
    
    
    4. create db on target M truncatula genome
    
    
    5. reciprocal blast on original M truncatula genome
    
Method 2.

1. Select 10 AMS genes from Bravo et al.,2017
2. blast all the genes to Argan transcriptome
3. Filter blast hits with %similarity >50%
4. select the gene id and Look at the Dataset if Differentially expressed and make a table.
5. Redo for Drought genes. (find list of conserved drought genes.
   
    
