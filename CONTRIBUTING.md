# Contributing

## Development setup

Follow the [README build instructions](README.md#build-and-run) to open the
solution in Visual Studio 2022 with MSVC v143 and a Windows SDK. There are no
external packages to restore.

Read the [implementation notes](docs/IMPLEMENTATION.md) before changing padding,
hash formatting, file handling, or search behavior. The project is an educational
implementation with custom string and memory helpers.

## Making changes

- Keep changes focused and follow the surrounding C++ style: tab indentation,
  existing function naming, and matching declarations in headers.
- Keep console interaction in `UserInterface.cpp` and operation dispatch in
  `Main.cpp`; hashing code belongs in `Hash.cpp`.
- If adding source files, update both the `.vcxproj` and its `.filters` file so
  Visual Studio builds and displays them.
- Update usage documentation when prompts, input rules, or output behavior change.
- Leave generated binaries, intermediate files, `.vs` data, and user-specific
  Visual Studio settings out of commits. The existing `.gitignore` covers these.

## Verification

Build the affected configurations and run the relevant checks in
[the testing guide](docs/TESTING.md). There is currently no automated test target
or CI workflow in the repository. For hashing changes, compare both message and
file results with an independent SHA-256 implementation, including padding
boundaries. Record any known failing cases explicitly.

Before submitting, run:

```powershell
git diff --check
git status --short
```

Describe the behavior changed, why it changed, the configurations built, and the
checks performed. For a bug report, include the exact input or file bytes,
expected and actual output, build configuration, and compiler version. Use small,
non-sensitive samples that reproduce the issue.
