


# ChromSyn -Create syntheny plots between chromosomes
https://github.com/slimsuite/chromsyn/blob/main/Walkthrough.md


#1.  telociraptor in local

    for GENOME in *.fasta; do PREFIX=$(basename ${GENOME/.fasta/}) ; python /Users/mateusgo/ZZ_Software/telociraptor/code/telociraptor.py seqin=$GENOME basefile=$PREFIX i=-1 tweak=F telonull=T ;done


#2.  Busco in cluster


    sbatch --partition=pibu_el8 --job-name=hap1Busco --time=0-10:00:00 --mem-per-cpu=50G --ntasks=12 --cpus-per-task=1 --output=BuscoHap1.out --error=BuscoHap1.error --mail-type=END,FAIL --wrap "module load BUSCO; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/01_Assembly/01_hap1; busco -o /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/02_BUSCO/01_hap1/ -i Aspinosa_hap1.fa -l eudicots_odb10 --cpu 12 -m genome -f"

    sbatch --partition=pibu_el8 --job-name=hap2Busco --time=0-10:00:00 --mem-per-cpu=50G --ntasks=12 --cpus-per-task=1 --output=BuscoHap2.out --error=BuscoHap2.error --mail-type=END,FAIL --wrap "module load BUSCO; cd /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/01_Assembly/02_hap2; busco -o /data/projects/p782_RNA_seq_Argania_spinosa/30_FinalAssemblyPaper/02_BUSCO/02_hap2 -i Aspinosa_hap2.fa -l eudicots_odb10 --cpu 12 -m genome -f"

    sbatch --partition=pibu_el8 --job-name=QLODBusco --time=0-10:00:00 --mem-per-cpu=50G --ntasks=12 --cpus-per-task=1 --output=QLODBusco.out --error=QLODBusco.error --mail-type=END,FAIL --wrap "module load BUSCO; cd /data/projects/p782_RNA_seq_Argania_spinosa/21_RNAseqV2/07_Busco/QLOD/; busco -o QLOD -i Max1M_QLOD.fasta -l eudicots_odb10 --cpu 12 -m genome -f"

#2b. Consolidate Busco result

    for GENOME in *.fasta; do PREFIX=$(basename ${GENOME/.fasta/}); cp -v run_$PREFIX/run_mammalia_odb10/full_table.tsv $PREFIX.busco5.tsv ; done

#3. tidk in local

    tidk search hap1.fasta -o hap1 -s AACCCT --dir hap1;cp -v hap1/hap1_telomeric_repeat_windows.tsv hap1/hap1.tidk.csv; cp hap1/hap1.tidk.csv .

    tidk search hap2.fasta -o hap2 -s AACCCT --dir hap2;cp -v hap2/hap2_telomeric_repeat_windows.tsv hap2/hap2.tidk.csv; cp hap2/hap2.tidk.csv .

# 4. move the files into a single directory

i.e example/gendata/
─ ensDEVIL.busco5.tsv
─ ensDEVIL.gaps.tdt
─ ensDEVIL.telomeres.tdt
─ ensDEVIL.tidk.csv
─ ensMONDO.busco5.tsv
─ ensMONDO.gaps.tdt
─ ensMONDO.telomeres.tdt
─ ensMONDO.tidk.csv
─ ensPLATY.busco5.tsv
─ ensPLATY.gaps.tdt
─ ensPLATY.telomeres.tdt
─ ensPLATY.tidk.csv

#5. generate *.fofn

    ls *.busco5.tsv | sed -E 's#([A-Za-z]+)\.#\1 gendata/\1.#' | tee ../busco.fofn
    ls *.gaps.tdt | sed -E 's#([A-Za-z]+)\.#\1 gendata/\1.#' | tee ../gaps.fofn
    ls *.telomeres.tdt | sed -E 's#([A-Za-z]+)\.#\1 gendata/\1.#' | tee ../sequences.fofn
    ls *.tidk.csv | sed -E 's#([A-Za-z]+)\.#\1 gendata/\1.#' | tee ../tidk.fofn

# 6. run crhomsym

    Rscript ../chromsyn.R
    Rscript ../chromsyn.R | tee chromsyn.log
