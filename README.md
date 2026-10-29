# SHA-256 Hasher

A C++ console application that implements SHA-256 hashing for messages and files,
compares an input against an existing digest, and demonstrates brute-force message
search. Written by Ivan Venkov for the Introduction to Programming course at the
Faculty of Mathematics and Informatics, Sofia University, winter semester 2023/2024.

The Visual Studio solution and executable retain the original name
`SHA-256 Encryptor`. SHA-256 produces a hash, rather than encrypting a message;
the search feature tries candidate messages and compares their hashes.

## Features

- Hash a single line of text or the raw bytes of a file.
- Display a digest as 64 lowercase hexadecimal characters.
- Compare a message or file with a supplied digest.
- Search for a message matching a digest, up to a specified byte length.
- Optionally save a generated digest or a recovered message to a file.

**Known correctness limitation:** inputs whose byte length is 55 modulo 64
(55, 119, 183, ...) receive incorrect padding and produce a nonstandard digest.
This is an educational implementation; check results against a trusted tool
before relying on them. See [implementation notes](docs/IMPLEMENTATION.md) for
details and other limits.

## Build and run

### Requirements

- Windows with Visual Studio 2022 or its Build Tools.
- The **Desktop development with C++** workload, including the MSVC v143 toolset
  and a Windows SDK compatible with the project's `10.0` SDK setting.

The project uses the C++ standard library and has no third-party dependencies.
The checked-in build configuration is a Visual Studio solution; no CMake or
Makefile build is provided.

### Visual Studio

1. Open `SHA-256 Encryptor.sln`.
2. Select **Release** and **x64** in the configuration toolbar.
3. Choose **Build > Build Solution**.
4. Run with **Debug > Start Without Debugging** (`Ctrl+F5`).

### Command line

In **Developer PowerShell for VS 2022**, from the repository root:

```powershell
msbuild '.\SHA-256 Encryptor.sln' /m /p:Configuration=Release /p:Platform=x64
& '.\x64\Release\SHA-256 Encryptor.exe'
```

Use `/p:Configuration=Debug` for a debug build, then run the executable under
`x64\Debug`. The solution also supports `/p:Platform=x86`, which maps to the
project's `Win32` platform.

If MSBuild is unavailable, open the Visual Studio developer shell. If the v143
toolset or Windows SDK is missing, install those components through Visual Studio
Installer.

## Usage

The program prompts for input interactively and handles one operation per run.
Enter each option on its own line, then press Enter. There are no command-line
flags.

| Option | Action | Next inputs |
| --- | --- | --- |
| `h` | Hash a message or file | `m` for a message or `f` for a file, then the content or path |
| `c` | Compare with a digest | Existing digest, then `m` or `f` and the content or path |
| `r` | Search for a matching message | Lowercase digest, then maximum search length |
| `x` | Exit | None |

### Hash a message

Enter `h`, `m`, `abc`, and `n` at the successive prompts:

```text
Enter option: h
Enter 'm' or 'f' to get the hashcode of a message or a file.
('m' or 'f'): m
Enter message: abc
Hash code: ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
Do you want to save it in a file? (y/n): n
```

The terminating Enter is excluded from a message's hash. An empty line hashes
the empty message. Spaces and letter case within the message are preserved.

### Hash a file or save a result

Choose `h`, then `f`, and enter an existing file path. Paths may contain spaces;
enter them **without surrounding quotes** at the program's prompts. Relative
paths are resolved from the process's working directory. Files are read in
binary mode, so a file's newline bytes and encoding affect its digest.

After hashing, choose `y` and enter a destination path to save the digest.
The program creates or **overwrites** that file and writes the digest without
a trailing newline. The parent directory must already exist.

### Compare a digest

Choose `c`, paste a 64-character hexadecimal digest, then choose a message or
file source. The program prints whether the digests match. The comparison accepts
uppercase and lowercase hex digits. A mismatch is a completed comparison and
still returns exit code `0`.

### Search for a message

Choose `r`, enter a **lowercase** 64-character hexadecimal digest, and enter a
small nonnegative maximum length. Start with `1` for a demonstration.

The search tries byte values 1 through 255 for lengths 1 through the maximum.
It cannot find the empty message or messages containing a zero byte. Exhausting
length `n` takes `255^n` candidate hashes; larger limits quickly become
impractical. Use `Ctrl+C` to interrupt a long search. A matching message can
contain bytes that do not display as readable text.

The UI calls this operation "dehashing". It is an exhaustive search, not an
inverse of SHA-256, and success is not guaranteed. Unlike comparison mode,
search mode rejects uppercase hex. At its save prompt, use lowercase `y` or `n`.

### Exit codes and input errors

- `0`: operation completed, including a comparison mismatch, an exhausted search,
  or choosing `x`.
- `1`: invalid input or a reported file/hashing error.

The menu options and `m`/`f` choices accept either letter case. Hashes must contain
exactly 64 hexadecimal digits with no surrounding whitespace. Text input is a
single line; use file mode for multiline or binary input. For redirected input,
supply every requested line and end each with a newline: the line reader does
not handle premature end-of-file correctly.

## Further documentation

- [Implementation and known limitations](docs/IMPLEMENTATION.md)
- [Manual verification and reference digests](docs/TESTING.md)
- [Contributing](CONTRIBUTING.md)

## License

No license file is currently included in this repository.
