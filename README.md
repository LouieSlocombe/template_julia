# template-julia

A compact starting point for a Julia package, modelled on
[template-python](https://github.com/LouieSlocombe/template_python). It uses a
Pkg workspace, a `@main` entry point installable as a Pkg app, Runic formatting,
Aqua and ExplicitImports quality checks, Julia's `Test` standard library, full
source-line coverage, pre-commit, Dependabot, and GitHub Actions.

The example package exposes a tiny numerical API and a command-line entry point
so the template works end to end before you replace the sample code. It has no
external runtime dependencies.

## Requirements

- Julia 1.12 or newer in the 1.x series. Install it with
  [Juliaup](https://julialang.org/downloads/). Julia 1.12 is the first release
  with Pkg workspaces, Pkg apps, and `julia -m`.
- Optional: [pre-commit](https://pre-commit.com/#installation) 4.1 or newer for
  Git hooks.

CI tests the minimum supported Julia version and the latest stable Julia on
Linux, macOS, and Windows, plus the upcoming Julia prerelease on Linux as a
non-blocking early warning.

## Quick start

From the repository root, instantiate and precompile the package:

```bash
julia --project=. -e 'using Pkg; Pkg.instantiate(); Pkg.precompile()'
```

Run the example through the package's `@main` entry point:

```bash
julia --project=. -m TemplateJulia Ada
julia --project=. -m TemplateJulia --help
```

Omit the name to print `Hello, World!`. Quote names containing spaces, or use
`--` before a name beginning with a dash. The CLI returns status `2` for invalid
arguments.

### Install the command-line app

The `[apps]` section of `Project.toml` declares a `template-julia` executable.
Install it from your checkout so that source edits take effect immediately:

```bash
julia -e 'using Pkg; Pkg.Apps.develop(path=".")'
template-julia Ada
```

Pkg places the launcher in `~/.julia/bin`, which you need to add to your `PATH`
once. The launcher passes arguments before its first `--` to Julia, for example
`template-julia --threads=2 -- Ada`, so write `template-julia -- -- -h` to greet
a name that begins with a dash. Remove the app with
`julia -e 'using Pkg; Pkg.Apps.rm("template-julia")'`. Pkg apps are still
marked experimental by Julia.

### Use the library

Start Julia with `julia --project=.` and load the package:

```julia
using TemplateJulia

print_hello("Ada")
samples = line(-1.0, 1.0; num = 5)
println(samples)
```

`line` returns a `Vector{Float64}` including both interval endpoints. A sample
count of zero returns an empty vector; a count of one returns only the starting
value. Negative counts raise an `ArgumentError`. Public functions have docstrings
available through Julia's help mode, for example `?line`.

`TemplateJulia.main` is declared `public` but is deliberately not exported:
exporting an `@main` function would make it the entry point of any script that
runs `using TemplateJulia`.

To work on this package from another Julia project, use
`Pkg.develop(path="/path/to/template_julia")` in that project's environment.

## Development

Run the tests from the repository root:

```bash
julia --project=. -e 'using Pkg; Pkg.test()'
```

`Pkg.test()` installs the test dependencies and runs behavior tests, entry
point subprocess tests, selected type-inference checks, and package quality
checks. Aqua checks issues such as method ambiguities, undefined exports, stale
dependencies, and missing compatibility bounds. ExplicitImports checks that the
package does not rely on implicit imports or non-public names.

Test dependencies live in `test/Project.toml`, which the root `Project.toml`
includes as a [workspace](https://pkgdocs.julialang.org/v1/toml-files/#Workspaces)
project. Pkg resolves the package and its tests together into a single
`Manifest.toml`. Add a test dependency with:

```bash
julia --project=test -e 'using Pkg; Pkg.add("PackageName")'
```

### Formatting

Code is formatted with [Runic](https://github.com/fredrikekre/Runic.jl), which
has no configuration. Install it once as a Pkg app, then check or apply
formatting:

```bash
julia -e 'using Pkg; Pkg.Apps.add("Runic")'
runic --check --diff .
runic --inplace .
```

### Coverage

Install the separate coverage tools once, then run the coverage gate used by CI:

```bash
julia --project=build_tools -e 'using Pkg; Pkg.instantiate()'
julia --project=build_tools -e 'using CoverageTools; clean_folder("src")'
julia --project=. -e 'using Pkg; Pkg.test(coverage=true)'
julia --project=build_tools build_tools/coverage.jl
```

The gate requires 100% coverage of executable lines in `src/` and writes
`lcov.info`. Julia reports line coverage, so this is not the Python template's
branch-coverage metric. Clearing previous coverage first prevents stale runs
from hiding missing tests. Coverage reports do not require a hosted service or
secret token.

### Git hooks

Install the optional Git hooks once, then run them on all files:

```bash
pre-commit install
pre-commit run --all-files
```

The hooks check file hygiene, Runic formatting, and GitHub Actions security
with [zizmor](https://docs.zizmor.sh/). pre-commit installs Runic itself but
needs `julia` on your `PATH`. See [build_tools/README.md](build_tools/README.md)
for tool maintenance.

### Continuous integration

CI checks formatting, runs the tests on every supported platform, smoke tests
`julia -m`, the installed app, and loading the package from a fresh consumer
environment, and enforces the coverage gate. Julia packages are distributed as
source; there is no wheel-building step.

Actions are pinned to commit SHAs with the release tag in a comment. Dependabot
updates those pins, Julia `[compat]` bounds, and pre-commit hooks monthly, after
a seven-day cooldown on new releases.

## Project layout

```text
.
├── .github/
│   ├── dependabot.yml        # monthly updates for actions, Julia compat, hooks
│   └── workflows/ci.yml      # formatting, tests, smoke tests, coverage
├── build_tools/              # separate coverage environment and gate script
├── src/
│   ├── TemplateJulia.jl      # package module, exports, and public names
│   ├── cli.jl                # @main command-line entry point
│   └── main.jl               # example library functions
├── test/
│   ├── Project.toml          # test dependencies (workspace project)
│   └── runtests.jl           # behavior and package quality tests
├── .pre-commit-config.yaml   # file hygiene, Runic, and zizmor hooks
└── Project.toml              # package metadata, app, compatibility, workspace
```

## Use this template

After creating a repository from this template:

1. Rename `TemplateJulia` in `Project.toml`, `test/Project.toml`, the module,
   imports, tests, and the `-m TemplateJulia` commands in CI and this README.
   Rename `src/TemplateJulia.jl` to match the new package name. Rename the app
   under `[apps]` and update the CLI help text.
2. Generate a new package UUID with `julia -e 'using UUIDs; println(uuid4())'`
   and replace the UUID in both `Project.toml` and `test/Project.toml`.
3. Update the author, repository links, version, and license details.
4. Choose supported Julia versions in `[compat]`; CI follows it through the
   `min` version. Add runtime dependencies with `Pkg.add`, which records
   compatibility bounds for them. Remove the `[apps]` section and `src/cli.jl`
   if the package does not need a command line.
5. Replace the example API and tests while keeping the quality gates green.
   Configure registry registration and releases when the package is ready.

Manifests are ignored because this is a reusable library template: its tests
resolve dependencies within the declared compatibility bounds. If you turn it
into an application that needs a fixed environment, commit the relevant
`Manifest.toml` and update `.gitignore` accordingly.

## License

Released under the [MIT License](LICENSE).
