# Manual verification

The repository currently has no automated test suite. Build the application as
described in the [README](../README.md#build-and-run), then use the checks below.
Start a fresh process for each operation: the menu does not loop.

## Message reference cases

Select `h`, then `m`, enter the indicated message, and choose `n` when asked to
save. For the empty message, press Enter without typing any text.

| Message | Expected SHA-256 |
| --- | --- |
| Empty message | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `abc` | `ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad` |

Message mode excludes the Enter used to submit the message. Also compare a
message containing spaces and uppercase letters with an independent digest to
check that the input bytes are preserved.

## File hashing and saving

In PowerShell, create an exact three-byte ASCII fixture without a newline:

```powershell
$samplePath = Join-Path $env:TEMP 'sha256-hasher-abc.txt'
[System.IO.File]::WriteAllBytes($samplePath, [System.Text.Encoding]::ASCII.GetBytes('abc'))
(Get-FileHash -LiteralPath $samplePath -Algorithm SHA256).Hash.ToLowerInvariant()
```

Run the application, select `h`, then `f`, and paste the value of `$samplePath`
without quotes. Expect the `abc` digest above. Repeat with an empty file and a
binary file containing zero bytes, comparing each result with `Get-FileHash`.
Include a path containing spaces.

Choose `y` to save a digest to a disposable destination. Verify that the file
contains exactly 64 lowercase hexadecimal characters, without a newline. Repeat
with an existing disposable destination to verify that its contents are replaced.
Saving to a nonexistent parent directory should report an opening error and
return `1`.

## Comparison and input handling

| Check | Expected behavior |
| --- | --- |
| `c`, the `abc` digest, `m`, `abc` | Prints that the hash codes are the same; exit `0` |
| Same comparison with uppercase digest letters | Still matches |
| The `abc` digest compared with message `abcd` | Prints that the hash codes are not the same; exit `0` |
| Digest with 63 or 65 characters, or a nonhex character | Reports incorrect input; exit `1` |
| `h`, `f`, nonexistent file path | Reports a file-opening error; exit `1` |
| Unrecognized menu option | Reports an unrecognized option; exit `1` |
| `x` | Exits with `0` |

In PowerShell, inspect `$LASTEXITCODE` immediately after the program finishes.
Do not use comparison's process exit code to decide whether the digests match.

## Bounded search

Generate a one-byte message digest independently:

```powershell
$sha = [System.Security.Cryptography.SHA256]::Create()
try {
    $aDigest = [System.BitConverter]::ToString(
        $sha.ComputeHash([System.Text.Encoding]::ASCII.GetBytes('a'))
    ).Replace('-', '').ToLowerInvariant()
    $aDigest
} finally {
    $sha.Dispose()
}
```

Select `r`, paste `$aDigest`, and use maximum length `1`. Expect
`Dehashed message: a`; use lowercase `n` at the save prompt. Repeat with lowercase
`y` and verify that the saved file contains the single byte for `a`.

Repeat with maximum length `0`: expect no match and exit `0`. The empty-message
digest also has no match with maximum length `1`, because search excludes the
empty message. An uppercase digest containing letters should be rejected by
search mode. Keep search limits small to avoid exponential runtimes.

## Padding boundaries and known failure

For hashing changes, check message and file lengths 0, 1, 54, 55, 56, 63, 64, 65,
119, and 120 bytes against an independent implementation. For example, create a
55-byte file in PowerShell:

```powershell
$boundaryPath = Join-Path $env:TEMP 'sha256-hasher-55.txt'
[System.IO.File]::WriteAllBytes($boundaryPath, [System.Text.Encoding]::ASCII.GetBytes(('a' * 55)))
(Get-FileHash -LiteralPath $boundaryPath -Algorithm SHA256).Hash.ToLowerInvariant()
```

Hash this file in the application and compare the outputs. **The current code is
expected to disagree at lengths 55 and 119.** Record these as known failures,
rather than accepting the application's output as a reference. See the
[padding explanation](IMPLEMENTATION.md#known-limitations). A future padding
fix should make both message and file cases agree at every listed length.

Use complete, newline-terminated input when scripting console interactions. The
current reader does not terminate correctly on unexpected EOF.
