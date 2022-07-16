## Useful Data Management Commands

Below are a list of commands that are useful to lab members as they manage their data on the Harvard cluster. These have been divided into sections based on tasks with some very basic examples. In all cases, these are well-supported, commonly-used programs, so further Google searches should suffice if any additional questions arise. With all of these programs, there are lots of additional options that can be modified (see documentation of each tool), but the basic examples below will cover most applications.

### General

For file formats not covered below, generic compression options should suffice. This includes custom file formats used by certain software/pipelines or large output or log files produced by some programs. Phylogenetics file formats with many loci (e.g., Nexus, Newick, etc.) will also probably need to be compressed. Compression options vary and have been adopted at different rates, so support for certain options in certain circumstances may be minimal. If you provided a proper plain-text file, I expect all general compression options to work to compress the file, but the degree to which downstream bioinformatics software will utilize that file format varies (this is what I largely mean when referring to 'support'). For example, gzipped files are far more common, and more likely to be supported in bioinformatics software, than files compressed with bzip2 or xz.

`gzip` (`.gz` extension) is the most commonly used option and is well supported.

```
# gzip a file
gzip <file>
# ungzip a file
gunzip <file.gz>
# adjust compression level between 1 and 9 (default = 5)
gzip -7 <file>
# gzip all files in directory
gzip *
# gzip all files ending in .txt
gzip *.txt
```

`bzip2` (`.bz2` extension) is another common compression type that will be encountered. It is also fairly well supported (not as much as `gzip`) and tends to produce more compressed files than `gzip`.

```
# gzip a file
bzip2 <file>
# ungzip a file
bunzip2 <file.bz2>
# adjust compression level between 1 and 9 (default = 5)
bzip2 -7 <file>
# gzip all files in directory
bzip2 *
# gzip all files ending in .txt
bzip2 *.txt
```

`xzip` (`.xz` extension) is the last common compression type to note. It is less common than the other two and is therefore less likely to be supported. However, it compresses files the best, on average.

```
# gzip a file
xz <file>
# ungzip a file
xz -d <file.xz>
# adjust compression level between 1 and 9 (default = 5)
xz -7 <file>
# gzip all files in directory
xz *
# gzip all files ending in .txt
xz *.txt
```

One can also create traditional `zip` files of one or more files, but these tend to be encountered much less in a bioinformatic setting. The above compression types should suffice for any large files you produce.

Occassionally, you may want to compress a bunch of files at once into one compressed file (say, a large set of output files from a program). `zip` will do this, but usually in the Unix/bioinformatics world, users will instead create a tape archive (`.tar`) file, which can be compressed. Tape archive refers to the way we used to archive data and does not have much meaning anymore, but look for the `.tar` extension.

```
# create a gzipped tar archive
tar -czvf name-of-archive.tar.gz /path/to/directory-or-file(s)
# create a bzip2 tar archive
tar -cjvf name-of-archive.tar.bz2 /path/to/directory-or-file(s)
# extract contents of a gzipped archive
tar -xzvf name-of-archive.tar.gz
# extract contents of a bzip2 archive
tar -xjvf name-of-archive.tar.bz2
```

Note that it is seldom necessary to use a gzipped tar (often called a 'tarball') in bioinformatics. Just properly zipping large files and placing them into a standard directory should suffice. Additionally, placing them into a tarball does not really save any additional space. And 'tarballing' several uncompressed files just makes it more difficult to extract a single file if you ever needed to do that. So don't count on needing to do this very often.

### Sequence Files (FASTA/FASTQ)

**Sequence files like FASTA and, especially, FASTQ should always be zipped.** You will likely receive raw sequencing from the sequencing provider as gzipped FASTQ (`.fastq.gz`) or large genomic files from databases as gzipped FASTA (`.fasta.gz`). In general, it is fine to just leave them in this format. With sequencing reads, any mapping software worth using will work directly with gzipped FASTQ. With reference genomes in FASTA, most mapping software will also index a gzipped version of the reference without issue. Therefore, unless you are working with a piece of non-traditional software, keeping both FASTA and FASTQ gzipped is ideal.

If you ever need to recompress sequence files, the commands under General will apply. Please note that while gzipped versions of these files are commonly supported directly, bzip2 and xz files are much less likely to be supported, so the additional compression may not be worth the loss of compatability.

### Mapping Files (SAM/BAM)

The original mapping file format was SAM, which you can open and read. BAM quickly followed as a binary (i.e., compressed) version of SAM. **All mapping data should be in BAM format (at least).** Essentially all downstream software will work directly with BAM files. Moreover, most mappers will either write directly to BAM or will write to STDOUT, which can be piped through `samtools` to produce a BAM file on the fly. So there is really never a need to have SAM files, which are much less space efficient.

If you ever need to write a BAM file from an existing SAM or want to convert to BAM on the fly, the following command is what you need.

```
# write existing SAM to BAM
samtools view -S -b sample.sam > sample.bam
# write BAM output on the fly when mapping with BWA or something similar
<mapping command that writes to STDOUT> | samtools view -S -b - > sample.bam
```

