# Implementation notes

## Source layout

All C++ source files are under `SHA-256 Encryptor/`.

| File | Responsibility |
| --- | --- |
| `Main.cpp` | Entry point, menu dispatch, hash/compare/search workflows, exit codes |
| `UserInterface.h/.cpp` | Prompts, message/file selection, digest display preparation, saving results |
| `Hash.h/.cpp` | Digest structure, SHA-256 compression, message padding, formatting, exhaustive search |
| `FileHelpers.h/.cpp` | File size, binary loading, padded buffer size calculation |
| `StringHelpers.h/.cpp` | Input lines, string operations, hex formatting and validation, ASCII case conversion |
| `Endianness.h/.cpp` | Host byte-order detection and conversion to big-endian values |
| `Memory.h/.cpp` | Byte-by-byte memory copy |
| `k` | Text containing the empty-message digest; not a build input or runtime dependency |

`SHA-256 Encryptor.sln` defines Debug/Release configurations for x64 and x86.
The `.vcxproj` selects MSVC v143 and Windows SDK `10.0`; its `.filters` companion
organizes files in Visual Studio.

## Hashing flow

1. `HashString` calls `PadString`; `HashFile` calls `LoadFileInMemory`.
2. The input is copied to a zero-initialized buffer. A `0x80` byte follows the
   input, and the input's bit length is written as an eight-byte big-endian value
   at the end of the buffer. `AlignBuffSize` computes the buffer size.
3. Each 64-byte chunk is expanded from 16 words into a 64-word message schedule.
4. Sixty-four compression rounds update eight working words using the constants
   in `Hash.cpp`. The chunk result is added to the accumulated state.
5. The final state is stored in `Hash::h0` through `Hash::h7`. `HashToStr` formats
   each word as eight lowercase hexadecimal digits.

The message and file compression loops are implemented separately. A change to
the compression algorithm must be applied consistently to both paths.

## Internal interfaces and ownership

These are internal C++ helpers, rather than a separately packaged library.

| Function | Result and caller responsibilities |
| --- | --- |
| `HashString(const char*, Hash*)` | Hashes a null-terminated message into a valid output pointer |
| `HashFile(const char*, Hash*)` | Hashes the named file; returns `false` on reported loading failure |
| `HashToStr(char*, size_t, const Hash*)` | Writes a null-terminated hex digest; provide at least 65 bytes on the target platform |
| `PadString(const char*, int&)` | Returns a padded `Byte[]` buffer and its byte count; caller uses `delete[]` |
| `LoadFileInMemory(const char*, int&)` | Returns a padded file buffer and its byte count; caller uses `delete[]` |
| `GetString(char)` | Reads up to a delimiter into an allocated `char[]`; caller uses `delete[]` |
| `GetStringFromHash(const char*, unsigned int)` | Returns an allocated matching `char[]`, or `nullptr`; caller owns a successful result |

Hashing functions expect valid input and output pointers; their boolean results
do not cover every possible allocation or I/O failure. The implementation assumes
8-bit bytes, 32-bit `unsigned int`, and 64-bit `unsigned long long`, as on the
configured Windows targets.

## Exhaustive search

`GetStringFromHash` iterates candidate lengths up to the supplied maximum.
`TryAllMessagesOfLenN` recursively fills a null-terminated buffer using byte
values 255 down to 1 and hashes each candidate. Length zero returns no candidate.
Candidates use arbitrary nonzero bytes, rather than an alphabet restricted to
printable characters.

The worst-case candidate count for maximum length `L` is
`255 + 255^2 + ... + 255^L`. There is no progress indicator or internal time limit.
The first match is returned; the search cannot establish that it was the original
message.

## Known limitations

- **Incorrect padding at lengths congruent to 55 modulo 64.** `AlignBuffSize`
  first adds nine bytes, then always advances to the next multiple of 64, even
  when already aligned. A 55-byte input consequently gets a 128-byte buffer
  instead of 64 bytes. This affects both string and file hashing and produces
  incorrect digests for these lengths.
- **Whole-input buffering and signed sizes.** Files are loaded entirely into
  memory. File lengths, string lengths, and padded buffer sizes use `int`;
  large inputs can overflow size calculations. There is no streaming API or
  enforced safe maximum file size.
- **Limited error reporting.** Allocation exceptions are not handled, and file
  reads and output writes are not checked for completion. A file changing during
  hashing, a short read, or a failed write may not be reported correctly.
- **Text and EOF handling.** Messages are null-terminated byte strings, so embedded
  zero bytes terminate them. Console encoding determines non-ASCII input bytes.
  `GetString` waits for its delimiter without checking EOF; truncated redirected
  input can cause continued allocation instead of a clean exit.
- **Inconsistent case rules.** Comparison mode lowercases the supplied digest;
  search mode requires it already lowercase. The recovered-message save prompt
  reads `y`/`n` without case conversion.
- **Search resource use.** Runtime grows exponentially, and recursion depth grows
  with candidate length. The console search workflow does not release the
  returned message buffer before exiting.

Use this project to study the algorithm and console programming. Its current
correctness and error-handling limits make it unsuitable as a production
cryptographic component.
