---
layout: page
title: Notebook demo
exclude: true
---

*Note: this notebook is provided as a reference for students to consult while creating their own notebooks.
The material is irrelevant: the demo is intended to *

The genome of the paper wasp *Polistes canadensis* was [published in PNAS](http://dx.doi.org/10.1073/pnas.1515937112) in October 2015.
The genome assembly and annotation have been submitted to GenBank, but are not yet publicly available from that source.
However, the data has also been posted to an ["unofficial" project page at the CRG](http://wasp.crg.eu/) in the mean time.
Here I download the genome annotation and do a quick sanity check to make sure everything is in order.

## Download data

There are several files available, but I'm just checking the gene annotation and associated translated sequences.
The files were downloaded with the following commands.

```
wget http://wasp.crg.eu/PCAN.v01.gff3
wget http://wasp.crg.eu/PCAN.v01.pep.fa.gz
```

## Protein sequence IDs

First I want to see the format of the protein sequence IDs.
The following command will show the first 10 deflines in the Fasta file.

```
zgrep '^>' PCAN.v01.pep.fa.gz | head -n 10
```

Here is the output.

```
>PCAN011a000001P1
>PCAN011a000002P1
>PCAN011a000003P1
>PCAN011a000004P1
>PCAN011a000005P1
>PCAN011a000006P1
>PCAN011a000007P1
>PCAN011a000008P1
>PCAN011a000009P1
>PCAN011a000010P1
```

## Mapping to the annotation

Now I want to search the GFF3 file for one of these sequence IDs to see how they are stored.

```
grep PCAN011a000001P1 PCAN.v01.gff3
```

Which gives this output (you might have to scroll over to see the whole line.

```
scaffold_0	EVM_PASA	CDS	3641	3861	.	+	2	ID=PCAN011a000001C1;Parent=PCAN011a000001T1;Target=PCAN011a000001P1 1 74;partial_gene=true
```

So it looks like the protein sequence IDs from the Fasta file are stored in the `Target` attribute of CDS features in the GFF3 file.
The following command will show the first 10 in the GFF3 file.

```
grep $'\tCDS\t' PCAN.v01.gff3 \
    | perl -ne 'm/Target=(\S+)/ and print "$1\n"' \
    | head -n 10
```

Here is the output.

```
PCAN011a000001P1
PCAN011a000002P1
PCAN011a000003P1
PCAN011a000003P1
PCAN011a000004P1
PCAN011a000004P1
PCAN011a000005P1
PCAN011a000005P1
PCAN011a000005P1
PCAN011a000006P1
```

Note that some IDs are duplicated, so we'll need to use `sort` and `uniq` later if we want each ID to show up only once.

## Store the IDs and compare

Now that we know how the sequence IDs are stored in the two files, let's check to make sure there are no IDs missing from either file.

```
# Grab IDs from annotation
grep $'\tCDS\t' PCAN.v01.gff3 \
    | perl -ne 'm/Target=(\S+)/ and print "$1\n"' \
    | sort \
    | uniq \
    > pcan-gff3-ids.txt

# Grab IDs from sequence data
zgrep '^>' PCAN.v01.pep.fa.gz \
    | perl -ne 'm/>(\S+)/ and print "$1\n"' \
    | sort \
    > pcan-fa-ids.txt
head pcan-*-ids.txt

# Quick check
head -n 10 pcan-*-ids.txt
```

Here's the output.

```
==> pcan-fa-ids.txt <==
PCAN011a000001P1
PCAN011a000002P1
PCAN011a000003P1
PCAN011a000004P1
PCAN011a000005P1
PCAN011a000006P1
PCAN011a000007P1
PCAN011a000008P1
PCAN011a000009P1
PCAN011a000010P1

==> pcan-gff3-ids.txt <==
PCAN011a000001P1
PCAN011a000002P1
PCAN011a000003P1
PCAN011a000004P1
PCAN011a000005P1
PCAN011a000006P1
PCAN011a000007P1
PCAN011a000008P1
PCAN011a000009P1
PCAN011a000010P1
```

Looks good so far.
Now let's check for missing IDs.

```
# Venn diagram

echo -n 'Present in both: '
comm -12 pcan-fa-ids.txt pcan-gff3-ids.txt | wc -l

echo -n 'Missing from Fasta: '
comm -13 pcan-fa-ids.txt pcan-gff3-ids.txt | wc -l

echo -n 'Missing from GFF3: '
comm -23 pcan-fa-ids.txt pcan-gff3-ids.txt | wc -l
```

Here is the output.

```
Present in both:    16973
Missing from Fasta:        0
Missing from GFF3:        0
```

Great, it looks like there are no missing IDs!