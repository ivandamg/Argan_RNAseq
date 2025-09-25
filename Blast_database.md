# Commands to blast a sequence to a ref database

# blastp

makeblastdb -dbtype prot -in ../04_GenomeAnnotation/01_hap1/Sspinosum_hap1.faa -parse_seqids -out db/Hap1_db
makeblastdb -dbtype prot -in ../04_GenomeAnnotation/02_hap2/Sspinosum_hap2.faa -parse_seqids -out db/Hap2_db

blastp -db /Users/mateusgo/Documents/40_RNAseqPaper/03_MycorrhizalGenes/db/Mtruncatula_db -outfmt 6 -out DMI1/Blast_DMI1_Medicago.xml -query /Users/mateusgo/Documents/40_RNAseqPaper/03_MycorrhizalGenes/DMI1/DMI1.fa 
