# ncov-recombinant
<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-7-orange.svg?style=flat-square)](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/ktmeaton/ncov-recombinant/blob/master/LICENSE)
[![GitHub issues](https://img.shields.io/github/issues/ktmeaton/ncov-recombinant.svg)](https://github.com/ktmeaton/ncov-recombinant/issues)
[![Install CI](https://github.com/ktmeaton/ncov-recombinant/actions/workflows/install.yaml/badge.svg)](https://github.com/ktmeaton/ncov-recombinant/actions/workflows/install.yaml)
[![Test CI](https://github.com/ktmeaton/ncov-recombinant/actions/workflows/test.yaml/badge.svg)](https://github.com/ktmeaton/ncov-recombinant/actions/workflows/test.yaml)
[![Pipeline CI](https://github.com/ktmeaton/ncov-recombinant/actions/workflows/pipeline.yaml/badge.svg)](https://github.com/ktmeaton/ncov-recombinant/actions/workflows/pipeline.yaml)

SARS-CoV-2 recombinant sequence detection inspired by [nextstrain/ncov](https://github.com/nextstrain/ncov).

1. Align sequences and perform clade/lineage assignments with [Nextclade](https://github.com/nextstrain/nextclade).
1. Identify parental clades and plot recombination breakpoints with [sc2rf](https://github.com/lenaschimmel/sc2rf).
1. Create tables, plots, and powerpoint slides for reporting.

---

A recombinant lineage is defined as a group of sequences with a unique combination of:

- lineage assignment (ex. `XM`)
- parental clades (ex. `Omicron/21K,Omicron/21L`)
- parental lineages (ex. `BA.1.1,BA.2.12.1`)
- breakpoints (ex. `17411:21617`)

Novel recombinants (i.e. undesignated) can be identified by a lineage assignment that does not start with `X*` (ex. BA.1.1) _or_ with a lineage assignment that contains `-like` (ex. `XM-like`). A cluster of sequences may be flagged as `-like` if one of following criteria apply:

1. The lineage assignment by [Nextclade](https://github.com/nextstrain/nextclade) conflicts with the published breakpoints for a designated lineage (`resources/breakpoints.tsv`).

    - Ex. An `XE` assigned sample has breakpoint `11538:12879` which conflicts with the published `XE` breakpoint (`ex. 8394:12879`). This will be renamed `XE-like`.

1. The cluster has 10 or more sequences, which share at least 3 private mutations in common.

    - Ex. A large cluster of sequences (N=50) are assigned `XM`. However, these 50 samples share 5 private mutations `T2470C,C4586T,C9857T,C12085T,C26577G` which do not appear in true `XM` sequences. This will be renamed `XM-like`. Upon further review of the reported matching [pango-designation issues](https://github.com/cov-lineages/pango-designation/issues) (`460,757,781,472,798`), we find this cluster to be a match to `proposed798`.

## Table of Contents

1. [Install](https://github.com/ktmeaton/ncov-recombinant#install)
1. [Update](https://github.com/ktmeaton/ncov-recombinant#update)
1. [Tutorial](https://github.com/ktmeaton/ncov-recombinant#tutorial)
1. [Output](https://github.com/ktmeaton/ncov-recombinant#output)
1. [Configuration](https://github.com/ktmeaton/ncov-recombinant#configuration)
1. [High Performance Computing](https://github.com/ktmeaton/ncov-recombinant#high-performance-computing)
1. [FAQ](https://github.com/ktmeaton/ncov-recombinant#faq)
1. [Credits](https://github.com/ktmeaton/ncov-recombinant#credits)

## Install

1. Clone the repository, using the `--recursive` flag for submodules:

    ```bash
    git clone --recursive https://github.com/ktmeaton/ncov-recombinant.git
    cd ncov-recombinant
    ```

2. Install dependencies in a conda environment (`ncov-recombinant`):

    ```bash
    mamba env create -f workflow/envs/environment.yaml
    conda activate ncov-recombinant
    ```

## Update

> **Tip**: If you have modified any pipeline files, it is often easier to instead do a [fresh install](https://github.com/ktmeaton/ncov-recombinant#install).

1. Save (`stash`) any modifications you may have made to pipeline files.

    ```bash
    git stash
    ```

1. Download and apply the latest updates.

    ```bash
    git pull --recurse-submodules
    ```

1. Update your conda environment.

    ```bash
    mamba env update -f workflow/envs/environment.yaml
    ```

## Tutorial

> **Tip**: Remember to run `conda activate ncov-recombinant` first!

1. Preview the steps that are going to be run.

    ```bash
    snakemake --profile profiles/tutorial --dryrun
    ```

1. Run the workflow.

    ```bash
    snakemake --profile profiles/tutorial
    ```

1. Explore the [output](https://github.com/ktmeaton/ncov-recombinant#output).

    - Slides | `results/tutorial/report/report.pptx`
    - Tables<sup>*</sup> | `results/tutorial/report/report.xlsx`
    - Plots | `results/tutorial/plots`
    - Breakpoints<sup>†</sup>
        - Between clades (primary) | `results/tutorial/sc2rf/recombinants.ansi.primary.txt`
        - Within Omicron (secondary) | `results/tutorial/sc2rf/recombinants.ansi.secondary.txt`

<sup>*</sup> Individual tables are available as TSV linelists in `results/tutorial/linelists`.  
<sup>†</sup> Visualize breakpoints with `less -S` or [Visual Studio ANSI Colors](https://marketplace.visualstudio.com/items?itemName=iliazeus.vscode-ansi).  

## Output

### Tables

Linelists are collated into a spreadsheet for excel/google sheets:

1. `lineage`: The recombinant lineages observed.
1. `parents`: The parental combinations observed.
1. `linelist`: Results from <u>all</u> input sequences (minimal statistics).
1. `summary`: Results from <u>all</u> input sequences (all possible statistics, for troubleshooting).
1. `positives`: Results from sequences classified as a <u>recombinant</u> by at least 2 of 3 classifiers.
1. `false_positives`: Results from sequences flagged as recombinants by Nextclade, that were not verified by [sc2rf](https://github.com/lenaschimmel/sc2rf) or [UShER](https://github.com/yatisht/usher).
1. `negatives`: Results from sequences classifed as a <u>non-recombinant</u> by nextclade.
1. `issues`: Metadata of issues related to recombinant lineages posted in the [pango-designation](https://github.com/cov-lineages/pango-designation/issues) repository.

[![excel_output](images/excel_output.png)](
https://docs.google.com/spreadsheets/d/1kVqQScrJneeJ4t7RmSeJTeWVGHecir1CEgq6hl8qirI)

### Slides

Powerpoint/google slides with plots embedded for presenting.

[![powerpoint_output](images/powerpoint_output.png)](https://docs.google.com/presentation/d/1Jo4BYBa2K8kvRnKxo9ZpnqhS86ZLh3D1HTF3E8hfD0Q)

### Breakpoints

Visualization of breakpoints by parental clade and parental lineage.

|                                         Clade                                          |                                         Lineage                                          |
|:--------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------:|
| ![breakpoints_clade](https://github.com/ktmeaton/ncov-recombinant/raw/dev/images/breakpoints_clade.png) | ![breakpoints_lineage](https://github.com/ktmeaton/ncov-recombinant/raw/dev/images/breakpoints_lineage.png) |

Visualization of parental alleles from [sc2rf](https://github.com/lenaschimmel/sc2rf).

![sc2rf_output](images/sc2rf_output.png)

## Controls

- After completing the tutorial, a good next step is to run the `controls` build.
- This build analyzes publicly available sequences in [`data/controls`](https://github.com/ktmeaton/ncov-recombinant/tree/master/data/controls), which include recombinant ("positive") and non-recombinant ("negative") sequences.
- Instructions for how to include the `controls` in your custom build are in the [configuration](https://github.com/ktmeaton/ncov-recombinant#configuration) section.

1. Preview the steps that are going to be run.

    ```bash
    snakemake --profile profiles/controls --dryrun
    ```

1. Run the workflow.

    ```bash
    snakemake --profile profiles/controls
    ```

## Configuration

1. Create a new directory for your data.

    ```bash
    mkdir -p data/custom
    ```

1. Copy over your <u>unaligned</u> `sequences.fasta` and `metadata.tsv` to `data/custom`.

    > - **Note**: `metadata.tsv` MUST have at minimum the columns `strain`, `date`, `country`.  
    > - **Note**: The first column MUST be `strain`.

1. Create a profile for your custom build.

    ```bash
    scripts/create_profile.sh --data data/custom
    ```

    ```text
    2022-06-17 09:15:06     Searching for metadata (data/custom/metadata.tsv)
    2022-06-17 09:15:06     SUCCESS: metadata found
    2022-06-17 09:15:06     Checking for 3 required metadata columns (strain date country)
    2022-06-17 09:15:06     SUCCESS: 3 columns found.
    2022-06-17 09:15:06     Searching for sequences (data/custom/sequences.fasta)
    2022-06-17 09:15:06     SUCCESS: Sequences found
    2022-06-17 09:15:06     Checking that the metadata strains match the sequence names
    2022-06-17 09:15:06     SUCCESS: Strain column matches sequence names
    2022-06-17 09:15:06     Creating new profile directory (my_profiles/custom)
    2022-06-17 09:15:06     Creating build file (my_profiles/custom/builds.yaml)
    2022-06-17 09:15:06     Adding default input data (defaults/inputs.yaml)
    2022-06-17 09:15:06     Adding custom input data (data/custom)
    2022-06-17 09:15:06     Adding `custom` as a build
    2022-06-17 09:15:06     Creating system configuration (my_profiles/custom/config.yaml)
    2022-06-17 09:15:06     Adding default system resources
    2022-06-17 09:15:06     Done! The custom profile is ready to be run with:

                            snakemake --profile my_profiles/custom
    ```

    > - **Note**: you can add the param `--controls` to add a `controls` build that will run in parallel.
    > - **Note**: The `controls` build analyzes a dataset of positive and negative recombinant sequences.

1. Edit `my_profiles/custom/config.yaml`, so that the `jobs` and `default-resources` match your system.

    > **Note**: For HPC environments, see the [High Performance Computing](https://github.com/ktmeaton/ncov-recombinant#high-performance-computing) section.

    ```yaml
    #------------------------------------------------------------------------------#
    # System config
    #------------------------------------------------------------------------------#

    # Maximum number of jobs to run
    jobs : 2

    # Default resources for a SINGLE JOB
    default-resources:
    - cpus=2
    - mem_mb=8000
    - time_min=120
    ```

1. Do a dry run to confirm setup.

    ```bash
    snakemake --profile my_profiles/custom --dry-run
    ```

1. Run your custom profile.

    ```bash
    snakemake --profile my_profiles/custom
    ```

## High Performance Computing (HPC)

`ncov-recombinant` can alternatively be dispatched using the SLURM job submission system.

1. Create an HPC-compatible profile to store your build configuration.

    ```bash
    scripts/create_profile.sh --data data/custom --hpc
    ```

    ```text
    2022-06-17 09:16:55     Searching for metadata (data/custom/metadata.tsv)
    2022-06-17 09:16:55     SUCCESS: metadata found
    2022-06-17 09:16:55     Checking for 3 required metadata columns (strain date country)
    2022-06-17 09:16:55     SUCCESS: 3 columns found.
    2022-06-17 09:16:55     Searching for sequences (data/custom/sequences.fasta)
    2022-06-17 09:16:55     SUCCESS: Sequences found
    2022-06-17 09:16:55     Checking that the metadata strains match the sequence names
    2022-06-17 09:16:55     SUCCESS: Strain column matches sequence names
    2022-06-17 09:16:55     Creating new profile directory (my_profiles/custom-hpc)
    2022-06-17 09:16:55     Creating build file (my_profiles/custom-hpc/builds.yaml)
    2022-06-17 09:16:55     Adding default input data (defaults/inputs.yaml)
    2022-06-17 09:16:55     Adding custom input data (data/custom)
    2022-06-17 09:16:55     Adding `custom` as a build
    2022-06-17 09:16:55     Creating system configuration (my_profiles/custom-hpc/config.yaml)
    2022-06-17 09:16:55     Adding default HPC system resources
    2022-06-17 09:16:56     Done! The custom-hpc profile is ready to be run with:

                            snakemake --profile my_profiles/custom-hpc
    ```

2. Edit `my_profiles/custom-hpc/config.yaml` to specify the number of `jobs` and `default-resources` to use.

    ```yaml
    # Maximum number of jobs to run
    jobs : 4

    # Default resources for a SINGLE JOB
    default-resources:
    - cpus=64
    - mem_mb=64000
    - time_min=720
    ```

3. Dispatch the workflow using the slurm wrapper script:

    ```bash
    scripts/slurm.sh --profile my_profiles/custom-hpc
    ```

    > - **Tip**: Display log of most recent workflow: `cat $(ls -t logs/ncov-recombinant/*.log | head -n 1)`

4. Use the `--help` parameter to get additional options for SLURM dispatch.

    ```bash
    scripts/slurm.sh --help
    ```

    ```text
    usage: bash slurm.sh [-h] [--profile PROFILE] [--conda-env CONDA_ENV] [--target TARGET] [--cpus CPUS] [--mem MEM]

            Dispatch a Snakemake pipeline using SLURM.

            Required arguments:
                    --profile PROFILE                Snakemake profile to execute (ex. profiles/tutorial-hpc)

            Optional arguments:
                    --conda-env CONDA_ENV            Conda environment to use. (default: ncov-recombinant)
                    --target TARGET                  Snakemake target(s) to execute (default: all)
                    --cpus CPUS                      CPUS to use for the main pipeline. (default: 1)
                    --mem MEM                        Memory to use for the ain pipeline. (default: 4GB)
                    -h, --help                       Show this help message and exit.
    ```

## FAQ

1. What do I do if the workflow won't run because the directory is "locked"?

    ```bash
    snakemake --profile profiles/tutorial --unlock
    ```

1. How do I troubleshoot workflow errors?

    - Start with investigating the logfile of the rule that failed.

    ![rule_log_output](images/rule_log_output.png)

    - [Issue submissions](https://github.com/ktmeaton/ncov-recombinant/issues/33) are welcome and greatly appreciated!

1. How do I troubleshoot SLURM errors?

    - If the workflow was dispatched with `scripts/slurm.sh`, the master log will be stored at: `logs/ncov-recombinant/ncov-recombinant_<date>_<jobid>.log`

    > - **Tip**: Display log of most recent workflow: `cat $(ls -t logs/ncov-recombinant/*.log | head -n 1)`

1. How do I include more of my custom metadata columns into the linelists?

    - By default, the mandatory columns `strain`, `date`, and `country` will appear from your metadata.
    - Extra columns can be supplied as a parameter to `summary` in your `builds.yaml` file.
    - In the following example, the columns `division`, and `genbank_accession` will be extracted from your input `metadata.tsv` file and included in the final linelists.

    ```yaml
    - name: controls
      metadata: data/controls/metadata.tsv
      sequences: data/controls/sequences.fasta

      summary:
        extra_cols:
          - genbank_accession
          - division
    ```

1. How do I cleanup all the output from a previous run?

    ```bash
    snakemake --profile profiles/tutorial --delete-all-output
    ```

1. Why are "positive" sequences missing from the plots and slides?

    - First check and see if they are in `plots_historical` and `report_historical` which summarize all sequences regardless of collection date.
    - The most likely reason is that these sequences fall outside of the reporting period.
    - The default reporting period is set to 16 weeks before the present.
    - To change it for a build, add custom `plot` parameters to your `builds.yaml` file.

    ```yaml
    - name: custom
      metadata: data/custom/metadata.tsv
      sequences: data/custom/sequences.fasta

      plot:
        min_date: "2022-01-10"
        max_date: "2022-04-25" # Optional, can be left blank to use current date
    ```

1. Where can I find the plotting data?

    - A data table is provided for each plot:

        - Plot: `results/tutorial/plots/lineage.png`
        - Table: `results/tutorial/plots/lineage.tsv`
        - The rows are the epiweek, and the columns are the categories (ex. lineages)

## Credits

[ncov-recombinant](https://github.com/ktmeaton/ncov-recombinant) is built and maintained by [Katherine Eaton](https://ktmeaton.github.io/) at the [National Microbiology Laboratory (NML)](https://github.com/phac-nml) of the Public Health Agency of Canada (PHAC).

<table>
  <tr>
    <td align="center"><a href="https://ktmeaton.github.io"><img src="https://s.gravatar.com/avatar/0b9dc28b3e64b59f5ce01e809d214a4e?s=80" width="100px;" alt=""/><br /><sub><b>Katherine Eaton</b></sub></a><br /><a href="https://github.com/ktmeaton/ncov-recombinant/commits?author=ktmeaton" title="Code">💻</a> <a href="https://github.com/ktmeaton/ncov-recombinant/commits?author=ktmeaton" title="Documentation">📖</a> <a href="#design-ktmeaton" title="Design">🎨</a> <a href="#ideas-ktmeaton" title="Ideas, Planning, & Feedback">🤔</a> <a href="#infra-ktmeaton" title="Infrastructure (Hosting, Build-Tools, etc)">🚇</a> <a href="#maintenance-ktmeaton" title="Maintenance">🚧</a></td>
  </tr>
</table>

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/nextstrain/nextclade"><img src="https://avatars.githubusercontent.com/u/22159334?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Nextstrain (Nextclade)</b></sub></a><br /><a href="#data-nextstrain" title="Data">🔣</a> <a href="#plugin-nextstrain" title="Plugin/utility libraries">🔌</a></td>
    <td align="center"><a href="https://github.com/lenaschimmel/sc2rf"><img src="https://avatars.githubusercontent.com/u/1325019?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Lena Schimmel (sc2rf)</b></sub></a><br /><a href="#plugin-lenaschimmel" title="Plugin/utility libraries">🔌</a></td>
    <td align="center"><a href="https://github.com/yatisht/usher"><img src="https://avatars.githubusercontent.com/u/34664884?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Yatish Turakhia (UShER)</b></sub></a><br /><a href="#data-yatisht" title="Data">🔣</a> <a href="#plugin-yatisht" title="Plugin/utility libraries">🔌</a></td>
    <td align="center"><a href="https://github.com/yatisht/usher"><img src="https://avatars.githubusercontent.com/u/186983?v=4" width="100px;" alt=""/><br /><sub><b>Angie Hinrichs (UShER)</b></sub></a><br /><a href="#data-AngieHinrichs" title="Data">🔣</a> <a href="#plugin-AngieHinrichs" title="Plugin/utility libraries">🔌</a></td>
    <td align="center"><a href="https://www.inspq.qc.ca/en/auteurs/2629/all"><img src="https://i1.rgstatic.net/ii/profile.image/278724097396748-1443464411327_Q128/Benjamin-Delisle.jpg" width="100px;" alt=""/><br /><sub><b>Benjamin Delisle</b></sub></a><br /><a href="https://github.com/ktmeaton/ncov-recombinant/issues?q=author%3Abenjamindeslisle" title="Bug reports">🐛</a> <a href="https://github.com/ktmeaton/ncov-recombinant/commits?author=benjamindeslisle" title="Tests">⚠️</a></td>
  </tr>
  <tr>
    <td align="center"><a href="https://ca.linkedin.com/in/dr-vani-priyadarsini-ikkurti-4a2ab676"><img src="https://media-exp1.licdn.com/dms/image/C5603AQHaG8Xx4QLXSQ/profile-displayphoto-shrink_200_200/0/1569339145568?e=2147483647&v=beta&t=3WrvCciW-x8J3Aw4JHGrWOpuqiikrrGV2KsDaISnHIw" width="100px;" alt=""/><br /><sub><b>Vani Priyadarsini Ikkurthi</b></sub></a><br /><a href="https://github.com/ktmeaton/ncov-recombinant/issues?q=author%3Avanipriyadarsiniikkurthi" title="Bug reports">🐛</a> <a href="https://github.com/ktmeaton/ncov-recombinant/commits?author=vanipriyadarsiniikkurthi" title="Tests">⚠️</a></td>
    <td align="center"><a href="https://ca.linkedin.com/in/mark-horsman-52a14740"><img src="https://ui-avatars.com/api/?name=Mark+Horsman" width="100px;" alt=""/><br /><sub><b>Mark Horsman</b></sub></a><br /><a href="#ideas-markhorsman" title="Ideas, Planning, & Feedback">🤔</a> <a href="#design-markhorsman" title="Design">🎨</a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
