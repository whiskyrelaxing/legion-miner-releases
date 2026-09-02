# Legion Miner Releases

Prebuilt binaries for Legion Miner, a native RandomX cpu miner for Salvium with
self-organizing Legion farm coordination.

Grab a build from the [Releases page](https://github.com/whiskyrelaxing/legion-miner-releases/releases).
Each release carries:

| file | platform |
|---|---|
| `legion-miner-linux-x86_64` | Linux x86_64 (glibc) |
| `legion-miner-linux-x86_64-musl` | Linux x86_64 (static, any distro or container) |
| `legion-miner-linux-aarch64-musl` | arm64 Linux, static (also termux) |
| `legion-miner-android-arm64.tar.gz` | Android arm64 (native) |
| `legion-miner-windows-x86_64.exe` | Windows x86_64 |
| `legion-miner-macos-*` | macOS (arm64 and x86_64) |
| `checksums.sha256` | sha256 of every file above |

Verify a download:

    sha256sum -c checksums.sha256

## Usage

    legion-miner -p pool.example.com:4444 -u <wallet> --clean

The Legion (self-organizing farm coordination) is on by default; `--standalone`
mines solo. Run `legion-miner --help` for the full option set.

Keep a desktop build current in place, no reinstall:

    legion-miner --check-upgrade   # report if a newer build exists
    legion-miner --upgrade         # download, verify, and swap it in

Legion Miner is closed source. The binaries are free to use.
