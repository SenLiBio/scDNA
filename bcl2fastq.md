# convert bcl files to fastq files

*bcl2fastq* is a software developed to convert bcl files to fastq files.

Here we use *bcl2fastq* v2.20.

Official website: https://emea.support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html

A brief tutorial: https://www.10xgenomics.com/cn/support/software/cell-ranger/latest/analysis/inputs/cr-direct-demultiplexing-bcl2fastq

### 1. Information of index adapters

A practical guide to single-cell RNA-sequencing: https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-017-0467-4

In this study, we used Novaseq 6000 platform for sequencing, here's a brief introduction to the platform including the description of files in the output folders: https://support.illumina.com/content/dam/illumina-support/documents/documentation/system_documentation/novaseq/1000000019358_17_novaseq-6000-system-guide.pdf

In the sequencing output dir, you can see a file named `RunInfo.xml`, which contains lists the run name, number of cycles in each read, whether the read is an Index Read, and the number of swaths and tiles on the flow cell. 

Information for the reads:

```               <Reads>
<Reads>
	<Read Number="1" NumCycles="101" IsIndexedRead="N" IsReverseComplement="N"/>
	<Read Number="2" NumCycles="10" IsIndexedRead="Y" IsReverseComplement="N"/>
	<Read Number="3" NumCycles="10" IsIndexedRead="Y" IsReverseComplement="Y"/>
```

Then we can use `--use-bases-mask=Y101,I10,I10`, and for that the `IsIndexRead="Y" ` and `IsReverseComplement="Y"` for read number 3, we need to  use ==the sequences of index1 and reversed complemented sequences of index2== as the input index sequences for *bcl2fastq*.

---

### 2. SampleSheet.csv

The main aim of using samplt sheet is to demultiplex reads the lane into different samples by the index sequences. So we can reserve only the index information as following:

>[!Note]
>Remember that the header is `[Data]`

```[BCLConvert_Data]
[Data]
Sample_ID,Index,Index2
cell1,GACCAAGTTA,GATTCACGAC
cell2,CTCCGCTTAT,GATTCACGAC
cell3,ATTAAGGTCG,GATTCACGAC
```

---

### 3. run the software

```
bcl2fastq --use-bases-mask=Y101,I10,I10 \
--create-fastq-for-index-reads \
--minimum-trimmed-read-length=8 \
--mask-short-adapter-reads=8 \
--ignore-missing-positions \
--ignore-missing-controls \
--ignore-missing-filter \
--ignore-missing-bcls \
-r 3 -w 3 -p 10\
-R ${FLOWCELL_DIR} \
--output-dir=${OUTPUT_DIR} \
--sample-sheet=${SAMPLE_SHEET_PATH}
```



>[!Note]
>When failed with error reporting `Too many open files`, check this document:https://www.howtogeek.com/805629/too-many-open-files-linux/
