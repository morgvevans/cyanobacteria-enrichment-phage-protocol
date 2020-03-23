# cyanobacteria-enrichment-phage-protocol

**This is the protocol used on the cyanophage project, for identifying microbial signatures in the sample, and identifying and clustering viral genomes**

*Step 1 From RAW Reads, use SingleM to extract single copy genes for an "OTU" table reconstruction*
```
#PBS -N cyanobacteria_singlem_03122020
#PBS -A PAS1331
#PBS -l walltime=8:00:00
#PBS -l nodes=1:ppn=6
#PBS -j oe
#PBS -m abe


cd /fs/project/PAS1331/cyanobacteria

singularity run docker://wwood/singlem:v0.13.0 pipe --sequences ./CLEAN_READS/cyanobacteria_concat.fastq  --otu_table OTU_table_singleM_03122020.csv --threads 12
singularity run docker://wwood/singlem:v0.13.0 summarise --input_otu_tables OTU_table_singleM_03122020.csv --biom_prefix OTU_table_singleM_03122020

source activate qiime2-2020.2
biom convert -i OTU_table_singleM_03122020.S1.3.ribosomal_protein_L5_rplE.biom -o cyanobacteria_otutable.txt --to-tsv
```

**Step 2 Assembly (these reads are single end, and already trimmed, so skipping that step but make sure to do trimming and host read removal if needed)**
```
#PBS -N cyano_assembly
#PBS -A PAS1331
#PBS -l walltime=12:00:00
#PBS -l nodes=1:ppn=14
#PBS -j oe
#PBS -m abe



cd /fs/project/PAS1331/cyanobacteria

source activate metawrap-env

megahit -r cyanobacteria.fastq -o cyanobacteria_megahit_assembly -t 28 --min-contig-len 1000
```

**Step 3 Run VIBRANT to ID viral contigs**
```
#PBS -N cyanobacteria_VIBRANT_03122020
#PBS -A PAS1331
#PBS -l walltime=8:00:00
#PBS -l nodes=1:ppn=6
#PBS -j oe
#PBS -m abe



cd /fs/project/PAS1331/cyanobacteria

source activate vibrant-viral-env
VIBRANT_run.py -i ./cyanobacteria_megahit_assembly/final.contigs.fa -t 12 -folder ./VIBRANT_out
```

**Step 4 Use VContact2 to cluster viral contigs. Before you run this script make sure to generate a gene2genome.csv file using de.cyverse.org/de using the VContact2Gene2Genome app, after running any fna files through prodigal to get faa**
```
#PBS -N cyanophage_vcontact2_VIBRANT
#PBS -A PAS1331
#PBS -l walltime=4:00:00
#PBS -l nodes=1:ppn=10
#PBS -j oe
#PBS -m abe



cd /fs/project/PAS1331/cyanobacteria


singularity run /users/PAS1117/osu9664/eMicro-Apps/vConTACT2-0.9.9.sif -r final.contigs.phages_combined_03232020.faa -p vcontact_g2g_03232020.csv -t 10
```


