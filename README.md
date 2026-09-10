# Legion Miner Releases

Prebuilt binaries for Legion Miner, a multi algorithm miner with self-organizing
Legion farm coordination.

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

Every platform also ships as a `.tar.gz` carrying the binary under a plain name,
for mining os integrations that install from an archive.

Verify a download:

    sha256sum -c checksums.sha256

## Usage

    legion-miner -p pool.example.com:4444 -u <wallet> --clean

The Legion (self-organizing farm coordination) is on by default; `--standalone`
mines solo. Miners only pair with others mining the same algorithm at the same
pool or daemon, so farms for different coins stay separate on one network.

Run `legion-miner --help` for the full option set.

## Algorithms

`--algo` selects one, defaulting to rx/0.

| algo | used by |
|---|---|
| `rx/0` | Salvium, Monero, and other stock RandomX chains |
| `rx/sfx` | Safex |
| `rx/wow` | Wownero |
| `sha256d` | Veil's asic lane |
| `progpow` | Veil's gpu lane, on the cpu |
| `progpow-gpu` | Veil's gpu lane, on a gpu through Vulkan |

Solo daemon mining supports Salvium and Veil. Veil runs three proof of work lanes
in parallel, each with its own difficulty, and `--algo` picks which one to mine:

    legion-miner --coin veil --algo sha256d --daemon http://127.0.0.1:8332 \
      --rpc-user <user> --rpc-pass <pass>

## Gpu

`--algo progpow-gpu` mines Veil's ProgPoW lane on a gpu through Vulkan, which is
already present on a stock desktop, so nothing has to be installed. `--list-gpus`
reports every device and whether it could actually mine.

That lane needs 4.71 GiB of ram for its dag and the same again on disk to cache
it. The dag is built once per epoch, about every 5.7 days, and reloaded in
seconds after that.

## Upgrading

Keep a desktop build current in place, no reinstall:

    legion-miner --check-upgrade   # report if a newer build exists
    legion-miner --upgrade         # download, verify, and swap it in

The check compares this binary's checksum against the latest release, so a
rebuilt release of the same version is still detected.

Legion Miner is closed source. The binaries are free to use.
