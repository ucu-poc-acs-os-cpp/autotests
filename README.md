# labtest-data

Test data for the `labtest` autotests. The binary downloads these files on first use (raw URLs, SHA-256 checked) and
caches them. The layout mirrors `labs/<course>/<lab>/testdata/` of the private `autotests` repository.

- `acs/1brc/testdata`: official 1BRC samples with the expected outputs
- `acs/words-*/testdata`: a subset of the Project Gutenberg collection and the expected word counts
- `poc/disasm/testdata`: original binaries of the disassembly lab

Do not edit by hand: the files are generated and their checksums are compiled into `labtest`.
