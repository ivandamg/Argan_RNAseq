# script to identify homology similarity % identity between orthologs of hap1 and hap2

### 1. Extract only 1:1 orthologs

        cat Sspinosum_hap2-Sspinosum_hap1.txt | grep "1:1" > Sspinosum_hap2-Sspinosum_hap1_ONLY_1to1.txt

### 2. Analysis with only 1st isoform


        cat Sspinosum_hap2-Sspinosum_hap1_ONLY_1to1.txt | grep ".t1" > Sspinosum_hap2-Sspinosum_hap1_ONLY_1to1_t1.txt

### 3. Extract seq. id A: Hap2 , B: Hap1

        awk '{print $3}' Sspinosum_hap2-Sspinosum_hap1_ONLY_1to1_t1.txt > A_inorder.txt
        awk '{print $4}' Sspinosum_hap2-Sspinosum_hap1_ONLY_1to1_t1.txt > B_inorder.txt

### 4. extract sequences for A and B preserving order:


        ~/ZZ_Software/seqkit grep -f A_inorder.txt Sspinosum_hap2.fa -o A_inorder.faa 
        ~/ZZ_Software/seqkit grep -f B_inorder.txt Sspinosum_hap1.fa -o B_inorder.faa

### 5. Blast each sequence of A into all B. select only the first hit.
columns output: qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore


                blastp -query A_inorder.faa -subject B_inorder.faa -outfmt 6 -max_target_seqs 1 -num_threads 8 -out blast_pairwise_results.tsv


### 6. Plot and identify in R orthologs that have sequence divergence


### 7. merge hap 1 and hap2

### 8. Map reads with STAR 2 passes


### 9. Count reads with feature counts


### 10 . DEseq with R

