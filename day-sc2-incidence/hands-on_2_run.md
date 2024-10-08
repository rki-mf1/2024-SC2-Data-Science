# Hands-on session Day 4: Incidence estimation

### 5 Initialization

As input the pipeline requires names of the input files, binning- and smoothing parameters.
These variables are stored in file **config.yml**.

*Note:* paths to files in **config.yml** should be either:
- absolute,
- or relative to the main pipeline Snakefile,
- or relative to the working directory.

For more information about the YAML markup format refer to documentation: https://yaml.org

#### 5.1 Input CSV/FASTA file
The pipeline requires a samples file along with a sequecing-, or better, collection date (format **%YYYY-%mm-%dd**). The file can be either given as a FASTA file or a table containing the dna profile (SNVs).

The pipeline automatically chooses the workflow according to the file extension. So please name your sample file either with *.fasta* or *.csv*.

Copy the file path and paste it into the variable **samples** of **config.yml**:

```bash
samples: "path/to/sequences/datafile"
```

In the field **name**, you can provide a name for the given samples, for example the country, location or any specifier for the given sequences, which is used for the plots.

```bash
name: country
```

*1. FASTA file*

If the input file is a FASTA file,the date must be part of the header behind a vertical bar, similar to sequence-names in GISAID: **'>some_name|%YYYY-%mm-%dd'**.

Along with the FASTA file a reference sequence needs to be provided (also in FASTA format). 
The header of the reference sequence can contain an arbitrary name, but importantly without white spaces:
**'>some_reference_name'**. If there are whitespaces present in the reference name string, a substring before the first whitespace will be used.

Add the file path of reference sequence into the variable **reference** of **config.yml**.

```bash
reference: "path/to/consensus/sequence.fasta"
```

*2. CSV file*

The input file can also be a CSV file containing one column with the date and one column with the mutations which are separated by a blank. Each mutation has a format *WildtypePositionMutant*. The column names must be given as *date* and *dna_profile*

```bash
date,dna_profile
2023-01-06,C241T T595C T670G C1931A C2790T
2023-01-06,C44T T670G T2954C
```

In this case, no reference file is needed.

#### 5.2 Reported cases data file

To compare estimated population dynamics with reported active cases, provide the following parameters in the corresponding config field like this:

```bash
reported_cases: ["path/to/reported_cases.csv", ",", date, new_cases, '%Y-%m-%d']
```

Alternatively, all arrays in the configuration file can be given as a list, like this:

```yaml
  reported_cases:
    - path/to/reported_cases.csv
    - ","
    - date
    - new_cases
    - '%Y-%m-%d'    
```

where the first element of the list is the file name with format extension, the second element is the delimiter type in this file, date column name, active cases column name, and a format the date is stored in.

If no reported cases data is provided, leave the fields empty like this:

```yaml
reported_cases: []
```

#### 5.3 Binning parameters

You also have to set the parameters for some of the binning methods in **config.yml**.
You can set the number of sequences per bin, and the number of days.
Parameters can be given as an array or a list (see section above). Additionally, minimal bin size and maximal days span should be
provided.

```yaml
  seq_per_bin: [20, 30]
  days_per_bin: [7, 10, 30]
  min_bin_size: 15
  max_days_span: 21
  min_days_span: 2
```

If parameter **seq_per_bin** is an empty list, a default mode with predefined fractions of sequences (2%, 5%, 7%) is used.

#### 5.4 Threshold for mutations

Low-abundance point mutations are filtered out to avoid using sequencing errors as real mutations.
The cutoff for the amount of mutations at a certain sequence position in the whole dataset is set with:

```yaml
  freq_cutoff: 2
```

#### 5.5 Trajectory smoothing parameters
Before the minimum true incidence is calculated the phi estimates and reported cases can be smoothed to prevent normalisation by an outlier. Smoothing bandwith is set with:

```yaml
  smoothing_bandwidth_phi: 7
  smoothing_bandwidth_mi: 7 
```

#### 5.6 Selecting time period 
Also the time frame to be considered can be optionally set with parameters

```yaml
from_date: "2022-01-01"
to_date: "2022-12-31"
```

If all dates are considered, leave the fields empty: 

```yaml
from_date: ""
to_date: ""
```

### 6 Run pipeline with demo dataset and configuration file
As a demo, we provide a subset of German SARS-CoV-2 sequences along with reported cases.

The directory **demo** comprises a data set with

- a csv file containing the dna profiles of SARS-CoV-2 sequences over time (demo_samples.csv)
- the reported cases (demo_reported_cases.csv)
- the config file to call the pipeline with (demo_config.yaml)

To run the pipeline go into the repository where the GInPipe file is located and run

```bash
snakemake --snakefile GInPipe --configfile demo/demo_config.yaml -j -d demo
```

### 7 Run pipeline with Germany data

Download a different subset of Germany data ([https://osf.io/hxk5m](https://osf.io/hxk5m)), configure and run the pipeline as described above.