As you can see, `samtools` if your best friend here. However, there is alternative software that you can essentially plug in, which may have advantages.

More recently, due to large amounts of data not being produced, the CRAM file format was introduced as an even more compressed version of SAM/BAM. In general, if you are working with large resequencing datasets, you should move towards producing these files intead of BAM, as they are quite a bit smaller. Most downstream software in standard variant calling should work with CRAM, but if in doubt, keeping as BAM in the beginning and later converting to CRAM for more permanent storage is probably fine. The command to produce CRAM files is similar to what is above, but you also need a sorted BAM file and the reference genome.

```
# sort the existing SAM
samtools sort -O bam -o file.bam file.sam
# create the CRAM file
samtools view -T reference.fasta -C -o file.cram file.bam
# write directly to CRAM when mapping
bwa mem reference.fasta read1.fastq.gz read2.fastq.gz | \
samtools sort -O bam - | \
samtools view -T reference.fasta -C -o file.cram -
```

Sometimes, you may want to view the contents of a mapping file as SAM output to look at something manually. The nice thing about the compressed BAM/CRAM is that you can index them, which gives you random access to the contents of the files based on the coordinate location of the locus of interest (based on the reference). To do this, you must have a BAM/CRAM file, and then you can index it.

```
samtools index <file.bam>
```

A new, small index file will appear in the working directory. Then you can actually specify SAM output from a region of interest.

```
# all mappings to chromosome 1
samtools view <file.bam> chr1
# mappings to chromosome 2 beginning at position 1,000,000 until the end of the reference sequence
samtools view <file.bam> chr2:1000000
# mappings to chromosome 3 that span positions 1001 to 2000 of that reference sequence
samtools view <file.bam> chr3:1000-2000
```

One last note is that when mapping read data, unmapped reads are usually stored alongside the reads that mapped (see settings for BWA and other tools). Therefore, you can actually extract the raw input reads from the BAM/CRAM files, which usually represents quality-filtered/trimmed reads. Therefore, you can purge these quality-trimmed reads and manually extract them again from the BAM/CRAM mapping file later. In practice, those working with model organisms and doing a lot of genome sequencing will actually receive their raw data is BAM files mapped to their model reference genome, as BAM is an efficient format for storing raw sequencing data and eliminates the need to also keep FASTQ files. With non-model organisms, this will not usually happen, but this possibility is mentioned here for the sake of completeness. Given how much data is multiplying, strain on computational resources is growing, and this is a valuable way to reduce that strain.

### Coordinate Files (VCF/GFF/BED)

In a standard variant calling pipeline, BAM mapping files are used to call variants, which are usually writen to a variant calling format with the `.vcf` extension. As is the case above, VCF files can also be further compressed to save space, which may be worthwhile with large variant datasets. Look at the options for your variant calling tool, as it may be possible to write to a more compressed data format from the start. If you ever want to compress VCF, you have three compression options: (1) zipped VCF (`.vcf.gz`), (2) binary VCF (`.bcf`), or (3) compressed binary VCF (`.bcf.gz`). `bcftools` is one great tool for making these conversions, but there are likely others.

```
# convert VCF to zipped VCF
bcftools view -Oz file.vcf > file.vcf.gz
# convert VCF to BCF
bcftools view -Ou file.vcf > file.bcf
# convert VCF to compressed BCF
bcftools view -Ob file.vcf > file.bcf.gz
```

Usually, just doing the standard compression is sufficient for variant files, as they usually contain far less data than mapping or sequencing files because they only encode variant sites. But for large variant files or in cases where a user wishes to also export invariant sites, the heavier compression options may pay dividends. An alternative way to produce a `.vcf.gz` file is to use the program `bgzip`, which is my preferred option.

```
# convert VCF to zipped VCF
bgzip file.vcf > file.vcf.gz
# uncompress the zipped VCF
bgzip -d file.vcf.gz > file.vcf
```

The nice thing about `bgzip` is that it is designed to work on all coordinate data - basically any dataset where the reference scaffold/chromosome and positions are encoded as certain columns of a tabular data file. Therefore, `bgzip` will also compress other standard genomics file formats, namely BED and GFF3.

```
# compress BED
bgzip file.bed > file.bed.gz
# compress GFF3
bgzip file.gff3 > file.gff3.gz
```

So this is a great tool for genomics data in general, as BED files and GFF3 files can be relatively large and are commonly encountered.

And finally, the really nice thing about `bgzip` and these compressed file formats in general is that they can also be indexed using a companion tool called `tabix`. This allows random access to common genomics file formats, so you can pull out mappings, features, etc. from certain regions of VCF, BED, or GFF3 files. In order to do this, the tabular file format must be sorted by scaffold/chromosome and then position. There are tools that can do this, or you can sort by the appropriate columns using Unix `sort` (just Google for examples). Then you can compress using `bgzip` (see above) and index using `tabix`.

```
# index BED with tabix
tabix file.bed.gz
# tabix is usually smart, but you can tell it what the input is
tabix -p bed file.bed.gz
```

