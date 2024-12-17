## Archiving

- Archiving is way for combining multiple files and directories into one file

- Primarily used for backup and simplification of distribution

- Main tools are `tar`, `gzip` nad `bzip2`

### Tools

1. `tar`: Originally for tape archiving. It can manage and organize file into archive. It is a command used to create, maintain, modify, and extract files from an archive file.

2. `gzip` & `bzip`: Used for file compression, reducing the file size and making data transmission easier.

#### Examples

1. To create a tar archive:

```bash
  tar cvf archive_name.tar directory_to_archive/
```

_Options_:

The `c` option tells tar to create a new archive. This option is used when you want to bundle files and directories into a single archive file.

The `v` option stands for "verbose." It tells tar to list the files being added to the archive as they are processed. This provides feedback in the terminal, showing which files are being included in the archive.

The `f` option specifies the name of the archive file. It tells tar that the next argument (`archive_name.tar`) is the name of the archive file to create.

2. To extract tar archive:

```bash
  tar xvf archive_name.tar
```

_Options_:

The `x` option tells tar to extract the contents of the archive. This option is used when you want to unpack the files and directories from the archive into the current directory.

The `v` option stands for "verbose." It tells tar to list the files being extracted as they are processed. This provides feedback in the terminal, showing which files are being unpacked.

The `f` option specifies the name of the archive file. It tells tar that the next argument (`archive_name.tar`) is the name of the archive file to extract.

3. To create a gzip compressed tar archive:

```bash
  tar cvzf archive_name.tar.gz directory_to_archive/
```

_Options_:

The `c` option tells tar to create a new archive. This is used when you want to bundle files and directories into a single archive file.

The `v` option stands for "verbose." It tells tar to list the files being added to the archive as they are processed. This provides feedback in the terminal, showing which files are being included in the archive.

The `z` option tells tar to compress the archive using gzip, a popular compression algorithm. This results in a smaller file size, which is useful for saving space or transferring the archive over a network.

The `f` option specifies the name of the archive file. It tells tar that the next argument (archive_name.tar.gz) is the name of the archive file to create.

4. To extract a gzip compressed tar archive:

```bash
tar xvfz node_exporter-1.8.2.linux-amd64.tar.gz
```

_Options_:

The `x` option tells tar to extract the contents of the archive. This option is used when you want to unpack the files and directories from the archive into the current directory.

The `v` option stands for "verbose." It tells tar to list the files being added to the archive as they are processed. This provides feedback in the terminal, showing which files are being included in the archive.

The `z` option tells tar to decompress the archive using gzip. This is needed because the file is compressed (.tar.gz).

The `f` option specifies the name of the archive file. It tells tar that the next argument (archive_name.tar.gz) is the name of the archive file to create.

5. To create a bzip2 compressed tar archive:

```bash
tar cvjf archive_name.tar.bz2 directory_to_archive/
```

_Options_:

The `c` option tells tar to create a new archive. This is used when you want to bundle files and directories into a single archive file.

The `v` option stands for "verbose." It tells tar to list the files being added to the archive as they are processed. This provides feedback in the terminal, showing which files are being included in the archive.

The `j` option tells tar to compress the archive using bzip2, a compression algorithm that typically produces smaller files than gzip, but may take longer to compress.

The `f` option specifies the name of the archive file. It tells tar that the next argument (archive_name.tar.bz2) is the name of the archive file to create.

6. To extract a bzip2 compressed tar archive:

```bash
tar xvjf archive_name.tar.bz2
```

_Options_:

The `x` option tells tar to extract the contents of the archive. This option is used when you want to unpack the files and directories from the archive into the current directory.

The `v` option stands for "verbose." It tells tar to list the files being added to the archive as they are processed. This provides feedback in the terminal, showing which files are being included in the archive.

The `j` option tells tar to decompress the archive using bzip2

The `f` option specifies the name of the archive file. It tells tar that the next argument (archive_name.tar.bz2) is the name of the archive file to create.

**Note**: In Linux, **archiving** and **compression** are separate processes, hence tar to archive and gzip/bzip2 to compress. Although they’re commonly used together, they can very much be used separately as per the requirements.
