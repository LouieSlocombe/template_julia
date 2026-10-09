# Development tools

This directory contains a Julia environment for CoverageTools and the coverage
gate script. Keeping it separate from the root project avoids adding developer
tools to the package's runtime dependencies. It is intentionally not part of
the package workspace, so tool releases can never block resolving the package
or its tests. Aqua, ExplicitImports, and Test belong to the `test` workspace
project and are installed automatically by `Pkg.test()`.

Install Julia with [Juliaup](https://julialang.org/downloads/), then run all of
these commands from the repository root:

```bash
julia --project=build_tools -e 'using Pkg; Pkg.instantiate()'
julia --project=build_tools -e 'using CoverageTools; clean_folder("src")'
julia --project=. -e 'using Pkg; Pkg.test(coverage=true)'
julia --project=build_tools build_tools/coverage.jl
```

Coverage is checked for the package source in `src/`. The gate fails on missing
coverage or uncovered executable lines and writes an LCOV report to `lcov.info`.
Uncalled functions count as uncovered, including in files without a `.cov`
report. The test suite and development scripts are not part of that coverage
percentage; subprocesses started by the tests with `Base.julia_cmd()` are
tracked.

Formatting uses [Runic](https://github.com/fredrikekre/Runic.jl), which runs as
a Pkg app, a pre-commit hook, and a GitHub Action rather than from this
environment. See the main [README](../README.md#formatting).

To update the tools within their compatibility bounds:

```bash
julia --project=build_tools -e 'using Pkg; Pkg.update()'
```

Dependabot proposes `[compat]` updates for the root, `test`, and `build_tools`
projects, for GitHub Actions, and for pre-commit hooks. Runic minor releases
can change formatting; when CI reports new formatting, run `runic --inplace .`,
review the result, and commit it. This library template does not commit
generated manifests.

Git hooks are optional and need a separate installation of
[pre-commit](https://pre-commit.com/#installation) 4.1 or newer, which can
install Julia hooks. Once it is on your `PATH`:

```bash
pre-commit install
pre-commit run --all-files
```
