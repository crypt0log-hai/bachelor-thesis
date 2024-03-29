$if(readme_title)$
# $readme_title$
$else$
# Bachelor Thesis
$endif$
[![License](https://img.shields.io/github/license/$username$/$repo.name$.svg)](https://github.com/$username$/$repo.name$/blob/master/LICENSE)
[![Build Status](https://travis-ci.org/Skogarmadr/bachelor-thesis.svg?branch=master)](https://travis-ci.org/Skogarmadr/bachelor-thesis)
[![PDF](https://img.shields.io/badge/PDF-latest-blue.svg?style=flat)](https://github.com/$username$/$repo.name$/blob/master-pdf/$generated_file$.pdf)

This repository contains the report for the bachelor thesis:



<p align="center" style="font-size:larger;">
<i>$title$</i>
</p>
$if(subtitle)$
<p align="center" style="font-size:large;">
<i>$subtitle$</i>
</p>
$endif$

written by $author$, under the supervision of $advisor.title$ $advisor.name$ (advisor) $if(coadvisor)$ and  $coadvisor.title$ $coadvisor.name$ (coadvisor)$endif$, submitted to the $faculty$ of the $university$.


## License
This thesis is made available under the $license.name$. A copy of the full license is available in the [LICENSE](/LICENSE) file.

## PDF version
A rendered PDF version of the thesis is automatically generated using [travis-ci](https://travis-ci.org/Skogarmadr/bachelor-thesis) and pushed back to this repository at [master-pdf:$generated_file$.pdf](https://github.com/$username$/$repo.name$/blob/master-pdf/$generated_file$.pdf).

Generating the pdf locally requires pandoc (>=2.1.3), XeLaTeX, and Bibtex. Then, run:

``` bash
make
```

This will generate the report in pdf under the name ``$generated_file$.pdf` in the root folder.

To change the name or the generated pdf file, simply overwrite the `OUTPUT` variable (without the extension):

``` bash
make OUTPUT_FILE="new_file"
```

## Fast build
A "fast build" which only runs `pandoc` and `xelatex` once without `makeglossaries` or `bibtex` can be executed to quickly generate a PDF without references or glossary, with:

``` bash
make fast
```

> The `OUTPUT_FILE` variable can also be defined in this case.

## Template
The template used to generate the report is available at [template/template.tex](template/template.tex).

It has been adapted from the official [USI Informatics Master Thesis template](http://www.inf.usi.ch/msc-thesis-stylesheet-159301.zip). Note that this adaptation is only compatible for the master thesis and the PHD related elements of the template have been removed.

## Debug
The `tex` files and associated generated intermediary  files such as `.aux` are kept in the `BUILD` directory (default: `./build`) and can be inspect in case of issues when generating the PDF.

---
$if(license.badge)$>$license.badge$$endif$

<small><i>This README was generated automatically using the information in [`metadata.yml`](metadata.yml) via `make readme`.</i></small>

# Thanks to
Jacques Dafflon, for the [template](https://github.com/0xjac/master-thesis).