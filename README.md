# scRNA-cellbender-pipline
This document provides essential information on generating filtered count matrix files for Seurat or Scanpy from raw sequencing data. The pipeline begins with NCBI raw data, followed by fasterq-dump, renaming, cellranger count, and concludes with CellBender.

Due to the similarity of all processes, I will provide a detailed walkthrough using the E11_1 dataset as an example.

> note: In future work, I intend to organize the pipeline into a systematically structured workflow utilizing Snakemake for improved reproducibility and efficiency.

## fasterq-dump

First of all, for all 10X scRNA-seq raw data downloaded from [NCBI](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE151060), use following command to generate `.fastq` files:

``` zsh
for i in SRR118315**; do fasterq-dump ./$i -e 18 -m 55000MB -b 3000MB -v -p; done # adjust -e (threads), -m (memory), and -b (block size) according to your system specifications and resource availability.
```

## E11

### data info

Library1: SRR11831500 and SRR11831501

Library2: SRR11831502 and SRR11831503

| Run                                                          |  # of Spots | # of Bases |   Size |  Published |
| ------------------------------------------------------------ | ----------: | ---------: | -----: | ---------: |
| [SRR11831500](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831500) | 153,652,605 |      46.1G |   16Gb | 2021-08-07 |
| [SRR11831501](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831501) | 143,551,648 |      43.1G |   15Gb | 2021-08-07 |
| [SRR11831502](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831502) | 163,421,056 |        49G | 17.5Gb | 2021-08-07 |
| [SRR11831503](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831503) | 161,322,966 |      48.4G | 16.9Gb | 2021-08-07 |

### rename

