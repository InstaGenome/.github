= InstaGenome - super-fast genome analysis software for whole genome sequencing data

== Introduction

InstaGenome is super-fast genome analysis software that produces an identical
result to the GATK best practices with a >20x speed on most CPU servers without
using an additional hardware such as GPU or FPGA.

InstaGenome includes BWA, MarkIlluminaAdapters, MergeBamAlignment,
MarkDuplicatesSpark, BaseRecalibrator, and ApplyBQSR of the GATK best practices.
The current version of InstaGenome processes short reads (75bp ~ 300bp).

InstaGenome is proprietary software of Genome4me, Inc. Please read the file
LICENSE for the licensing policy of InstaGenome. The InstaGenome BWA module is
based on BWA Apache license version.


= How to use

== Quick guide

Activate an InstaGenome license, index a reference FASTA file and then execute
InstaGenome binary to produce an analysis-ready SAM/BAM file.

```
# 1. Activate a license
$ instagenome license
License activation required.
Please enter a product key (xxxx-xxxx-xxxx-xxxx)
> ****-****-****-****

License activated successfully.

Licensed to: ********
License type: Time-based license
Valid until: ****/**/** [UTC]
Activation ID: *****-*****-*****-*****-*****-*

# 2. Index a reference
$ instagenome index <reference fasta>

# 3. Execute InstaGenome
$ instagenome pre_process --mark_illumina_adapter --fix_alignment --mark_dup \
  --bqsr <reference fasta> <read1.fastq> [<read2.fastq>]
```

== How to activate a license

All InstaGenome commands require a valid license except 'version' and 'help'
commands. The user should activate a license before executing InstaGenome.
If InstaGenome cannot find a valid license, a procecss to activate a license
will begin.

The user can activate a license by using 'license' command and typing a product
key as follows:
```
$ instagenome license
License activation required.
Please enter a product key (xxxx-xxxx-xxxx-xxxx)
> ****-****-****-****

License activated successfully.

Licensed to: ********
License type: Time-based license
Valid until: ****/**/** [UTC]
Activation ID: *****-*****-*****-*****-*****-*
```

Since the user can activate a license on a limited number of machines, the
license should be activated carefully. If the user wants to move the license
to another machine or replace the hardware of the marchine, the old license
should be deactivated so as to decrement the number of activated licenses.
A license can be deactivated as follows:
```
$ instagenome license -d
License deactivated successfully.
```


== How to execute InstaGenome

1. Index the reference FASTA file.
```
$ instagenome index <reference fasta>
```

2. Execute InstaGenome to produce an analysis-ready SAM/BAM file.
```
$ instagenome pre_process --mark_illumina_adapter --fix_alignment --mark_dup \
  --bqsr <reference fasta> <read1.fastq> [<read2.fastq>]
```

'pre_process' command performs followings:
- By default, the command aligns query sequences using improved BWA-MEM.
- 'mark_illumina_adapter' option is equivalent to GATK MarkIlluminaAdapter
  module.
- 'fix_alignment' option is equivalent to GATK MergeBamAlignemt module.
- 'mark_dup' option is equivalent to GATK MarkDuplicatesSpark module.
- 'bqsr' option is equivalelnt to GATK BaseRecalibrator and ApplyBQSR
  modules.

The 'pre_process' command automatically configures the number of threads and
memory usage based on the number of available processors and memory amount of
a system. The user can explicitly override the number of threads by using
'--threads' option and also can control the memory usage with '--high-memory'
or '--low-memory' option. The '--high-memory' option forces InstaGenome to use
more memory so as to run faster with higher risk of out-of-memory, while the
'--low-memory' option forces InstaGenome to use less memory so as to avoid
out-of-memory.

The 'pre_process' command also accepts existing 'BWA mem' options such as
'-Y' or '-K'. 'instagenome --help' and 'instagenome pre_process --help'
list available commands and options.

= Minimum System Requirement

InstaGenome requires the following linux distributions:
- Rocky Linux 8.8
- CentOS 7.9
- Ubuntu 22.04 LTS Server / Desktop
- Ubuntu 20.04 LTS Server / Desktop
  - The user should install the curl package on Ubuntu 20.04 LTS Desktop:
    - sudo apt install curl

InstaGenome may also run on other linux distributions if the following
libraries are installed:
- libbz2
- libcurl
- liblzma
- libz

InstaGenome requires the following minimum hardware specifications to properly
process 30x human WGS (Whole Genome Sequencing) FASTQ files.
- CPU: x86_64 processors
- RAM: 64 GB
- Disk: 4 times input FASTQ files' size

For input FASTQ files larger than 30x human WGS data, InstaGenome may need
larger memory than 64 GB. Because InstaGenome leverages the most of system
memory to execute faster, only a single InstaGenome process should be executed
at a time.

Please make sure that there is enough free disk space 4 times the size of input
FASTQ files because InstaGenome stores intermediate data into temporary files.


= GATK compatible output

By examining >250 human WGS datasets, we confirmed that InstaGenome produces
SAM output, mark-duplicate metrics, and BQSR report files identical to those of
the GATK best-practice pipeline. In our test, we used GATK best-practice GCP
(Google Cloud Platform) v3.1.11 and the following software versions:
- GATK 4.3
- Picard 2.26.10
- BWA 0.7.15-r1140
- HTSJDK 3.0.1

== Detailed GATK steps

Here are the detailed steps of the GATK pipeline we tested:
- FastqToSam
- MarkIlluminaAdapters
- SamToFastq
- BWA
- MergeBamAlignment
- MarkDuplicatesSpark
- BaseRecalibrator
- ApplyBQSR

