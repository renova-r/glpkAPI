# glpkAPI 1.3.4.9002

Platform compatibility patch to Renova release 1.3.4.9001, which is based on CRAN glpkAPI 1.3.4 (published 2022-11-10).

- `configure.ac`, `configure`: the GLPK selection of the published version (both `--with-glpk-include` and `--with-glpk-lib`, an explicit `--prefix`, the prefix of `glpsol`, otherwise `/usr` and `/usr/local`, with the same flags and order) is kept and tried first, now validated by compiling and linking a GLPK call. Only if it cannot be used (for example macOS without `/usr/include`, or a cross build such as WebAssembly where host headers in `/usr/include` shadow the cross compiler's) these fallbacks are tried, in order: `GLPK_INCLUDE`/`GLPK_LIB` from the environment or a single `--with-glpk-include`/`--with-glpk-lib`, `pkg-config glpk` (`$PKG_CONFIG` if set, else `<host>-pkg-config` when cross compiling, else `pkg-config`), the prefixes named in `PKG_CONFIG_PATH`/`EM_PKG_CONFIG_PATH` that contain `include/glpk.h`, and the compiler's default search path. Wherever the published configure worked it selects the same flags as before.
- `configure.ac`, `configure`: R's `LDFLAGS` are now used for the configure link tests unless `LDFLAGS` is set in the environment (an explicit value, even empty, is kept as before). The shipped `configure` was regenerated with the same generator as the original (GNU Autoconf 2.69 as patched by Debian/Ubuntu, so `--runstatedir` is kept); it now also contains the `test` rewrite of 1.3.4.9001. `PKG_CPPFLAGS` plus `PKG_LIBS` behave as before.
- No R or C code changed.
- `DESCRIPTION`: `Version` set to `1.3.4.9002`. `README.md` and `NEWS.md` updated.

# glpkAPI 1.3.4.9001

Renova release, based on CRAN glpkAPI 1.3.4 (published 2022-11-10).

- `DESCRIPTION`: the stale optional `Date: 2022-11-10` field was removed.
- `configure.ac`: `test [ -n ... -a -n ... ]` and `-o` conditions are now separate `test` commands joined with `&&` and `||`. Only `configure.ac` changed; the shipped generated `configure` script is untouched.
- `man/getRiiGLPK.Rd`, `man/getSjjGLPK.Rd`, `man/setRiiGLPK.Rd`, `man/setSjjGLPK.Rd`: `$r_{ii}$` and `$s_{jj}$` became `$r_\code{ii}$` and `$s_\code{jj}$` (Rd markup fix). No R or C code was changed.
- `DESCRIPTION`: `Version` set to `1.3.4.9001`; "Compatibility updates by Renova." appended to `Description`.
- The stale `MD5` file was removed. `README.md`, `NEWS.md` and `.Rbuildignore` were added.