rename files to meet [the requirement](https://www.10xgenomics.com/support/software/cell-ranger-arc/latest/analysis/inputs/specifying-input-fastq-count) of `cellranger count`:

``` zsh
x=1

 for i in *.fastq; do
    # Extract sample_id, read_id, and base_name
    sample_id=$(echo "$i" | grep -Po '(?<=SRR118315)..')
    read_id=$(echo "$i" | grep -Po '(?<=_).')
    base_name=$(echo "$i" | grep -o 'SRR118315..')

    # Create lane_id with leading zeros
    lane_id=$(printf "%03d" "$x")

    # Construct final_name
    final_name="E11_S1_L${lane_id}_R${read_id}_001.fastq"

    # Check if 'renamed' directory exists
    if [ ! -d ./renamed ]; then
        mkdir renamed
    else
        echo "Directory './renamed' already exists."
    fi

    # Create symbolic link with absolute path
    ln -s "$(readlink -f $i)" ./renamed/"$final_name"

    # Increment x for the next iteration
    ((x++))

    echo "$final_name done"
done

```

### cellranger count

``` zsh
cellranger count --id=E11_1 --fastqs=. --sample=E11 --transcriptome=/home/analysis/Mus_GRCm38.101/Mus_musculus
```

### list of files

``` zsh hl:27-30
(base)  mint-desktop@mint-desktop-H  /run/user/1000/gvfs/smb-share:server=truenas.local,share=public/HL/projects/sc/scRNA-old/02_cellRanger/data/BMK_DATA_20240301133142_1(E13_E15_data)/Data tree -L 2

E11_1
├── renamed
│   ├── E11_1
│   │   ├── _cmdline
│   │   ├── E11_1.mri.tgz
│   │   ├── _filelist
│   │   ├── _finalstate
│   │   ├── _invocation
│   │   ├── _jobmode
│   │   ├── _log
│   │   ├── _mrosource
│   │   ├── outs
│   │   ├── _perf
│   │   ├── SC_RNA_COUNTER_CS
│   │   ├── _sitecheck
│   │   ├── _tags
│   │   ├── _timestamp
│   │   ├── _uuid
│   │   ├── _vdrkill
│   │   └── _versions
│   ├── E11_S1_L000_R1_001.fastq -> ../SRR11831500_1.fastq
│   ├── E11_S1_L000_R2_001.fastq -> ../SRR11831500_2.fastq
│   ├── E11_S1_L001_R1_001.fastq -> ../SRR11831501_1.fastq
│   └── E11_S1_L001_R2_001.fastq -> ../SRR11831501_2.fastq
├── SRR11831500_1.fastq
├── SRR11831500_2.fastq
├── SRR11831501_1.fastq
└── SRR11831501_2.fastq
```

### cellbender

take **unfiltered** cellranger output as cellbender input:
Input file: /mnt/D/F_240226/cellbender/E11_1/raw_feature_bc_matrix.h5
Output file: /mnt/D/F_240226/cellbender/E11_1/14k/E11_1_ambient_removed.h5

``` zsh
# E11_1 and E11_2 use identical parameters, i.e., --expected-cells 7000 and --total-droplets-included 14000.
cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./14k/E11_1_ambient_removed.h5 --expected-cells 7000 --total-droplets-included 14000
```

## E13

### data info

> unpublished

| Client ID | Sample ID              |
| --------- | ---------------------- |
| RA9-13    | Unknown_BU571-003X0001 |

``` zsh hl:12,13
(base)  mint-desktop@mint-desktop-H  /run/user/1000/gvfs/smb-share:server=truenas.local,share=public/HL/projects/sc/scRNA-old/02_cellRanger/data/BMK_DATA_20240301133142_1(E13_E15_data)/Data tree -L 2
.
├── data_md5.txt
├── E13_1.fq.gz
├── E13_2.fq.gz
├── E15_1.fq.gz
├── E15_2.fq.gz
├── rawdata
│   ├── data_md5.txt
│   ├── sampleName_clientId.txt
│   ├── uncompressFileSize.metadata
│   ├── Unknown_BU571-003X0001_1.fq.gz
│   ├── Unknown_BU571-003X0001_2.fq.gz
│   ├── Unknown_BU571-003X0002_1.fq.gz
│   └── Unknown_BU571-003X0002_2.fq.gz
├── sampleName_clientId.txt
└── uncompressFileSize.metadata

1 directory, 14 files
```

### cellbender

``` zsh
cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./20k/E13_1_ambient_removed.h5 --expected-cells 10000 --total-droplets-included 20000
```


## E14

### data info

Library1: SRR11831504 - SRR11831513

Library2: SRR11831514 and SRR11831523

|      | [Run](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382016&o=acc_s%3Aa#) | [Bases](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382016&o=acc_s%3Aa#) | [Bytes](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382016&o=acc_s%3Aa#) | [create_date](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382016&o=acc_s%3Aa#) |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | [SRR11831504](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831504) | 10.13 G                                                      | 3.72 Gb                                                      | 2020-05-2213:47:00Z                                          |
| 2    | [SRR11831505](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831505) | 1.36 G                                                       | 484.42 Mb                                                    | 2020-05-2213:26:00Z                                          |
| 3    | [SRR11831506](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831506) | 8.61 G                                                       | 3.12 Gb                                                      | 2020-05-2213:44:00Z                                          |
| 4    | [SRR11831507](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831507) | 7.98 G                                                       | 2.93 Gb                                                      | 2020-05-2213:43:00Z                                          |
| 5    | [SRR11831508](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831508) | 7.92 G                                                       | 2.91 Gb                                                      | 2020-05-2213:43:00Z                                          |
| 6    | [SRR11831509](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831509) | 7.93 G                                                       | 2.90 Gb                                                      | 2020-05-2213:45:00Z                                          |
| 7    | [SRR11831510](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831510) | 1.23 G                                                       | 435.20 Mb                                                    | 2020-05-2213:25:00Z                                          |
| 8    | [SRR11831511](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831511) | 10.57 G                                                      | 3.89 Gb                                                      | 2020-05-2213:48:00Z                                          |
| 9    | [SRR11831512](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831512) | 10.53 G                                                      | 3.90 Gb                                                      | 2020-05-2214:23:00Z                                          |
| 10   | [SRR11831513](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831513) | 10.65 G                                                      | 3.93 Gb                                                      | 2020-05-2213:55:00Z                                          |
| 11   | [SRR11831514](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831514) | 11.43 G                                                      | 4.16 Gb                                                      | 2020-05-2214:00:00Z                                          |
| 12   | [SRR11831515](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831515) | 10.03 G                                                      | 3.70 Gb                                                      | 2020-05-2213:59:00Z                                          |
| 13   | [SRR11831516](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831516) | 1.61 G                                                       | 573.81 Mb                                                    | 2020-05-2213:27:00Z                                          |
| 14   | [SRR11831517](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831517) | 10.67 G                                                      | 3.87 Gb                                                      | 2020-05-2213:52:00Z                                          |
| 15   | [SRR11831518](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831518) | 10.07 G                                                      | 3.69 Gb                                                      | 2020-05-2214:23:00Z                                          |
| 16   | [SRR11831519](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831519) | 1.50 G                                                       | 531.22 Mb                                                    | 2020-05-2213:26:00Z                                          |
| 17   | [SRR11831520](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831520) | 8.93 G                                                       | 3.28 Gb                                                      | 2020-05-2214:11:00Z                                          |
| 18   | [SRR11831521](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831521) | 8.96 G                                                       | 3.30 Gb                                                      | 2020-05-2213:55:00Z                                          |
| 19   | [SRR11831522](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831522) | 8.90 G                                                       | 3.28 Gb                                                      | 2020-05-2213:46:00Z                                          |
| 20   | [SRR11831523](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831523) | 9.64 G                                                       | 3.50 Gb                                                      | 2020-05-2213:54:00Z                                          |

### cellbender

``` zsh
# E14_1 and E14_2 use identical parameters, i.e., --expected-cells 20000 and --total-droplets-included 40000.
cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./40k/E14_1_ambient_removed.h5 --expected-cells 20000 --total-droplets-included 40000
```


## E15

### data info

> notice: NOT published yet.

Library1: Unknown_BU571-002X0001

Library2: Unknown_BU571-003X0002

| clientId  | SampleName<br>         |
| --------- | ---------------------- |
| RA10-15&1 | Unknown_BU571-002X0001 |

| Client ID | Sample ID              |
| --------- | ---------------------- |
| RA9-15    | Unknown_BU571-003X0002 |


``` zsh hl:6-7
# Library1
(base)  mint-desktop@mint-desktop-H  /run/user/1000/gvfs/smb-share:server=truenas.local,share=public/HL/projects/sc/scRNA-old/02_cellRanger/data/BMK_DATA_20240111164212_1/Data   master ±  tree rawdata      
rawdata
├── data_md5.txt
├── sampleName_clientId.txt
├── uncompressFileSize.metadata
├── Unknown_BU571-002X0001_1.fq.gz
└── Unknown_BU571-002X0001_2.fq.gz

0 directories, 5 files

```

``` zsh hl:14,15
# Library2
(base)  mint-desktop@mint-desktop-H  /run/user/1000/gvfs/smb-share:server=truenas.local,share=public/HL/projects/sc/scRNA-old/02_cellRanger/data/BMK_DATA_20240301133142_1(E13_E15_data)/Data tree -L 2
.
├── data_md5.txt
├── E13_1.fq.gz
├── E13_2.fq.gz
├── E15_1.fq.gz
├── E15_2.fq.gz
├── rawdata
│   ├── data_md5.txt
│   ├── sampleName_clientId.txt
│   ├── uncompressFileSize.metadata
│   ├── Unknown_BU571-003X0001_1.fq.gz
│   ├── Unknown_BU571-003X0001_2.fq.gz
│   ├── Unknown_BU571-003X0002_1.fq.gz
│   └── Unknown_BU571-003X0002_2.fq.gz
├── sampleName_clientId.txt
└── uncompressFileSize.metadata

1 directory, 14 files
```

### cellbender

``` zsh
cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./30k/E15_ambient_removed.h5 --expected-cells 15000 --total-droplets-included 30000

cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./19k/E15_2_ambient_removed.h5 --expected-cells 11000 --total-droplets-included 19000
```

## P0

Library1: SRR11831524 - SRR11831527

Library2: SRR11831528 and SRR11831531

### data info

| Run                                                          |    of Spots | of Bases |   Size |  Published |
| ------------------------------------------------------------ | ----------: | -------: | -----: | ---------: |
| [SRR11831524](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831524) | 125,371,257 |    37.6G | 13.9Gb | 2021-08-07 |
| [SRR11831525](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831525) |  44,457,669 |    13.3G |  4.8Gb | 2021-08-07 |
| [SRR11831526](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831526) |  96,622,540 |      29G | 10.7Gb | 2021-08-07 |
| [SRR11831527](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831527) |  30,730,887 |     9.2G |  3.3Gb | 2021-08-07 |
| [SRR11831528](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831528) | 127,927,109 |    38.4G | 14.3Gb | 2021-08-07 |
| [SRR11831529](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831529) |  39,902,396 |      12G |  4.4Gb | 2021-08-07 |
| [SRR11831530](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831530) |  99,630,143 |    29.9G |   11Gb | 2021-08-07 |
| [SRR11831531](https://trace.ncbi.nlm.nih.gov/Traces?run=SRR11831531) |  31,957,525 |     9.6G |  3.5Gb | 2021-08-07 |

### cellbender

``` zsh
# P0_1 and P0_2 use identical parameters, i.e., --expected-cells 14000 and --total-droplets-included 28000.
cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./28k/P0_1_ambient_removed.h5 --expected-cells 14000 --total-droplets-included 28000
```

## P7

Library: SRR11831532 - SRR11831547

### data info

|      | [Run](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382018&o=acc_s%3Aa#) | [Bases](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382018&o=acc_s%3Aa#) | [Bytes](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382018&o=acc_s%3Aa#) | [create_date](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRX8382018&o=acc_s%3Aa#) |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | [SRR11831532](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831532) | 14.27 G                                                      | 4.95 Gb                                                      | 2020-05-2213:57:00Z                                          |
| 2    | [SRR11831533](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831533) | 12.74 G                                                      | 4.44 Gb                                                      | 2020-05-2214:01:00Z                                          |
| 3    | [SRR11831534](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831534) | 13.03 G                                                      | 4.55 Gb                                                      | 2020-05-2213:54:00Z                                          |
| 4    | [SRR11831535](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831535) | 13.07 G                                                      | 4.55 Gb                                                      | 2020-05-2214:03:00Z                                          |
| 5    | [SRR11831536](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831536) | 12.73 G                                                      | 4.47 Gb                                                      | 2020-05-2214:23:00Z                                          |
| 6    | [SRR11831537](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831537) | 12.36 G                                                      | 4.34 Gb                                                      | 2020-05-2214:06:00Z                                          |
| 7    | [SRR11831538](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831538) | 12.62 G                                                      | 4.44 Gb                                                      | 2020-05-2214:13:00Z                                          |
| 8    | [SRR11831539](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831539) | 12.65 G                                                      | 4.44 Gb                                                      | 2020-05-2214:25:00Z                                          |
| 9    | [SRR11831540](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831540) | 13.79 G                                                      | 4.79 Gb                                                      | 2020-05-2214:03:00Z                                          |
| 10   | [SRR11831541](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831541) | 14.21 G                                                      | 4.94 Gb                                                      | 2020-05-2214:13:00Z                                          |
| 11   | [SRR11831542](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831542) | 14.24 G                                                      | 4.94 Gb                                                      | 2020-05-2214:12:00Z                                          |
| 12   | [SRR11831543](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831543) | 15.14 G                                                      | 5.29 Gb                                                      | 2020-05-2214:05:00Z                                          |
| 13   | [SRR11831544](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831544) | 14.69 G                                                      | 5.14 Gb                                                      | 2020-05-2214:10:00Z                                          |
| 14   | [SRR11831545](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831545) | 14.98 G                                                      | 5.24 Gb                                                      | 2020-05-2214:42:00Z                                          |
| 15   | [SRR11831546](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831546) | 15.02 G                                                      | 5.25 Gb                                                      | 2020-05-2214:09:00Z                                          |
| 16   | [SRR11831547](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR11831547) | 13.17 G                                                      | 4.59 Gb                                                      | 2020-05-2214:51:00Z                                          |

### cellbender

``` zsh
# P7
cellbender remove-background --cuda --input ./raw_feature_bc_matrix.h5 --output ./24k/P7_1_ambient_removed.h5 --expected-cells 12000 --total-droplets-included 24000
```


> [!INFO]
>
> - reference: Mus_GRCm38.101
> - cellranger version: 7.2.0
> - cellbender version: 0.3.0
