# glpkAPI

R interface to the C API of the GNU Linear Programming Kit (GLPK); requires GLPK (>= 4.42).

Compatibility updates by Renova.

Version 1.3.4.9001 removes the stale optional `Date` field from `DESCRIPTION`, rewrites the bracketed `test [ ... -a ... -o ... ]` conditions in `configure.ac` as separate `test` commands joined by `&&` and `||`, and changes the `$r_{ii}$` and `$s_{jj}$` scale-factor notation in four help pages to `\code{ii}` and `\code{jj}` markup. The generated `configure` script is not part of the change; no R or C code was changed.

## Install

From R-universe:

```r
install.packages("glpkAPI", repos = c(
  Renova = "https://renova-r.r-universe.dev",
  CRAN = "https://cloud.r-project.org"
))
```

Or the exact release tag:

```r
install.packages("remotes")
remotes::install_github("renova-r/glpkAPI@renova-1.3.4.9001")
```

[Release notes](NEWS.md) · [Renova package directory](https://github.com/renova-r/revival-catalog)

glpkAPI was written by Gabriel Gelius-Dietrich, with Mayo Roettger and Louis Luangkesorn; Mihail Anton is the maintainer. License: GPL-3 | file LICENSE (see [LICENSE](LICENSE), the unmodified GNU GPL v3 text). Source files keep their "Copyright (C) 2011-2014 Gabriel Gelius-Dietrich" notices.
