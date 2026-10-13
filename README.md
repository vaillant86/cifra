# Cifra

A command-line tool to encrypt and decrypt files and directories with **AES-256-GCM**, using a key derived with **scrypt** from a password or a keyfile.

Single Python file, one dependency, works on any system with Python 3.

## Features

- **Strong authenticated encryption**: AES-256-GCM, so any wrong password or modified file is detected on decryption. The entire file header is also protected via Additional Authenticated Data (AAD).
- **Memory-hard key derivation**: scrypt (`n=2^17`, `r=8`, `p=1`, about 128 MiB of RAM) with a random 16-byte salt for every file.
- **Files and directories**: a directory is archived with `tar` on the fly and encrypted in a single stream.
- **Streaming & Pipes**: data is processed in 64 KB chunks. Use `-` as the target to encrypt/decrypt data streams directly via Unix pipes without touching the disk.
- **File exclusion**: ignore specific files or directories during archiving using standard wildcard patterns (`-e`).
- **Password or keyfile**: type a password (asked twice when encrypting) or use any file as a key with `-k`.
- **Safe writes**: output goes to a temporary `.tmp` file and replaces the destination only after the operation succeeds. On errors or `Ctrl-C` the temporary file is removed.
- **Overwrite protection**: asks before replacing an existing file, unless `-f` is given.
- **Hardened decryption**: scrypt parameters read from the file are validated (`n` a power of two between 2^14 and 2^20, `r` 1-16, `p` 1-4) before any memory is allocated, so a crafted file cannot exhaust your RAM.
- **Performance optimized**: smart I/O throttling on the progress bar maximizes NVMe disk speeds without terminal lag.

## Requirements

- Python 3.8 or newer
- The [`cryptography`](https://pypi.org/project/cryptography/) package

On Debian, Ubuntu or Linux Mint:

```bash
sudo apt install python3-cryptography
```

On Fedora, RHEL, or CentOS:

```bash
sudo dnf install python3-cryptography
```

On Arch Linux or Manjaro:

```bash
sudo pacman -S python-cryptography
```

## Installation

```bash
chmod +x cifra
sudo mv cifra /usr/local/bin/
```

## Usage

```
cifra [-h] [-v] [-f] [-o OUTPUT] [-k KEYFILE] [-e PATTERN] {encrypt,decrypt} target
```

| Argument | Description |
|----------|-------------|
| `encrypt` | Encrypt `target` and create `<target>.enc` |
| `decrypt` | Decrypt `target` (a `.enc` file created by this tool) |
| `target` | File or directory to process, or `-` for standard input/output |
| `-o`, `--output` | Specify an explicit output file path |
| `-f`, `--force` | Overwrite existing output files without asking |
| `-k`, `--keyfile` | Use the contents of a file as the key instead of a password |
| `-e`, `--exclude` | Exclude files/directories matching this pattern (e.g. `*.log`) |
| `-v`, `--version` | Show program's version number and exit |

Notes about keyfiles:

- Max size of a readable keyfile is set to 4 MB and an empty keyfile is rejected.
- Use a binary file, not a text file: if an editor changes even one byte (for example by adding a trailing newline), the key changes and you will not be able to decrypt.
- Keep a backup copy. Without the keyfile the data cannot be recovered.

## Examples

### Basic encryption and decryption

```console
$ cifra encrypt foo.bar

$ cifra encrypt --exclude "*.log" /var/www/html

$ cifra decrypt foo.bar.enc
```

### Use a keyfile instead of a password

```bash
cifra encrypt -k my.key foo.bar
cifra decrypt -k my.key foo.bar.enc
```

### Run without installing

```bash
python3 cifra encrypt foo.bar
python3 cifra decrypt foo.bar.enc
```

## Limitations and security notes

- This is a personal tool and has **not** been independently audited. For critical data, prefer a well-established tool.
- The original file or directory is **not deleted** after encryption. Remove it yourself if you need to. Note that secure deletion is not reliable on SSDs and copy-on-write filesystems; full-disk encryption is the better protection there.
- A temporary `.tmp` file is written next to the output while the operation runs (unless streaming via pipes).
- Losing the password or the keyfile means losing the data: there is no recovery.
- AES-GCM with a single nonce is limited to about 64 GiB per file.
- With `-k` the keyfile alone is the secret. Anyone who holds that file can decrypt your data.
- Directories are restored as `.tar` archives rather than extracted automatically. As with any tar file, only extract archives you trust.

## License

This project is licensed under the GNU AGPLv3 License - see the [LICENSE](LICENSE) file for details.