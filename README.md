# Legion Miner Releases

Prebuilt binaries for Legion Miner, a multi algorithm miner with self-organizing
Legion farm coordination.

Grab a build from the [Releases page](https://gitlab.com/whiskyrelaxing-group/legion-miner-releases/-/releases).
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

    legion-miner --pool pool.example.com:4444 --user <wallet> --clean

The Legion is on by default; `--standalone` mines alone. Boxes only pair with
others mining the same algorithm at the same pool or node, so Legions for
different coins stay separate on one network. Within a Legion, boxes form
companies: a captain takes its node's work to the members that follow it over
the same gossip and submits their blocks, credited to the finder. Ranks follow
the work, a sergeant over its own sticks and cpu, a lieutenant over the network
miners on its pool, a captain over a company, and the boxes settle them among
themselves.

Every flag takes two dashes, there are no short forms. Run
`legion-miner --help` for the full option set.

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

## Coins

`--coin` selects the chain for solo mining against a node; `--list-coins`
prints this table from the binary.

| chain | --coin | lanes | notes |
|---|---|---|---|
| Salvium | `salvium` | rx/0 | the default |
| Monero | `monero` | rx/0 | also the Tari merge mining proxy, pointed at by `--daemon` |
| Wownero | `wownero` | rx/wow | |
| Safex | `safex` | rx/sfx | |
| Veil | `veil` | rx/0, sha256d, progpow | one lane per run, named with `--algo`; the node needs `-miningaddress` |
| Bitcoin | `bitcoin` | sha256d | needs `--payout-address` and an asic to matter |
| Litecoin | `litecoin` | scrypt | needs `--payout-address` |
| Dogecoin | `dogecoin` | scrypt | needs `--payout-address` |

Veil runs three proof of work lanes in parallel, each with its own difficulty,
and `--algo` picks which one to mine:

    legion-miner --coin veil --algo sha256d --daemon http://127.0.0.1:58812 \
      --rpc-user <user> --rpc-pass <pass>

A Bitcoin derived chain hands out the ingredients of a block rather than a
header, so the miner builds the block and `--payout-address` names who it pays.
The node itself checks the address, so a typo is a startup error. `--zmq` on
the node's zmq feed brings a new block to the miners the moment the node has it.

## Usb asics

GekkoScience Compac F sticks mine Bitcoin and Veil's sha256d lane. `--asic auto`
is the default: a box with sticks parks its cpu and splits the work by what
each stick measurably does. The chip's reset line is root only, so a stick run
starts under `sudo` and gives root back as soon as the sticks are open; no udev
rules or group changes. `--list-sticks` shows what a machine has.

## A pool for your network miners

`--stratum-port PORT` makes a box the solo pool for the network miners on your
lan, against your own node, sharing its Legion slice. It serves the stratum
Bitaxes, Antminers, Goldshells, and LuckyMiners speak for Bitcoin, Litecoin,
and Dogecoin; xmrig's protocol for Salvium, Monero, Wownero, Safex, and Veil's
randomx lane; T-Rex's dialect for Veil's ProgPoW lane; and for Veil's sha256d
lane the plain Bitcoin stratum, so an AxeOS unit, a Bitaxe, a Lucky Miner, or a
NerdQaxe, mines Veil while the miner builds each Veil block itself, with the
SHA256dv dialect of cpuminer-opt-veil on the port after. Every miner gets a
coinbase of its own, version rolling where the chain allows it, and a share
difficulty that tunes itself to each miner. Veil's blocks pay the node wallet's
mining address, made there if it has none, or `--wallet`.

Antminers cannot mine Veil this way, since cgminer refuses the empty extranonce
Veil requires. Veil leaves only eleven version bits to roll where Bitcoin leaves
sixteen, so an AxeOS unit on an older firmware runs a Veil job again before its
firmware replaces it and about two thirds of its hashing repeats; the miner
credits what is new and logs the rest.

    legion-miner --coin veil --algo sha256d --daemon http://127.0.0.1:58812 \
      --rpc-user <user> --rpc-pass <pass> --stratum-port 3330

    legion-miner --coin monero --daemon http://127.0.0.1:18081 --wallet <address> \
      --stratum-port 3333

## Dashboard

`--stats-html PATH` writes a self contained page a browser can open, readable
on a phone: every box in the Legion, a tab for each company, each box's share
of the search, trends, block finds, and the network miners under the box they
mine through.

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
