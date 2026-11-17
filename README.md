# autotests: UCU lab autotests

One tool (`labtest`) for all labs. It runs on Linux, Windows and macOS; the same tests run locally, on a server and in GitHub Actions.

This repository holds what students need: the binaries ([Releases](../../releases)), these instructions and the test data
that `labtest` downloads on first use. The tester itself lives in the private `autotests-core`.

## For students

Download `labtest-linux` / `labtest-windows.exe` / `labtest-macos` from the [releases](../../releases) and run it from your project directory:

```
labtest list                              # all labs
labtest <course> <lab> [variant] [options]
```

| Command | What is checked |
|---|---|
| `labtest poc string1` | lab 1: class `my_str_t` (`mystring.h(pp)` + `mystring.cpp`) |
| `labtest poc string2 [bash\|make\|cmake]` | lab 2: `compile.sh`, Makefiles, CMake for `sample/` and `mystring/` |
| `labtest poc string3 <1..5> [--exe name]` | lab 3: `prog <in> <out>`, your variant |
| `labtest poc asm <1..15> -p func_1` | lab 4: the assembly function of your variant |
| `labtest poc disasm <1..73> --orig prg_NN.x --patched your_file` | lab 5: patched binary |
| `labtest acs perf` | lab 1: `cxx_perf_io`, `cxx_perf_conv` |
| `labtest acs integral-serial` / `integral-parallel` / `integral-queue` / `integral-tpool` | labs 2, 3, 4, 5: integration |
| `labtest acs 1brc` | lab 6: One Billion Row Challenge |
| `labtest acs words-seq` / `words-par` / `words-proftools` / `words-tbb` | labs 7, 8, 9, 10: word counting |
| `labtest acs mpi [--exe name] [--args "config.cfg"]` | lab 11: MPI (Linux only, Ubuntu 26.04; uses Docker without `mpirun`) |
| `labtest cpp functions \| ratio \| optional \| vector \| array \| smartptr \| algorithms` | "Modern C++" labs |

`-p <dir>` is the project directory (default: current), `-v` prints more detail, `--json file` writes the result for CI.

### Additional tasks: `--ia`

Additional tasks are not checked without the flag.

```
labtest poc asm 5 -p func_1 --ia C,F      # asm, additional tasks C and F
labtest poc string1 --ia move,ops         # move semantics, + += * *=
labtest acs integral-parallel --ia f4     # function 4
labtest ... --ia                          # no list = all additional tasks of the lab
```
`labtest list` shows the available names. For asm, additional task `X` is looked up in `<repo>/extra_X/src/func.s` (also `func_X`, `func_extra`).

### Exit codes
`0`: everything passed, `1`: some check failed, `2`: environment problem (missing compiler, file not found, ...).
Your files are never modified: builds happen in a temporary directory.

### Test data
Some labs (`acs 1brc`, `acs words-*`, `poc disasm`) need big sample files. They are downloaded from this repository
(`<course>/<lab>/testdata/`) on first use into `~/.cache/labtest` (`%LOCALAPPDATA%\labtest\cache` on Windows) and checked
by SHA-256. `curl` is required. If the download fails, the checks that need the data are skipped, never failed.
Settings: `LABTEST_CACHE` (cache folder), `LABTEST_DATA_URL` (another mirror), `LABTEST_DATA_DIR` (a local clone of this repository, offline use).
Do not edit the files by hand: their checksums are compiled into `labtest`.

### GitHub Actions
In your lab repository:

```yaml
jobs:
  test:
    uses: ucu-poc-acs-os-cpp/autotests/.github/workflows/lab.yml@main
    with: { lab: "acs integral-parallel", ia: "f4", variant: "" }
```
The workflow downloads the latest `labtest-linux`, runs it in the repository and stores `result.json` as an artifact.

### Interface contract of the `cpp` labs

The course text leaves most interfaces open, so the tests fix these assumptions (std-like names):

| Lab | Files | Contract |
|---|---|---|
| `functions` | any header declaring `my_printf` / `my_print` (+ library `.cpp`) | `my_printf(const char*, ...)` with `%% %d %f %s`; `my_print(fmt, args...)` with `{}` placeholder (change with `--args "<placeholder>"`) |
| `ratio` | `ratio_t.h(pp)` / `ratio.h(pp)` | `ratio_t(n)`, `ratio_t(n, d)`, `+ - * /`, compound forms, unary `+ -`, comparisons, `<<` / `>>`; at least three of `abs pow floor ceil trunc round` |
| `optional` | `my_optional.h(pp)` | as in the task text; `my_bad_optional_access`; values created with `std::in_place` / `emplace` |
| `vector` | `my_vector.h(pp)` | as in the task text, including `is_empty()`, `rbegin/rend`, strong exception safety |
| `array` | `my_array.h(pp)` | same as vector where applicable; no dynamic allocation; `sizeof == N * sizeof(T)` |
| `smartptr` | `my_unique_ptr`, `my_shared_ptr`, `my_weak_ptr` headers | `get`, `*`, `->`, `release`, `reset`, `swap`, `use_count`, `lock`, `expired`, custom deleters |
| `algorithms` | any header declaring `my_copy` / `my_transform` | std signatures (`first, last, d_first[, op]`); standard algorithms must not be used |
