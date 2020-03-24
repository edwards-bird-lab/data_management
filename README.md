## Useful Data Management Commands

Below are a list of commands that are useful to lab members as they manage their data on the Harvard cluster. These have been divided into sections based on tasks with some very basic examples. In all cases, these are well-supported, commonly-used programs, so further Google searches should suffice if any additional questions arise. With all of these programs, there are lots of additional options that can be modified (see documentation of each tool), but the basic examples below will cover most applications.

### General

For file formats not covered below, genetic compression options should suffice. This includes custom file formats used by certain software/pipelines or large output or log files produced by some programs.

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

`bzip2` (`.bz2` extension) is another common compression type that will be encountered. It is also fairly well supported (not quite as much as `gzip`) and tends to produce more compressed files than `gzip`.

```
# gzip a file
bzip2 <file>
# ungzip a file
bunzip2 <file.gz>
# adjust compression level between 1 and 9 (default = 5)
bzip2 -7 <file>
# gzip all files in directory
bzip2 *
# gzip all files ending in .txt
bzip2 *.txt
```

`xzip` (`.xz` extension) is the last common compression type to note. It is probably less common than the other two and is therefore less likely to be supported. However, it compresses files the best, on average.

```
# gzip a file
xz <file>
# ungzip a file
xz -d <file.gz>
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
tar -cjvf name-of-archive.tar.gz /path/to/directory-or-file(s)
# extract contents of a gzipped archive
tar -xzvf name-of-archive.tar.gz
# extract contents of a bzip2 archive
tar -xjvf name-of-archive.tar.gz
```

### Sequence Files (FASTA/FASTQ)

### Mapping Files (SAM/BAM)

### Coordinate Files (VCF/GFF/BED)
