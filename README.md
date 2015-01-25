This is a small library and a bunch of clients to perform various operations on FASTQ files (such as demultiplexing raw Illumina files, merging partial or complete overlaps, and/or performing quality filtering). It works with, paired-end FASTQ files and tested with Illumina runs processed with CASAVA version 1.8.0 or higher.

Send your questions to `meren at mbl dot edu`.

# Contents

- [Installing](#installing)
    - [Requirements](#requirements)
- [Demultiplexing](#demultiplexing)
- [Config File Format](#config-file-format)
    - [[general] section](#general-section)
    - [[files] section](#files-section)
    - [[prefixes] section](#prefixes-section)
- [Merging Partially Overlapping Illumina Pairs](#merging-partially-overlapping-illumina-pairs)
    - [Example STATS output](#example-stats-output)
    - [Recovering high-quality reads from merged reads file](#recovering-high-quality-reads-from-merged-reads-file)
- [Merging Completely Overlapping Illumina Pairs](#merging-completely-overlapping-illumina-pairs)
- [Quality Filtering](#quality-filtering)
    - ["Complete Overlap" analysis for V6](#complete-overlap-analysis-for-v6)
        - [Example STATS output](#example-stats-output)
    - [Minoche et al.](#minoche-et-al)
        - [Example STATS output](#example-stats-output)
        - [Example PNG files](#example-png-files)
    - [Bokulich et al.](#bokulich-et-al)
        - [Example STATS output:](#example-stats-output)
        - [Example PNG files](#example-png-files)
- [Questions?](#questions)

# Installing

The easiest way to install illumina-utils is to do it through pip. To install the latest version, you can simply run this command on your terminal:

    sudo pip install illumina-utils

Alternatively, you can download the source code from [here](https://github.com/meren/illumina-utils/releases/tag/v1.0), unpack it, and install running the following command from within the illumina-utils directory:

    sudo python setup.py install

If you would like to play with the development version, you can create a copy of the codebase by simply installing `git` and running this command in your terminal window:

     git clone git://github.com/meren/illumina-utils.git

## Requirements

In order to use this software package fully, you need following items available on your system:

- [matplotlib](http://matplotlib.org/) (required for visualizations)
- [python-Levenshtein](https://pypi.python.org/pypi/python-Levenshtein/) (to merge partially overlapping reads)
- [R](http://r-project.org) (required for visualizations today and will be used for statistical analyses)
    - [ggplot2](http://ggplot2.org/) (the R module that needs to be installed for R requirement)

`matplotlib` and `python-Levenshtein` will be installed automatically if you install illumina-utils using `pip` or `setup.py`.


# Demultiplexing

If you have raw FASTQ files, you can demultiplex them into samples using `iu-demultiplex` script. In order to demultiplex a run, you will need an extra FASTQ file for indexes, and a TAB-delimited file for barcode-sample associations. The directory [examples/demultiplexing](https://github.com/meren/illumina-utils/tree/master/examples/demultiplexing) contains sample files for demultiplexing. You can start the process by running the following command:

    iu-demultiplex -s barcode_to_sample.txt --r1 r1.fastq --r2 r2.fastq --index index.fastq -o output/

If you have demultiplexed your raw files using this library, you can save yourself from generating config files (properties of which explained in the next section) by hand. The script `iu-gen-configs` takes the report file generated by `iu-demultiplex`, and automatically generates config files for each sample. For instance, the following command would have been an appropriate to run after the previous `iu-demultiplex` example:

    iu-gen-configs output/00_DEMULTIPLEXING_REPORT

# Config File Format

Most clients under the [scripts directory](https://github.com/meren/illumina-utils/tree/master/scripts) require information to be passed via a standard config file format, if they require a config file as an input. Following is a config file template this codebase uses (there is also a [sample](https://github.com/meren/illumina-utils/blob/master/examples/merging/general-config-SAMPLE.ini) file in the codebase):

    [general]
    project_name = project name
    researcher_email = reasearcher@institute.edu
    input_directory = /full/path/test_input
    output_directory = /full/path/test_output
    
    
    [files]
    pair_1 = pair_1_aaa, pair_1_aab, pair_1_aac, pair_1_aad, pair_1_aae, pair_1_aaf 
    pair_2 = pair_2_aaa, pair_2_aab, pair_2_aac, pair_2_aad, pair_2_aae, pair_2_aaf
    
    [prefixes]
    pair_1_prefix = ^....TACGCCCAGCAGC[C,T]GCGGTAA.
    pair_2_prefix = ^CCGTC[A,T]ATT[C,T].TTT[G,A]A.T


## [general] section

This is a mandatory section that contains `project_name`, `researcher_email`, `input_directory` and `output_directory` directives.

Two critical declerations in `[general]` section are `input_directory` and `output_directory`:

* `input_directory`: Full path to the directory where FASTQ files reside.
* `output_directory`: Full path to the directory where the output of the operation you will perform on this config to be stored. Since when it is Illumina we are dealing with huge files, the codebase is pretty conservative to protect users from making simple mistakes which may result in huge losses. So, if you don't create the `output_directory`, you will get an error (it will not be automatically generated). If there is already a file in the `output_directory` with the same name with one of the outputs, you will get an error (it will not be overwritten). `project_name` will be used as a prefix for the naming convention of output files, so it would be wise to choose something descriptive and UNIX-compatible.

## [files] section

`files` section is where you list your _file names_ to be found under `input_directory`. Each file name has to be comma separated. The index of each file name in the comma seperated list, *must match* with its pair in the second list (see the example config file above).

## [prefixes] section

`prefixes` section is optional. If you have barcodes and primers in your reads, and you want them to be trimmed, you can use [regular expression](http://en.wikipedia.org/wiki/Regular_expressions)s to specify them. If prefixes are defined, results would contain only pairs that matched them.


# Merging Partially Overlapping Illumina Pairs

Pairs generated by paritally overlapping library preperation can be merged using [merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs). Once you create your config file, you simply call it with the config file as a parameter.

By default, merging program uses Levenshtein distance to find the best merging strategy for two reads in a pair, starting from the minimum expected overlap (15 nt is the default, and can be changed through the appropriate command line parameter).

The merging strategy requires [Levenshtein module](http://code.google.com/p/pylevenshtein/) to be installed.

[merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs) will create FASTA files for reads that were merged successfuly, or failed to merge. In the FASTA file for merged reads, the length of the overlapped region and the number of mismatches found in the overlapped part will be reported in the header line for each entry. The place of mismatch will be shown with capital letters in the sequences.

An example header line from the FASTA file for merged reads is shown below:

    >M01028:4:000000000-A1Y0P:1:1101:18829:1947 1:N:0:1|o:83|m/o:0.048193|MR:n=0;r1=2;r2=2|Q30:p,77;p,76|mismatches:4

Each field is separated from each other by a "|" character. Field one is the original defline from the FASTQ file of read 1. Following items explain details of these fields and command line options that affect them:

* `o`: Length of the overlap.
* `m/o`: The P value. P value is the ratio of the number of mismatches and the length of the overlap. Merged sequences can be discarded based on this ratio. The default is 0.3. This value should be changed through the command line parameter `-P` depending on the expected overlap size (if the expected length of overlap is 100 nts and if you choose to eliminate any pairs with more than 5 mismatches at the overlapping region, you can set the `-P` parameter to 0.05).
* `MR`: "Mismatches Recovered". When there is a mismatch in the overlapped region, the base to be used in the final merged sequence is picked from whichever read possesses the higher Q-score (and shown as a capital letter in the merged sequence). If a mismatch is recovered from read 1, it increases the number next to r1 in this field, and so forth. However, if there is a disagreement between two reads, *and* neither of the reads have a Q-score higher than a minimum acceptable value, the corresponding base denoted with an `N` in the merged read, and the number next to `none` is increased by one. By default, the minimum Q-score value is 10. This value can be changed via the command line parameter `--min-qual-score`. Note that if `--ignore-Ns` flag is not declared, all merged sequences that had at least one disagreement which can't be recovered from neither reads due to `--min-qual-score` value will be discarded.
* `Q30`: By default, quality filtering is being done based only on the mismatches found in the overlapped region, and the beginning and the end of merged reads are not being checked. However a final control can be enforced using the command line flag `--enforce-Q30-check`. This flag turns on the Q30 check, as it was explained by [Minoche _et al_](http://genomebiology.com/2011/12/11/R112). Briefly, Q30-check eliminates pairs if the 66% of bases in the *first half* of each read do not have Q-scores over Q30. Note that Q30 is applied only to the parts of reads that did not overlap. If either of reads fail Q30 check, merged sequence is discarded. `p,77;p,76` in the example header reads as "read 1 passed Q30 check (threfore `p`, failed case denoted by an `f`), and 77 bases in the first half of it had a better Q-score than 30; read 2 passed Q30 check, and 76 bases in the first half of it had a better Q-score than 30". If Q30-check was not enforced `n/a` appears next to it.
* `mismatches`: Number of mismatches at the overlapped region for quick filtering of resulting reads.

Here is a snippet from the merged sequences file (reads are trimmed from both ends for readability):

    >M01028:4:000000000-A1Y0P:1:1101:15704:1943 1:N:0:1|o:87|m/o:0.022989|MR:n=0;r1=2;r2=0|Q30:p,77;p,72|mismatches:2
    [...]ggtagatggaatataacatgtagcggtgaaatGctTagatatgttatggaacaccgattgcgaaggcagtctactaagtcgatattgacgctgaggcacgaaagcgtgggtagcgaacag[...]
    >M01028:4:000000000-A1Y0P:1:1101:18231:1947 1:N:0:1|o:86|m/o:0.058140|MR:n=0;r1=5;r2=0|Q30:p,74;p,66|mismatches:5
    [...]ggaaagtggaatttctaGTGTagaggtgaaattcgtagatattagaaagaacatcaaaggcGaaggcaactttctggatcattactgacactgaggaacgaaagcatgggtagcgaagag[...]
    >M01028:4:000000000-A1Y0P:1:1101:18829:1947 1:N:0:1|o:83|m/o:0.048193|MR:n=0;r1=2;r2=2|Q30:p,77;p,76|mismatches:4
    [...]ggggggtagaatTccacgtgtagcagtgaaatgcgtagagatgtggaGgaatAtcaatggcgaaggcagccccctgggataacactgacgCtcatgcacgaaagcgtggggagcgaacag[...]


If the program runs successfully, these files will appear in the `output_directory`:

* `project_name_MERGED` (successfuly merged reads)
* `project_name_FAILED` (failed sequences due to `m/o`)
* `project_name_FAILED_WITH_Ns` (failed merged sequences for having ambiguous bases)
* `project_name_FAILED_Q30` (failed merged sequences for not passing Q30-check, if enforced)
* `project_name_MISMATCHES_BREAKDOWN` (number of mismatches breakdown)
* `project_name_STATS` (numbers regarding the run)

`project_name_MISMATCHES_BREAKDOWN` file can be visualized using the R script, [mismatches-distribution.R](https://github.com/meren/illumina-utils/blob/master/scripts/R/mismatches-distribution.R), included in the codebase (it will require ggplot2 to be available on the system). Here is an example:



![Example output](http://meren.org/tmp/breakdown.png)


When [merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs) is run with `--compute-qual-dicts` it will also generate visualization of quality scores for different number of mismatch levels. Please see command line options for more information.


## Example STATS output

The `project_name_STATS` file that is created in the output directory contains important information about the merging operation. It is a good practice to check the numbers and make sure there is no anomalies. Here is an example output:

    Number of pairs analyzed        	2500
    Prefix failed in read 1         	0
    Prefix failed in read 2         	0
    Prefix failed in both           	0
    Passed prefix total             	2500
    Failed prefix total             	0
    Merged total                    	1479
    Merge failed total              	1021
    Merge discarded due to P        	598
    Merge discarded due to Ns       	348
    Merge discarded due to Q30      	75
    Total number of mismatches      	13101
    Mismatches recovered from read 1	10360
    Mismatches recovered from read 2	1413
    Mismatches replaced with N      	1328
    
    Mismatches breakdown:
    
    0	372
    1	326
    2	225
    3	154
    4	120
    5	86
    6	70
    7	49
    8	40
    9	21
    10	11
    11	4
    12	1
    
    
    Command line            	merge-illumina-pairs miseq_partial_overlap_config.ini z --enforce --compute
    Work directory              	/path/to/the/working/directory
    "p" value                   	0.300000
    Min overlap size            	15
    Min Q-score for mismatches  	10
    Ns ignored?                 	False
    Q30 enforced?               	True
    Slow merge?                 	False




## Recovering high-quality reads from merged reads file

If [merge-illumina-pairs](https://github.com/meren/illumina-utils/blob/master/scripts/merge-illumina-pairs) finishes successfuly, it will generate `project_name_MERGED` for successfuly merged reads. A successful merge depends on the `o/r` value, Q30-check and lack of ambiguous bases in the merged sequence. However, succesfully merged reads based on user-defined or default parameters may not be as accurate as needed depending on the project. Further elimination of reads can be done by filtering out reads based on the number of mismatches they present at the overlapped region. For instance, user can decide to use only merged sequences with 0 mismatches from the resulting FASTA file.

Program [filter-merged-reads](https://github.com/meren/illumina-utils/blob/master/scripts/filter-merged-reads) can be used to retain high-quality reads from `project_name_MERGED` file. To retain reads with 0 mismatches at the overlapped region you can simply run this command on your `project_name_MERGED` to generate a file with filtered reads `project_name_FILTERED`:

     filter-merged-reads project_name_MERGED --max-mismatches 0 --output project_name_FILTERED

Resulting file would be the file to use in downstream analyses.

# Merging Completely Overlapping Illumina Pairs

Please use `merge-illumina-pairs` the same way explained in the [Merging Partially Overlapping Illumina Pairs](#merging-partially-overlapping-illumina-pairs) section, but include your command line these two flags:

    (...) --marker-gene-stringent --retain-only-overlap

# Quality Filtering

## "Complete Overlap" analysis for V6

[analyze-illumina-v6-overlaps](https://github.com/meren/illumina-utils/blob/master/scripts/analyze-illumina-v6-overlaps) can be used to generate very high quality short reads from sequences that were generated by a short insert size library preperation method. Library preperation method and the efficacy of the complete overlap analysis is described in [Eren _et al_](http://www.plosone.org/article/info:doi/10.1371/journal.pone.0066643). Once the manuscript is published, the reference will be available here. The output of the analyze-illumina-v6-overlaps script include these files:

* `project_name-STATS.txt` (an example output can be seen below)
* `project_name-PERFECT_reads.fa` (FASTA file for reads that passed the complete overlap analysis)
* `project_name-Q_DICT.cPickle.z` (gzipped cPickle object for Python that holds the machine reported quality scores for each group of failed and passed reads)
* `project_name-READ_IDs.cPickle.z` (gzipped cPickle object for Python that holds the read fate information)

If the program is run with `--visualize-quality-curves` option, following files will also be generated in the output directory:

* `project_name-PASSED.png` (visualization of mean machine reported quality scores per tile for pairs that passed the the complete overlap analysis)
* `project_name-FAILED_MISMATCH.png` (same for pairs that failed due to one or more mismatches at the region of read of interest)
* `project_name-FAILED_RP.png` (same for pairs that lacked a proper reverse primer)
* `project_name-FAILED_FP.png` (same for pairs that lacked a proper forward primer)

### Example STATS output

    $ cat 9022_B9-STATS.txt
    number of pairs                 : 828243
    total pairs passed              : 618589 (%74.69 of all pairs)
      perfect pairs with Ns         : 0 (%0.00 of perfect pairs)
      recovered ambiguous bases (p1): 0 (%0.00 of perfect pairs)
      recovered ambiguous bases (p2): 0 (%0.00 of perfect pairs)
    total pairs failed              : 209654 (%25.31 of all pairs)
      FP failed in both pairs       : 5086 (%2.43 of all failed pairs)
      FP failed only in pair 1      : 386 (%0.18 of all failed pairs)
      FP failed only in pair 2      : 116822 (%55.72 of all failed pairs)
      RP failed in both pairs       : 7076 (%3.38 of all failed pairs)
      RP failed only in pair 1      : 6767 (%3.23 of all failed pairs)
      RP failed only in pair 2      : 7647 (%3.65 of all failed pairs)
      FAILED_MISMATCH               : 65870 (%31.42 of all failed pairs)
      FAILED_RP                     : 21490 (%10.25 of all failed pairs)
      FAILED_FP                     : 122294 (%58.33 of all failed pairs)


## Minoche et al.

Quality filtering suggestions made by [Minoche _et al_](http://genomebiology.com/2011/12/11/R112) is implemented in [analyze-illumina-quality-minoche](https://github.com/meren/illumina-utils/blob/master/scripts/analyze-illumina-quality-minoche) script. The output of the scripts include these files:

* `project_name-STATS.txt` (file that contains all the numbers about quality filtering process, an example output can be seen below)
* `project_name-QUALITY_PASSED_R1.fa` (pair 1's that passed quality filtering)
* `project_name-QUALITY_PASSED_R2.fa` (matching pair 2's)
* `project_name-READ_IDs.cPickle.z` (gzipped cPickle object for Python that keeps the fate of read IDs, this file may be required by other scripts in the library for purposes such as visualization, or extracting a particular group of reads from the original FASTQ files)

If the program is run with `--visualize-quality-curves` option, these files will also be generated in the output directory:

* `project_name-PASSED.png` (visualization of mean quality scores per tile for pairs that passed the quality filtering)
* `project_name-FAILED_REASON_C33.png` (visualization of mean quality scores per tile for pairs that failed quality filtering due to C33 filtering (C33: less than 2/3 of bases were Q30 or higher in the first half of the read following the B-tail trimming))
* `project_name-FAILED_REASON_N.png` (same above, but for pairs that contained an ambiguous base after B-tail trimming)
* `project_name-FAILED_REASON_P.png` (same above, but for pairs that were too short after B-tail trimming)
* `project_name-Q_DICT.cPickle.z` (gzipped cPickle object for Python that holds mean quality scores for each group of reads)

### Example STATS output

    $ cat 9022_B9-STATS.txt
    number of pairs analyzed      : 122929
    total pairs passed            : 109041 (%88.70 of all pairs)
      total pair_1 trimmed        : 6476 (%5.94 of all passed pairs)
      total pair_2 trimmed        : 9059 (%8.31 of all passed pairs)
    total pairs failed            : 13888 (%11.30 of all pairs)
      pairs failed due to pair_1  : 815 (%5.87 of all failed pairs)
      pairs failed due to pair_2  : 12193 (%87.80 of all failed pairs)
      pairs failed due to both    : 880 (%6.34 of all failed pairs)
      FAILED_REASON_P             : 12223 (%88.01 of all failed pairs)
      FAILED_REASON_N             : 38 (%0.27 of all failed pairs)
      FAILED_REASON_C33           : 1627 (%11.72 of all failed pairs)

### Example PNG files

![Example output](http://meren.org/tmp/minoche.gif)

## Bokulich et al.

Quality filtering suggestions made by [Bokulich _et al_](http://www.nature.com/nmeth/journal/v10/n1/full/nmeth.2276.html) is implemented in [analyze-illumina-quality-bokulich](https://github.com/meren/illumina-utils/blob/master/scripts/analyze-illumina-quality-bokulich) script. The output of the scripts include these files:

* `project_name-STATS.txt`
* `project_name-QUALITY_PASSED_R1.fa`
* `project_name-QUALITY_PASSED_R2.fa`
* `project_name-READ_IDs.cPickle.z`

If the program is run with `--visualize-quality-curves` option, these files will also be generated in the output directory:

* `project_name-PASSED.png`
* `project_name-FAILED_REASON_P.png` (visualization of mean quality scores per tile for pairs that failed quality filtering for being too short after quality trimming)
* `project_name-FAILED_REASON_N.png` (same above, but having more ambiguous bases than `n` after quality trimming)
* `project_name-Q_DICT.cPickle.z`

### Example STATS output:

    number of pairs analyzed      : 122929
    total pairs passed            : 111598 (%90.78 of all pairs)
      total pair_1 trimmed        : 1994 (%1.79 of all passed pairs)
      total pair_2 trimmed        : 9227 (%8.27 of all passed pairs)
    total pairs failed            : 11331 (%9.22 of all pairs)
      pairs failed due to pair_1  : 738 (%6.51 of all failed pairs)
      pairs failed due to pair_2  : 10159 (%89.66 of all failed pairs)
      pairs failed due to both    : 434 (%3.83 of all failed pairs)
      FAILED_REASON_P             : 11299 (%99.72 of all failed pairs)
      FAILED_REASON_N             : 32 (%0.28 of all failed pairs)

### Example PNG files

![Example output](http://meren.org/tmp/bokulich.gif)


# Questions?

Please don't hesitate to get in touch with me via `meren at mbl dot edu`.
