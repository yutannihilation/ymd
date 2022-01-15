
<!-- README.md is generated from README.Rmd. Please edit that file -->
# ymd

<!-- badges: start -->
[![R-CMD-check](https://github.com/shrektan/ymd/workflows/R-CMD-check/badge.svg)](https://github.com/shrektan/ymd/actions) [![CRAN status](https://www.r-pkg.org/badges/version/ymd)](https://CRAN.R-project.org/package=ymd) [![Downloads from the RStudio CRAN mirror](https://cranlogs.r-pkg.org/badges/ymd)](https://cran.r-project.org/package=ymd) [![Rust Code Coverage](https://coveralls.io/repos/github/shrektan/ymd/badge.svg?branch=main)](https://coveralls.io/github/shrektan/ymd?branch=main) <!-- badges: end -->

Convert 'YMD' format number or string to Date efficiently, e.g., `211225` to `as.Date("2021-12-25")`, using Rust's standard library. It also provides helper functions to handle Date, e.g., quick finding the beginning or end of the given period, adding months to Date, etc.

It's similar to the `lubridate` package but is much lighter and focuses only on Date objects.

## Installation

### Binary version (no Rust toolchain required)

The binary package is provided by CRAN. So, if you are on Windows or macOS, the package can be installed via:

``` r
install.packages("ymd")
```

If you are on Linux, you can try to use the [RSPM (RStudio Package Manager) repo](https://packagemanager.rstudio.com) provided by RStudio PBC, via (remember to choose the correct binary repo URL for your platform):

``` r
install.packages("ymd", repos = "{RSPM-Repo-URL}")
```

In addition, you may download the binary package file generated by GitHub Action from [the release page](https://github.com/shrektan/ymd/releases) and install via:

``` r
install.packages("{the-downloaded-binary-pkg-file}", repos = NULL)
```

| artifact                                   |      platform|
|:-------------------------------------------|-------------:|
| ymd\_0.0.1.tgz                             |   macOS Intel|
| ymd\_0.0.1.zip                             |       Windows|
| ymd\_0.0.1\_R\_x86\_64-pc-linux-gnu.tar.gz |  Ubuntu-18.04|

### Source version (Rust toolchain required)

If you want to build the dev version from source, you'll need the Rust toolchain, which can be installed following [the instructions from the Rust book](https://doc.rust-lang.org/book/ch01-01-installation.html).

After that, you can build the package via:

``` r
remotes::install_github("ymd")
```

## Some use cases and benchmarks

``` r
print_bmk <- function(x) {
  x[[1]] <- format(x[[1]])
  x[[5]] <- format(x[[5]])
  rnd <- \(v) if (is.numeric(v)) round(v, 1) else v
  x[, 1:9] |> lapply(rnd) |> as.data.frame() |> knitr::kable() |> print()
}
x <- c("210101", "21/02/03", "89-1-03", "1989.03.05", "01 02 03")
x <- rep(x, 100)
bench::mark(
  ymd::ymd(x),
  lubridate::ymd(x), time_unit = "us"
) |> print_bmk()
```

| expression        |     min|  median|  itr.sec| mem\_alloc |  gc.sec|  n\_itr|  n\_gc|  total\_time|
|:------------------|-------:|-------:|--------:|:-----------|-------:|-------:|------:|------------:|
| ymd::ymd(x)       |    44.2|    44.9|  21975.6| 219.95KB   |     0.0|   10000|      0|     455049.8|
| lubridate::ymd(x) |  1539.5|  1579.0|    631.2| 8.23MB     |    17.5|     289|      8|     457833.8|

``` r

x <- c(210101, 210224, 211231, 19890103)
x <- rep(x, 100)
bench::mark(
  ymd::ymd(x),
  lubridate::ymd(x), time_unit = "us"
) |> print_bmk()
```

| expression        |     min|  median|  itr.sec| mem\_alloc |  gc.sec|  n\_itr|  n\_gc|  total\_time|
|:------------------|-------:|-------:|--------:|:-----------|-------:|-------:|------:|------------:|
| ymd::ymd(x)       |    11.9|    12.5|  78291.3| 3.17KB     |     0.0|   10000|      0|     127728.0|
| lubridate::ymd(x) |  1696.3|  1732.2|    573.8| 373.41KB   |    19.6|     264|      9|     460098.1|

``` r

x <- c("2021-01-01", "2022-12-31", "1995-03-22")
x <- rep(x, 100)
bench::mark(
  ymd::ymd(x),
  lubridate::ymd(x), time_unit = "us",
  as.Date(x)
) |> print_bmk()
```

| expression        |    min|  median|  itr.sec| mem\_alloc |  gc.sec|  n\_itr|  n\_gc|  total\_time|
|:------------------|------:|-------:|--------:|:-----------|-------:|-------:|------:|------------:|
| ymd::ymd(x)       |   32.5|    33.3|  29578.5| 2.39KB     |     0.0|   10000|      0|     338082.9|
| lubridate::ymd(x) |  783.8|   805.4|   1228.8| 201.1KB    |    21.9|     562|     10|     457341.8|
| as.Date(x)        |  663.5|   674.0|   1473.2| 87.54KB    |     0.0|     737|      0|     500287.8|

``` r

x <- ymd::ymd(210515) + 1:100
bench::mark(
  ymd::eop$tm(x),
  lubridate::ceiling_date(x, "month") - 1, time_unit = "us"
) |> print_bmk()
```

| expression                               |   min|  median|   itr.sec| mem\_alloc |  gc.sec|  n\_itr|  n\_gc|  total\_time|
|:-----------------------------------------|-----:|-------:|---------:|:-----------|-------:|-------:|------:|------------:|
| ymd::eop$tm(x)                           |   5.5|     5.9|  164688.1| 19.3KB     |    16.5|    9999|      1|      60714.8|
| lubridate::ceiling\_date(x, "month") - 1 |  94.5|    99.5|    9868.7| 255.1KB    |    35.4|    4462|     16|     452137.0|

``` r

`%m+%` <- lubridate::`%m+%`
x <- ymd::ymd(c(200115, 200131, 200229, 200331, 200401))
x <- rep(x, 100)
bench::mark(
  ymd::edate(x, 2),
  x %m+% months(2), time_unit = "us"
) |> print_bmk()
```

| expression       |    min|  median|  itr.sec| mem\_alloc |  gc.sec|  n\_itr|  n\_gc|  total\_time|
|:-----------------|------:|-------:|--------:|:-----------|-------:|-------:|------:|------------:|
| ymd::edate(x, 2) |   13.3|    13.9|  70065.3| 6.2KB      |     0.0|   10000|      0|     142724.0|
| x %m+% months(2) |  295.2|   307.7|   3230.1| 424.6KB    |    23.5|    1513|     11|     468403.5|

``` r
bench::mark(
  ymd::edate(x, -12),
  x %m+% months(-12), time_unit = "us"
) |> print_bmk()
```

| expression         |    min|  median|  itr.sec| mem\_alloc |  gc.sec|  n\_itr|  n\_gc|  total\_time|
|:-------------------|------:|-------:|--------:|:-----------|-------:|-------:|------:|------------:|
| ymd::edate(x, -12) |   13.4|    14.1|  69528.6| 3.95KB     |     0.0|   10000|      0|     143825.7|
| x %m+% months(-12) |  805.4|   838.2|   1152.7| 317.19KB   |    32.9|     491|     14|     425964.9|
