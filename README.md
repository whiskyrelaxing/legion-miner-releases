# Legion Miner Releases

Prebuilt binaries for Legion Miner, a multi algorithm miner with self-organizing
Legion farm coordination.

Grab a build from the releases page on [GitLab](https://gitlab.com/whiskyrelaxing-group/legion-miner-releases/-/releases) or [GitHub](https://github.com/whiskyrelaxing/legion-miner-releases/releases); they carry the same files.
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
themselves. `--captain` makes a box lead whenever it is up, taking the company
over from a captain that did not ask to lead, and `--no-captain` keeps a box
out of the running, say one that hosts other work. Every block a box finds, or
submits for a member, gets a banner in its log.

Every flag takes two dashes, there are no short forms. Run
`legion-miner --help` for the full option set. `--clean` condenses the console
to a stats line and events.

The default thread count is computed for the machine AND the algorithm. RandomX
wants each thread's 2 MB scratchpad resident, so it gets (total L2 + L3) / 2 MB
clamped to the logical core count. sha256d's working set is an 80 byte header
and a 32 byte midstate, so it gets every logical core: measured on a 7840U that
is 16 threads rather than 12, worth 19%. `--threads` overrides either.

`--list-ifaces` shows every network interface a Legion heartbeat could leave
by, and why the others cannot:

    INTERFACE          ADDRESS          BROADCAST        WHY
    lo                 127.0.0.1        -                loopback
    tailscale0         100.66.188.88    -                no broadcast address
    wlp1s0             192.168.9.133    192.168.9.255    usable

By default a heartbeat goes out of every usable one, which finds peers wherever
they are, including across a bridge. A box with docker or vm interfaces can
narrow that with `--legion-iface <name>`, repeated for more than one. A name
that matches nothing is refused at startup rather than leaving the miner
broadcasting to nowhere. `--legion-peer host:48711`, one per box, reaches
boxes a broadcast cannot, across subnets or a vpn.

Background mining (`--background`) drops every mining thread to the lowest
scheduling priority, nice 19 on Linux and macOS, idle priority class on Windows,
so the miner yields the moment anything else wants the cpu. That is the whole
feature: the scheduler does this better than a polling loop can, so there is no
idle detection, no cpu cap, and no auto pause.

`--hash-feed` emits the `[hash]` visualizer feed, a few sampled hash prefixes a
second for a front end that draws them. Off by default, and paced to a total
rate rather than one per worker, so it reads the same on eight threads as on
sixty four.

## Algorithms

`--algo` selects one, defaulting to rx/0.

| algo | used by |
|---|---|
| `rx/0` | Salvium, Monero, and other stock RandomX chains |
| `rx/sfx` | Safex |
| `rx/wow` | Wownero |
| `sha256d` | Veil's asic lane |
| `progpow` | Veil's gpu lane: a gpu when there is a usable one, cpu if not |
| `progpow-gpu` | the same lane, refusing to run without a gpu |
| `progpow-cpu` | the same lane, on the cpu deliberately |

Every algorithm is validated against real blocks from its own chain rather than
against published vectors: the tests reconstruct a mined block's hashing input
and reproduce the proof of work the network accepted. Nothing ships without that.

RandomX comes from tevador/RandomX, vendored as a submodule pinned to v1.2.1.
The three RandomX variants are separate builds of that source with their own
parameters, isolated at link time so they can live in one binary. ProgPoW and
ethash come from the vendored implementation Veil's node validates with.

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

GekkoScience Compac F and Compac A2 sticks mine Bitcoin and Veil's sha256d
lane. `--asic auto` is the default: a box with sticks parks its cpu and splits
the work by what each stick measurably does. The Compac F's reset line and the
A2's usb interface are root only, so a stick run starts under `sudo` and gives
root back as soon as the sticks are open; no udev rules or group changes.
`--list-sticks` shows what a machine has.

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
on a phone, that refreshes in place; any static web server serves it. It is a
menu of views: Dashboard with the Legion's cards and the map of the search,
Miners with each box's rate, trend, share of the space, and blocks, and the
network miners under the box they mine through, Hosts with each box's
temperature, clock, power, hashes per joule, memory, and uptime, Scoreboard
with the block finds and the Legion's ten best shares, each against the
network's bar when it was found, and History with the Legion's rate over the
last day and week. One company is the page itself; a bar of companies across
the top appears only when there are several. Block finds link to the coin's
explorer where one is known, and `--explorer-url` takes a template with
`{height}` for a chain that is not listed.

Each miner keeps its own block and share totals in a
`<binary>-stats-<cluster>.dat` beside the binary, with the trends, the week,
and the scoreboard beside it, so a restart or an upgrade loses none of them.
Every miner advertises only its OWN totals over gossip, so peers restore each
other but nobody can vouch for a box's own history.

A Windows box reports its memory and clock on the Hosts view, a Mac its memory.
Their temperature and power, and a Mac's clock, sit behind a kernel driver or a
root only tool those systems do not ship, so the page leaves them blank.

## Gpu

`--algo progpow` mines Veil's ProgPoW lane on a gpu through Vulkan, which is
already present on a stock desktop, so nothing has to be installed. The cpu is a
fallback and hundreds of times slower; `--algo progpow-gpu` refuses to start
without a gpu rather than quietly crawling.

`--list-gpus` reports every device and whether it could actually mine, which
answers the real question rather than just listing hardware. `--gpu` picks one
by that number, several as a comma separated list, or `all`:

    legion-miner --coin veil --algo progpow --gpu all \
      --daemon http://127.0.0.1:58812 --rpc-user <user> --rpc-pass <pass>

Each gpu gets its own worker and its own slice of the nonce space, so two cards
divide the work between them.

Two discrete cards add up. An integrated gpu alongside a discrete one usually
does not: its dag lives in system memory, and the bandwidth it pulls from there
costs the discrete card about as much as the integrated one contributes.

The dag is 4.71 GiB and each gpu needs that much of its own memory. It is
generated on the device from a small light cache, about a minute the first time
a machine sees an epoch, then cached to disk and reloaded in seconds. An epoch
lasts 8175 blocks, roughly 5.7 days.

The shader is generated for the device it will run on, so a gpu whose buffers or
subgroups are shaped unusually gets a shader that suits it. Before mining, one
nonce is hashed on the gpu and on the cpu and the two have to agree: a gpu that
disagrees drops to the portable path, or refuses to mine, rather than hashing at
full speed and finding nothing.

## Upgrading

Keep a desktop build current in place, no reinstall:

    legion-miner --check-upgrade   # report if a newer build exists
    legion-miner --upgrade         # download, verify, and swap it in

The check compares this binary's checksum against the latest release, so a
rebuilt release of the same version is still detected.

Legion Miner is closed source. The binaries are free to use.
