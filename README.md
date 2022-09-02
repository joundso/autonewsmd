# autonewsmd

<!-- badges: start -->
[![](https://www.r-pkg.org/badges/version/autonewsmd)](https://cran.r-project.org/package=autonewsmd)
[![CRAN checks](https://cranchecks.info/badges/summary/autonewsmd)](https://cran.r-project.org/web/checks/check_results_autonewsmd.html)
[![](http://cranlogs.r-pkg.org/badges/grand-total/autonewsmd?color=blue)](https://cran.r-project.org/package=autonewsmd)
[![](http://cranlogs.r-pkg.org/badges/last-month/autonewsmd?color=blue)](https://cran.r-project.org/package=autonewsmd)
[![Dependencies](https://tinyverse.netlify.com/badge/autonewsmd)](https://cran.r-project.org/package=autonewsmd)
[![R build status](https://github.com/kapsner/autonewsmd/workflows/R%20CMD%20Check%20via%20{tic}/badge.svg?branch=main)](https://github.com/kapsner/autonewsmd/actions)
[![R build status](https://github.com/kapsner/autonewsmd/workflows/lint/badge.svg?branch=main)](https://github.com/kapsner/autonewsmd/actions)
[![R build status](https://github.com/kapsner/autonewsmd/workflows/test-coverage/badge.svg?branch=main)](https://github.com/kapsner/autonewsmd/actions)
[![codecov](https://codecov.io/gh/kapsner/autonewsmd/branch/main/graph/badge.svg)](https://app.codecov.io/gh/kapsner/autonewsmd)
<!-- badges: end -->

The purpose of the `autonewsmd` R package is to bring the power of conventional commit messages to the R community. There is no need anymore to tediously maintain a changelog file manually. If you are using conventional commit messages, `autonewsmd` will do that for you and automatically generate a human readable changelog file directly from the repository's git history.

Conventional commit messages ([https://www.conventionalcommits.org/en/v1.0.0/](https://www.conventionalcommits.org/en/v1.0.0/)) come with some easy rules to create human readable commit messages for a git history. One advantage is that following these conventions, these messages are also machine readable and automated tools can run on top of them in order to, e.g., generate beautiful changelogs out of them. Similar tools for other languages are, for example, [`auto-changelog`](https://github.com/cookpete/auto-changelog) for JavaScript and [`auto-changelog`](https://github.com/KeNaCo/auto-changelog) for Python.

## Installation

You can install `autonewsmd` with:

```{r}
install.packages("autonewsmd")
```

You can install the development version of `autonewsmd` with:

```{r}
install.packages("remotes")
remotes::install_github("kapsner/autonewsmd")
```

## Example

First of all, create a small repository with some commit messages.

```{r}
library(autonewsmd)

# (Example is based on the public examples from the `git2r` R package)
## Initialize a repository
path <- file.path(tempdir(), "autonewsmd")
dir.create(path)
repo <- git2r::init(path)

## Config user
git2r::config(repo, user.name = "Alice", user.email = "alice@example.org")
git2r::remote_set_url(repo, "foobar", "https://example.org/git2r/foobar")

## Write to a file and commit
lines <- "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do"
writeLines(lines, file.path(path, "example.txt"))

git2r::add(repo, "example.txt")
git2r::commit(repo, "feat: new file example.txt")

## Write again to a file and commit
Sys.sleep(2) # wait two seconds, otherwise, commit messages have same time stamp
lines2 <- paste0(
  "eiusmod tempor incididunt ut labore et dolore magna aliqua. ",
  "Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris ",
  "nisi ut aliquip ex ea commodo consequat."
)
write(lines2, file.path(path, "example.txt"), append = TRUE)

git2r::add(repo, "example.txt")
git2r::commit(repo, "refactor: added second phrase")

## Also add a tag here
git2r::tag(repo, "v0.0.1")
```

Then, instantiate an `autonewsmd` object. Here, you must provide the `repo_name` (this argument is used to compose the title of the changelog file). The `repo_path`-argument can be provided optionally and defaults to the current working directory (`"."`). The `repo_path` should be the root of a git repository. The `$generate()`-method creates a list with all commit messages that is used for rendering the changelog file.

```{r}
an <- autonewsmd$new(repo_name = "TestRepo", repo_path = path)
an$generate()
```

Executing the `$write()`-method, the changelog is written to the path specified with the `repo_path`-argument. If `force = FALSE` (the default), a dialog is prompted to ask the user if the file should be (over-) written.

```{r writenmd}
an$write(force = TRUE)
```

Now, we can verify that the file `NEWS.md` also appears in the git folder and check its content.

```{r}
list.files(path)
#> [1] "example.txt" "NEWS.md"
```

```{r}
newsmd <- readLines(file.path(path, "NEWS.md"))
newsmd
#>  [1] "# TestRepo NEWS"                                                                                
#>  [2] ""                                                                                               
#>  [3] "## v0.0.1 (2022-08-27)"                                                                         
#>  [4] ""                                                                                               
#>  [5] "#### New features"                                                                              
#>  [6] ""                                                                                               
#>  [7] "-   new file"                                                                                   
#>  [8] "    ([22b8453](https://example.org/git2r/foobar/tree/22b845346a0f3686d79eb86445af6be71dc86da6))"
#>  [9] ""                                                                                               
#> [10] "#### Refactorings"                                                                              
#> [11] ""                                                                                               
#> [12] "-   added second phrase"                                                                        
#> [13] "    ([ec510eb](https://example.org/git2r/foobar/tree/ec510ebb465d25ab7ad27e8b637cf4113b55cbdf))"
#> [14] ""                                                                                               
#> [15] "Full set of changes:"                                                                           
#> [16] "[`22b8453...v0.0.1`](https://example.org/git2r/foobar/compare/22b8453...v0.0.1)"
```

## Used By 

**(If you are using `autonewsmd` and you like to have your repository listed here, please add the link pointing to the changelog file to this [`README.md`](./README.md)) and create a pull request.)**

[autonewsmd](https://github.com/kapsner/autonewsmd/blob/main/NEWS.md), [sjtable2df](https://github.com/kapsner/sjtable2df/blob/main/NEWS.md), [DIZtools](https://github.com/miracum/misc-diztools/blob/main/NEWS.md), [DIZutils](https://github.com/miracum/misc-dizutils/blob/master/NEWS.md), [DQAstats](https://github.com/miracum/dqa-dqastats/blob/master/NEWS.md), [DQAgui](https://github.com/miracum/dqa-dqagui/blob/master/NEWS.md), [miRacumDQA](https://github.com/miracum/dqa-miracumdqa/blob/master/NEWS.md)

## TODOs:

- add options for formatting the changelog
- add support for [Commit message with ! to draw attention to breaking change](https://www.conventionalcommits.org/en/v1.0.0/#commit-message-with--to-draw-attention-to-breaking-change)
- add support for [Commit message with scope](https://www.conventionalcommits.org/en/v1.0.0/#commit-message-with-scope)
- switch rendering of md to quarto -> challenges: currently, quarto does not allow to pass objects from the environment to the document that is rendered; hence, one needs to find a solution how to pass the `repo_list`, which is created in the `get_git_log` function (as well as some other objects) on to the *.qmd file for properly rendering the document.
