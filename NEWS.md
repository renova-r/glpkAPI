# glpkAPI 1.3.4.9001

Renova release, based on CRAN glpkAPI 1.3.4 (published 2022-11-10).

- `DESCRIPTION`: the stale optional `Date: 2022-11-10` field was removed.
- `configure.ac`: `test [ -n ... -a -n ... ]` and `-o` conditions are now separate `test` commands joined with `&&` and `||`. Only `configure.ac` changed; the shipped generated `configure` script is untouched.
- `man/getRiiGLPK.Rd`, `man/getSjjGLPK.Rd`, `man/setRiiGLPK.Rd`, `man/setSjjGLPK.Rd`: `$r_{ii}$` and `$s_{jj}$` became `$r_\code{ii}$` and `$s_\code{jj}$` (Rd markup fix). No R or C code was changed.
- `DESCRIPTION`: `Version` set to `1.3.4.9001`; "Compatibility updates by Renova." appended to `Description`.
- The stale `MD5` file was removed. `README.md`, `NEWS.md` and `.Rbuildignore` were added.
