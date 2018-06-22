# Adapter trimming directions for NEBNext Single Cell/Low Input RNA Library Prep Kit for Illumina

For adapter trimming, we recommend Flexbar v3.3.0 or later, an open-source tool. To download Flexbar and for program usage, please visit [the Flexbar GitHub page](https://github.com/seqan/flexbar). Once Flexbar is installed, adapters can be trimmed with the following 2-step command.

```
flexbar --reads input_reads_1.fastq \
        --reads2 input_reads_2.fastq \
        --stdout-reads \
        --adapters tso_g_wo_hp.fasta \
        --adapter-trim-end LEFT \
        --adapter-revcomp ON \
        --adapter-revcomp-end RIGHT \
        --htrim-left GT \
        --htrim-right CA \
        --htrim-min-length 3 \
        --htrim-max-length 5 \
        --htrim-max-first \
        --htrim-adapter \
        --min-read-length 2 \
        --threads 1 | \
    flexbar \
        --reads - \
        --interleaved \
        --target output_reads \
        --adapters ilmn_20_2_seqs.fasta \
        --adapter-trim-end RIGHT \
        --min-read-length 2 \
        --threads 1
```

Make sure there are no extra characters after `\` at the end of the line, when the above command is used in the shell.

The input files for this command are the paired-end sequencing reads from the sequencing instrument: `input_reads_1.fastq` and `input_reads_2.fastq`. The adapter files `tso_g_wo_hp.fasta` and `ilmn_20_2_seqs.fasta`, as well as example fastq input files with a small number of reads can be downloaded [here](https://github.com/nebiolabs/nebnext-single-cell-rna-seq).

Flexbar is executed in two steps: (1) trimming template-switching oligo-related sequences (`tso_g_wo_hp.fasta`), and (2) trimming Illumina adapters (`ilmn_20_2_seqs.fasta`). The trimmed output reads are: `output_reads_1.fastq` and `output_reads_2.fastq`. To shorten runtime and reduce disk usage, the two flexbar commands are connected by a pipe, avoiding writing intermediate files. Log files are written into `flexbarOut.log` and `output_reads.log` for steps (1) and (2), correspondingly.

Homopolymers adjacent to the template-switching oligo are trimmed as well, as specified by `--htrim*` options. G and T homopolymers are trimmed (`--htrim-left GT --htrim-right CA`). Homopolymer length to trim is 3-5 for G, and 3 or higher for T (`--htrim-min-length 3 --htrim-max-length 5 --htrim-max-first`).

Keeping a short minimum read length after trimming (`--min-read-length 2`) keeps informative long reads, whose mates may be short after trimming. The default parameter `--threads 1` can be adjusted to a higher number depending on your machine.

In our tests, trimming with flexbar takes 1-2 minutes for 1 million 2x75 read pairs input fastq files on an m4.2xlarge AWS instance (8 core, 32 GB RAM, Intel Xeon E5-2676 v3 processor, 2.4 GHz), using `--threads 4`.
