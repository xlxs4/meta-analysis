<div align="center">
<p>
    <a href="https://benchling.com/organizations/acubesat/">Benchling 🎐🧬</a> &bull;
    <a href="https://gitlab.com/acubesat/documentation/cdr-public/-/blob/master/DDJF/DDJF_PL.pdf?expanded=true&viewer=rich">DDJF_PL 📚🧪</a> &bull;
    <a href="https://spacedot.gr/">SpaceDot 🌌🪐</a> &bull;
    <a href="https://acubesat.spacedot.gr/">AcubeSAT 🛰️🌎</a>
</p>
</div>

## Description

Coming soon :tm:

## Table of Contents

<details>
<summary>Click to expand</summary>

- [Description](#description)
- [Table of Contents](#table-of-contents)
- [Getting Started](#getting-started)
  - [TL;DR](#tldr)
  - [Docker](#docker)
  - [Grab the Docker image](#grab-the-docker-image)
  - [Create a container](#create-a-container)
  - [Start it up](#start-it-up)
  - [Get in](#get-in)
  - [Clone the repository](#clone-the-repository)
  - [Development tips](#development-tips)
- [Script Rundown](#script-rundown)
  - [Input](#input)

</details>


## Getting Started

### TL;DR

1. `docker pull xlxs4/meta-analysis:latest` to grab the image
2. `docker run -it --entrypoint /bin/bash xlxs4/meta-analysis` to spawn a container running the image and use bash
3. `git clone --depth 1 https://gitlab.com/acubesat/su/bioinformatics/meta-analysis.git`
4. `cd meta-analysis`
5. `Rscript differential-expression-analysis/src/flocculation.R --qc --plots --feather`

### Docker

First of all, you'll need [Docker](https://www.docker.com/) :whale:
You can download and install [Docker for Desktop](https://www.docker.com/products/docker-desktop) for starters

### Grab the Docker image

* Open your favorite terminal. If you think you don't have one, you're on Windows
  In that case, get [Windows Terminal](https://aka.ms/terminal)
* You can pull the image straight from the cloud: `docker pull xlxs4/meta-analysis:latest`
* ...  or you can build it yourself (something like `docker build -t meta-analysis .`)

### Create a container

* `docker run -i -t xlxs4/meta-analysis:latest /bin /bash` 
* `Ctrl + d` to exit. [:camera:](https://prnt.sc/1bu4lcr) [:camera:](https://prnt.sc/1bu4p8s)
*  :bulb: Hint: click on a :camera: when you find one

### Start it up

* Everything below will help you get an IDE running inside the container.
  The cross-platform [Visual Studio Code](https://code.visualstudio.com/) will be used.
  If you want to use something else, you probably know what you're doing, so go to the [TL;DR](#tldr) section
* Get the VSCode [`remote-containers`](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) [:camera:](https://prnt.sc/1bueb0d)
  While not necessary, you could grab the [Docker extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) as well [:camera:](https://prnt.sc/1btzhi3)
* `Docker Containers: Start` [:camera:](https://prnt.sc/1bty5z9)  -> `Individual Containers` [:camera:](https://prnt.sc/1btyb4t) -> <container_hash>  [:camera:](https://prnt.sc/1btyimw)

### Get in

* `Open a Remote Window` [:camera:](https://prnt.sc/1btzqbd) -> `Attach to Container` [:camera:](https://prnt.sc/1btzy1h) -> <container_hash> [:camera:](https://prnt.sc/1bu0ffu)

### Clone the repository

* `Explorer` (`Ctrl + Shift + E`) -> `Clone Repository` [:camera:](https://prnt.sc/1bu52pf)
* Clone the `meta-analysis` repository [:camera:](https://prnt.sc/1bu5ttp)  [:camera:](https://prnt.sc/1bu602g)
* Open the cloned repository [:camera:](https://prnt.sc/1bu63we)  [:camera:](https://prnt.sc/1bu6990)

### Development tips

* Installing `python-pip3` and using it to install [`radian`](https://github.com/randy3k/radian) inside the container is highly recommended
* If you want to update the image, e.g. for CI/CD purposes, but have made the changes to the `renv` lockfile inside an attached running Docker container, you can `git clone` the repository in your main working environment, and then grab the updated `renv.lock` file, like this: `docker cp <container_name>:"<path_to_renv.lock>" .`
  Then, just do `docker build -t meta-analysis .`, `docker image tag meta-analysis:latest <repo_name>/meta-analysis:latest` and `docker push <repo_name>/meta-analysis:latest`
* [`conflicted`](https://www.tidyverse.org/blog/2018/06/conflicted/) is a great `tidyverse` package to check for conflicts arising from ambiguous function names. From within the container, open the R terminal (e.g. `radian`), and `install.packages("conflicted")`. Then, you can just load it (`library(conflicted)`) in the running session. To re-check for conflicts, simply run `conflicted::conflict_scout()`

## Script Rundown

### Input

[`flocculation.R`](https://gitlab.com/acubesat/su/bioinformatics/meta-analysis/-/blob/master/differential-expression-analysis/src/flocculation.R) uses [`docopt`](https://github.com/docopt/docopt.R) :books: to interface with the CLI and parse user input. You can read more on `docopt` [here](http://docopt.org/).

The complete interface for `flocculation.R` can be found inside the script:

```r
"DEA script for GLDS-62 GeneLab entry (raw data).

Usage:
  flocculation.R [-q | --no-color] [--time]
  flocculation.R (-h | --help)
  flocculation.R --version
  flocculation.R --qc [-r] [-n] [-t] [--plots] [-q | --no-color] [--feather] [--time]
  flocculation.R --plots [-q | --no-color] [--feather] [--time]
  flocculation.R --feather [--time]

Options:
  -h --help     Show this screen.
  --version     Show version.
  --qc          Produce QC reports.
  --plots       Produce DEA plots.
  --feather     Save result tibbles as (arrow) feather files.
  --time        Output script execution time.

" -> doc
```

First, some `docopt` magic :crystal_ball::

* `[ ]` square brackets indicate **optional elements**
* `( )` the parentheses mark **required elements**
* ` | ` the pipe operator is used to separate **mutually-exclusive elements**
* `-o` short options can be stacked: `-xyz ≡ -x -y -z`

So, you have a couple of different options:

* Call the script without any additional elements (`flocculation.R`). This will run the script beginning-to-end, excluding all code generating output, be it QC reports, various plots or `arrow:feather` files. Useful mainly to shorten execution time when developing, and to make profiling easier
* If you want to generate QC reports, you can set the `--qc` option. Note that each time the QC-generating code is called, new reports will be created regardless of whether there are already existing ones. To change this setting, open the script and remove the `force = TRUE` argument inside `arrayQualityMetrics()`. To speed up execution time, you can opt for running only a subset of the QC analyses, namely you can:
    * Additionally pass `-r` (`--qc -r`) to generate QC for the raw data (pre-RMA)
    * pass `-n` to generate QC for the normalized data (directly after RMA)
    * pass `-t` to generate QC for the statistical test results (after the `eBayes()` call on the linear model fit)
    * Example: `flocculation.R --qc -rt`
* To generate plots (as `.pdf`s), you can set the `--plots` option. Said plots include:
    * Various GSEA graphs
    * [Volcanoplot](https://rdrr.io/bioc/limma/man/volcanoplot.html)
    * [MD plot](https://www.rdocumentation.org/packages/limma/versions/3.28.14/topics/plotMD) (log2 fold-change vs mean log2 expression)
    * [Venn diagram](http://web.mit.edu/~r/current/arch/i386_linux26/lib/R/library/limma/html/venn.html)
    * [Heatmap](https://rdrr.io/bioc/limma/man/coolmap.html) (`coolmap`)
    * [EnhancedVolcano](https://bioconductor.org/packages/release/bioc/vignettes/EnhancedVolcano/inst/doc/EnhancedVolcano.html)
* `flocculation.R` uses `tibble`s from [tidyverse](https://tibble.tidyverse.org/) to hold various dataframe-like results. You can have them be saved on-disk as `.feather` [files](https://arrow.apache.org/docs/python/feather.html) by passing `--feather`
* If you want to time the script call without using some external wrapper program, set the `--time` option
* [`logger`](https://daroczig.github.io/logger/) is used to log various events to the user. If you want to disable all `INFO`/`WARN` log messages, set the `-q` option
* By default, messages logged are colorful, using `glue` to format the message and ANSI escape codes, depending on the [`crayon`](https://github.com/r-lib/crayon) package. If that's not to your taste, feel free to use `--no-color`