# FADU
Feature Aggregate Depth Utility

## Github Pages site
https://igs.github.io/FADU/

# Description
Generate counts of reads that map to non-overlapping portions of genes

# Authors
* Shaun Adkins (sadkins@som.umaryland.edu)
* Matthew Chung (mattchung@umaryland.edu)

# Requirements
Julia - v0.7 or later (v1.0 or later preferred)

## Modules
NOTE: Module installation instructions can be found at https://docs.julialang.org/en/v1/stdlib/Pkg/index.html#Getting-Started-1
* ArgParse
* BioAlignments - v1.0.0
* GenomicFeatures - v1.0.0

# Input
* A single BAM file of reads (sorted by position)
* A GFF3-formatted annotation file

# Output
The output is a tab-delimited file whose name is the BAM input file's basename, ending in '.counts.txt'.  For example if the name of the BAM file was 'sample123.bam', the output file name is 'sample123.counts.txt'

The output file has the following fields:
* Name of feature
* Number of bases that do not overlap with other features (uniq\_len)
* Number of BAM records that aligned to this feature's non-overlapping bases
  * Aligned reads not part of a fragment get 0.5 count, and fragments get 1 count
* Fractionalized feature counts 
  * For each aligned fragment, the depth is (intersect of fragment coords to feature coords that do not overlap with other features) / length of the fragment
  * Aligned reads that are not part of a fragment get their depth contribution cut in half
* TPM count in scientific notation

# Installation instructions
First `cd` to the directory of choice to install FADU
```
git clone https://github.com/IGS/FADU.git
cd FADU
```

# Example command

For this example we will create a directory called "fadu\_output" to write output into.  This example uses a BAM alignment file of subsampled Wolbachia reads.  This example should take about 30 seconds to run.
```
mkdir ./fadu_output
/usr/local/bin/julia ./fadu.jl --remove_multimapped -g "./test_data/GCF_000008385.1_ASM838v1_genomic.gff" -b "./test_data/SRR5192555.100000x.sortedbyposition.bam" -o "./fadu_output" -s "reverse" -f "CDS" -a "ID"
```
The resulting output file should be called "SRR5192555.100000x.sortedbyposition.counts.txt" and located in the "fadu\_output" directory.

# Usage
```
usage: fadu.jl -b /path/to/file.bam -g /path/to/annotation.gff3
               -o /path/to/output/dir/ [-s STRANDED] [-f FEATURE_TYPE]
               [-a ATTRIBUTE_TYPE] [-p] [-m MAX_FRAGMENT_SIZE]
               [-q MIN_MAPPING_QUALITY] [-M] [-C CHUNK_SIZE]
               [--version] [-h]

Generate counts of reads that map to non-overlapping portions of genes

optional arguments:
  -b, --bam_file /path/to/file.bam
                        Path to BAM file (SAM is not supported).
  -g, --gff3_file /path/to/annotation.gff3
                        Path to GFF3-formatted annotation file (GTF is
                        not supported).
  -o, --output_dir /path/to/output/dir/
                        Directory to write the output.
  -s, --stranded STRANDED
                        Indicate if BAM reads are from a
                        strand-specific assay. Choose between 'yes',
                        'no', or 'reverse'. (default: "no")
  -f, --feature_type FEATURE_TYPE
                        Which GFF3/GTF feature type (column 3) to
                        obtain. readcount statistics for.
                        Case-sensitive. (default: "gene")
  -a, --attribute_type ATTRIBUTE_TYPE
                        Which GFF3/GTF feature type (column 9) to
                        obtain. readcount statistics for.
                        Case-sensitive. (default: "ID")
  -p, --keep_only_proper_pairs
                        If enabled, keep only properly paired reads
                        when performing calculations.
  -m, --max_fragment_size MAX_FRAGMENT_SIZE
                        If the fragment size of properly-paired reads
                        exceeds this value, process pair as single
                        reads instead of as a fragment. Setting this
                        value to 0 will make every fragment pair be
                        processed as two individual reads. Maximum
                        value is 65535 (to allow for use of UInt16
                        type, and fragments typically are not that
                        large). If --keep_only_proper_pairs is
                        enabled, then any fragment exceeding this
                        value will be discarded. (type: Int64,
                        default: 1000)
  -q, --min_mapping_quality MIN_MAPPING_QUALITY
                        Set the minimum mapping quality score for a
                        read to be considered. Max value accepted is
                        255. (type: Int64, default: 10)
  -M, --remove_multimapped
                        If enabled, remove any reads or fragments that
                        are mapped to multiple regions of the genome,
                        indiated by the 'NH' attribute being greater
                        than 1.
  -C, --chunk_size CHUNK_SIZE
                        Number of validated reads to store into memory
                        before processing overlaps with features.
                        (type: Int64, default: 10000000)
  --version             show version information and exit
  -h, --help            show this help message and exit

```

# Special Notes
## Notes about kept reads for calculations
When processing the input BAM file, any reads that are not properly paired or are not primary mappings will be thrown out.  This is determined by the SAM-format bit-flag.

# Frequently Asked Questions
## Is FADU designed to work with both prokaryotic and eukaryotic samples?
FADU currently does not support eukaryotic samples

## When running FADU.jl, I get a "too many arguments" error.  How do I fix this?
A safe way to pass in values to the options is to enclose any string arguments (like file paths or strandedness) in quotations so that it ensures Julia's argument parser reads the parameter as an option value instead of a command-type argument.

## Can I use FADU to calculate counts for features using read depth instead of fragment depth?
Currently at this time, only fragment depth is supported.  Performing calculations using read depth may be implemented in the future though.

# Issues
If you have a question or suggestion about FADU, feel free to e-mail either of the authors above, or create an issue [here](https://github.com/IGS/FADU/issues)

