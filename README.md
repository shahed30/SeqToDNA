# SeqToDNA
This is the workflow used for the reconstruction of rDNA and target contigs from high-throughput sequencing data using reference sequences


## Dependencies

### samtools
### sambamba
### spades
### bedtools
### spades
### hisat2
### bwa
### bowtie2
### bbmap


## How to run the workflow

#### Step 1
##### Indexing the reference (Change reference file names)
`hisat2-build ciliate_ref.fasta spirostomum_index`       #(hisat2-build reference.fasta name_index)  

`bwa index -a bwtsw ciliate_ref.fasta`  

`bowtie2-build ciliate_ref.fasta reference_index`  

`bbmap.sh ref=ciliate_ref.fasta`  


#### Step 2
##### Run mapping (Change input file names)
`hisat2 -x spirostomum_index -1 RAW_trimmed__FPE.fastq.gz -2 RAW_trimmed__RPE.fastq.gz -S sample.sam`    #(hisat2 -x name_index -1 FPE.fastq.gz -2 RPE.fastq.gz -S mapped.sam)  

`bwa mem ciliate_ref.fasta RAW_trimmed__FPE.fastq.gz RAW_trimmed__RPE.fastq.gz > sample.sam`  

`bowtie2 -x reference_index -1 RAW_trimmed__FPE.fastq.gz -2 RAW_trimmed__RPE.fastq.gz -S sample.sam`  

`bbmap.sh in=/file_location/RAW_trimmed__FPE.fastq.gz in2=/file_location/RAW_trimmed__RPE.fastq.gz out=sample.sam`  


#### Step 3
##### Convert sam to bam
`samtools view -bS sample.sam > sample.bam`       #(samtools view -bS mapped.sam > mapped.bam)  


#### Step 4
##### Fixing 
`samtools fixmate -O bam sample.bam  fixmate_sample.bam`     #(samtools fixmate -O bam in_file out_file)  


#### Step 5
##### Sorting 
`samtools sort -O bam -o sorted_sample.bam fixmate_sample.bam`     #(samtools sort -O bam -o out_file in_file)  


#### Step 6
##### Remove duplicates
`sambamba markdup -r sorted_sample.bam sorted_sample.dedup.bam`      #(sambamba markdup -r in_file out_file)  


#### Step 7
##### Extract mapped reads only with q40
`samtools view -h -b -q 40 sorted_sample.dedup.bam > sorted_sample.dedup.q40.bam`  #(samtools view -h -b -q 40 in_file > out_file)  


#### Step 8
##### Extract mapped reads only with q20
`samtools view -h -b -q 20 sorted_sample.dedup.bam > sorted_sample.dedup.q20.bam`  


#### Step 9
##### Extract mapped reads only
`samtools view -h -F 4 -b sorted_sample.dedup.bam > onlymapped_sample.dedup.bam`  


#### Step 10
##### fastq to fasta
`bedtools bamtofastq -i sorted_sample.dedup.q20.bam -fq sample_q20.fastq`  

`bedtools bamtofastq -i sorted_sample.dedup.q20.bam -fq sample_q20.fastq`  

`bedtools bamtofastq -i onlymapped_sample.dedup.bam -fq sample_OM.fastq`  


#### Step 11
##### spades assembly
`rnaspades.py -s sample_q40.fastq -o rnaspades_bbmap_q40`  

`rnaspades.py -s sample_q20.fastq -o rnaspades_bbmap_q20`  

`rnaspades.py -s sample_OM.fastq -o rnaspades_bbmap_only_mapped`  