A small index file will be created alongside the data file. Finally, you can randomly access data by coordinates like we did above for BAM/CRAM files using `tabix`.

```
# all features to chromosome 1
tabix <file.bed.gz> chr1
# features on chromosome 2 beginning at position 1,000,000 until the end of the reference sequence
tabix <file.bed.gz> chr2:1000000
# features on chromosome 3 that span positions 1001 to 2000 of that reference sequence
tabix <file.bed.gz> chr3:1000-2000
```

The random access to genomic file formats is very useful and in combination with the compressed nature of the data, these tools are invaluable and worth learning for any bioinformatician. Many common tools (e.g., `samtools`, `bcftools`, `bedtools`, etc.) are actually written to take advantage of these compressed and indexed file formats, since they are so invaluable when analyzing large datasets.


### Comparing file checksums 

Calculating and comparing file checksums is important for ascertaining that large sequencing files you've copied from the data provider (e.g., a collaborator or the Bauer Core) are complete and intact after data transfer. Often when you copy sequencing directories over from the Bauer Core, they will contain a file like the following, containing calculated checksums, often named something like `md5sum.txt`. Checksums are unique markers or little digital breadcrumbs from a file which allow you to compare two different versions to ensure that the copied version has no errors.  

The following is a checksum file that was distributed with the Illumina sequencing data I recieved from the Bauer Sequencing Core. The 'checksum' is the alphanumeric string preceeding the filepath to each `fastq` file.

```
b61fdf435897082a1e564d1bb2977a93  fastq/PREP0053/PREP0053_SMacR10811A_A01v1_Mel-205-1384-16Oct-H20_S33_L001_R1_001.fastq.gz
f1ca4970690cc546e3c30a0ebb6d930e  fastq/PREP0053/PREP0053_SMacR10811A_A01v1_Mel-205-1384-16Oct-H20_S33_L001_R2_001.fastq.gz
7e07612a21142914d374978d57042892  fastq/PREP0053/PREP0053_SMacR10811A_B01v1_Cyan-185-1283-16Oct-H20_S34_L001_R1_001.fastq.gz
533a46cb3d0b20a1483fecc86fbb0293  fastq/PREP0053/PREP0053_SMacR10811A_B01v1_Cyan-185-1283-16Oct-H20_S34_L001_R2_001.fastq.gz
dfa2cbcf7382caaf7559dfe88c45c9d9  fastq/PREP0053/PREP0053_SMacR10811A_C01v1_Chry-194-1331-16Oct-H20_S35_L001_R1_001.fastq.gz
e74762cc2635942f2876fab56041b264  fastq/PREP0053/PREP0053_SMacR10811A_C01v1_Chry-194-1331-16Oct-H20_S35_L001_R2_001.fastq.gz
050b3bcb1991b5e44a863c3525dd2b27  fastq/PREP0053/PREP0053_SMacR10811A_D01v1_Striat-184-1281-15Oct-H20_S36_L001_R1_001.fastq.gz
f00897cb54aaf3506074e50821d14d65  fastq/PREP0053/PREP0053_SMacR10811A_D01v1_Striat-184-1281-15Oct-H20_S36_L001_R2_001.fastq.gz
5d3932783deb2d6c3b25619629a1d5dd  fastq/PREP0053/PREP0053_SMacR10811A_E01v1_Mel-178-1251-3Oct-H20_S37_L001_R1_001.fastq.gz
2e2893ef6b635c78d72cfebcf461518d  fastq/PREP0053/PREP0053_SMacR10811A_E01v1_Mel-178-1251-3Oct-H20_S37_L001_R2_001.fastq.gz
```

To compare my copied files against this checksum file, I executed the following command: 

```
md5sum -c md5sum.txt > checksums_output.txt
```

I was happily able to see that the integrity of my copied files was not compromised during the copying process (I used `rsync`, fyi) -- they were all deemed `OK` by the checksums comparion. 

```
md5sum -c <md5sum.txt>
# fastq/PREP0053/PREP0053_SMacR10811A_A01v1_Mel-205-1384-16Oct-H20_S33_L001_R1_001.fastq.gz: OK
# fastq/PREP0053/PREP0053_SMacR10811A_A01v1_Mel-205-1384-16Oct-H20_S33_L001_R2_001.fastq.gz: OK
# fastq/PREP0053/PREP0053_SMacR10811A_B01v1_Cyan-185-1283-16Oct-H20_S34_L001_R1_001.fastq.gz: OK
# fastq/PREP0053/PREP0053_SMacR10811A_B01v1_Cyan-185-1283-16Oct-H20_S34_L001_R2_001.fastq.gz: OK
```

If the checksums list file is not already generated for the original files, you can also do this yourself, and save to the directory of your choice. You should get into the habit of doing this whenever you are handling a new dataset, especially if the data provider has not given you any existing checksums. If you share data with others, you should provide these checksums so that they can verify the integrity of the data before they begin analysis.

```
md5sum <file(s)> > md5sum.txt
```