The GATK GCP pipeline executes BaseRecalibrator and ApplyBQSR steps in multiple
processes by dividing the whole genome into tens of intervals to run faster.
However, the output results of the multi-process BaseRecalibrator can be
slightly different from the output files of a single-process BaseRecalibatrator
because of random numbers generated in the middle of process. To remove the
random fluctuations in our test, we executed the GATK BaseRecalibrator and
ApplyBQSR in a single process and confirmed that the output files of InstaGenome
are identical to those of the GATK BaseRecalibrator and ApplyBQSR.

Here are the detailed commands and arguments we executed for our test:
'''
# InstaGenome
$ instagenome pre_process -Y -K 100000000 \
  --sort_input --mark_illumina_adapter --fix_alignment --mark_dup --bqsr \
  --bqsr-known_sites Homo_sapiens_assembly38.dbsnp138.vcf \
  --bqsr-known_sites Homo_sapiens_assembly38.known_indels.vcf.gz \
  --bqsr-known_sites Mills_and_1000G_gold_standard.indels.hg38.vcf.gz \
  -o $output_name Homo_sapiens_assembly38.fasta $fq1 $fq2

# GATK
$ java -jar picard.jar FastqToSam
  FASTQ=$fq1 \
  FASTQ2=$fq2 \
  O=$fastqtosam_output_name \
  SM=example \
  PL=ILLUMINA

$ java -jar picard.jar MarkIlluminaAdapters
  I=$fastqtosam_output_name \
  O=$mark_illumina_adapter_output_name \
  M=$mark_illumina_adapter_metrics_name

$ java -jar picard.jar SamToFastq \
  I=$mark_illumina_adapter_output_name \
  FASTQ=/dev/stdout \
  CLIPPING_ATTRIBUTE=XT \
  CLIPPING_ACTION=2 \
  INTERLEAVE=true \
  NON_PF=true
  | bwa mem -K 100000000 -t $threads_num -p -Y Homo_sapiens_assembly38.fasta /dev/stdin
  | java -jar picard.jar MergeBamAlignment \
  VALIDATION_STRINGENCY=SILENT \
  EXPECTED_ORIENTATIONS=FR \
  ATTRIBUTES_TO_RETAIN=X0 \
  ATTRIBUTES_TO_REMOVE=NM \
  ATTRIBUTES_TO_REMOVE=MD \
  ALIGNED_BAM=/dev/stdin \
  UNMAPPED_BAM=$fastqtosam_output_name \
  O=$merge_bam_alignment_output_name \
  R=Homo_sapiens_assembly38.fasta \
  SORT_ORDER=unsorted \
  IS_BISULFITE_SEQUENCE=false \
  ALIGNED_READS_ONLY=false \
  CLIP_ADAPTERS=false \
  CLIP_OVERLAPPING_READS=true \
  ADD_MATE_CIGAR=true \
  MAX_INSERTIONS_OR_DELETIONS=-1 \
  PRIMARY_ALIGNMENT_STRATEGY=MostDistant \
  PROGRAM_RECORD_ID="bwamem" \
  PROGRAM_GROUP_VERSION="0.7.15" \
  PROGRAM_GROUP_COMMAND_LINE="bwa mem -K 100000000 -p -t 16 -Y" \
  PROGRAM_GROUP_NAME="bwamem" \
  UNMAPPED_READ_STRATEGY=COPY_TO_TAG \
  ALIGNER_PROPER_PAIR_FLAGS=true \
  UNMAP_CONTAMINANT_READS=true \
  ADD_PG_TAG_TO_READS=false

$ java -jar picard.jar MarkDuplicates \
  -I $merge_bam_alignment_output_name \
  -O $mark_dup_output_name \
  -M $mark_dup_metrics_name \
  --VALIDATION_STRINGENCY SILENT \
  --READ_NAME_REGEX null \
  --OPTICAL_DUPLICATE_PIXEL_DISTANCE 2500 \
  --ASSUME_SORT_ORDER queryname \
  --CLEAR_DT false \
  --ADD_PG_TAG_TO_READS false

$ java -jar picard.jar SortSam \
  INPUT=$mark_dup_output_name \
  OUTPUT=$sort_output_name \
  SORT_ORDER="coordinate" \
  CREATE_INDEX=true \
  CREATE_MD5_FILE=true

$ java \
  -Dsamjdk.use_async_io_read_samtools=false \
  -Dsamjdk.use_async_io_write_samtools=true \
  -Dsamjdk.use_async_io_write_tribble=false \
  -Dsamjdk.compression_level=2 \
  -jar gatk.jar BaseRecalibrator \
  -I $sort_output_name \
  -O $base_recalibrator_output_name \
  -R Homo_sapiens_assembly38.fasta \
  --known-sites Homo_sapiens_assembly38.dbsnp138.vcf
  --known-sites Homo_sapiens_assembly38.known_indels.vcf.gz
  --known-sites Mills_and_1000G_gold_standard.indels.hg38.vcf.gz
  --use-original-qualities

$ java \
  -Dsamjdk.use_async_io_read_samtools=false \
  -Dsamjdk.use_async_io_write_samtools=true \
  -Dsamjdk.use_async_io_write_tribble=false \
  -Dsamjdk.compression_level=2 \
  -jar gatk.jar ApplyBQSR \
  -I $base_recalibrator_output_name \
  -O $output_name \
  -R Homo_sapiens_assembly38.fasta \
  --bqsr-recal-file $bqsr_recal_output_name \
  --static-quantized-quals 10 --static-quantized-quals 20 --static-quantized-quals 30 \
  --add-output-sam-program-record \
  --create-output-bam-md5 \
  --use-original-qualities
'''

= Support

If you have any questions about this software, please contact to
support@genome4me.com.
