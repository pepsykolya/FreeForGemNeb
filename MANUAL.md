# FreeForGemNeb (ffgn) 1.0.0 — manual for beginners

This guide takes you from zero to a running **NEB** miner: a wallet, the miner on Windows, HiveOS or Linux, and the two ways
to mine (through a pool, or solo through your own node). Русская версия — [MANUAL.ru.md](MANUAL.ru.md). HiveOS in detail:
[HIVEOS.md](HIVEOS.md).

## Contents

1. [What NEB is, in short](#1-what-neb-is-in-short)
2. [What you need](#2-what-you-need)
3. [Create a wallet](#3-create-a-wallet)
4. [The package and how to check it](#4-the-package-and-how-to-check-it)
5. [Pool or solo?](#5-pool-or-solo)
6. [Windows](#6-windows)
7. [Linux](#7-linux)
8. [HiveOS](#8-hiveos)
9. [Pools: Aria and Suprnova](#9-pools-aria-and-suprnova)
10. [Solo through your own node](#10-solo-through-your-own-node)
11. [Rigs and solo: ffgn-gate](#11-rigs-and-solo-ffgn-gate)
12. [All options](#12-all-options)
13. [Clocks, undervolt, auto-lock: speed and watts](#13-clocks-undervolt-auto-lock-speed-and-watts)
14. [Laptops, and mining only while you are away](#14-laptops-and-mining-only-while-you-are-away)
15. [Failover, watchdogs, restarts](#15-failover-watchdogs-restarts)
16. [Reading the log, the API, the balance](#16-reading-the-log-the-api-the-balance)
17. [Dev fee](#17-dev-fee)
18. [Troubleshooting and FAQ](#18-troubleshooting-and-faq)
19. [Updating, removing, contact](#19-updating-removing-contact)

## 1. What NEB is, in short

NEB (Neural Engine Byte) is a proof-of-work coin. One mining attempt is one integer training step of a small neural network
that the chain itself owns. There are two networks and the block height decides which one works: **KEP1** (a network on
NASA Kepler transit signals) and **RET2** (a network on retinal scans of the OCTMNIST set). They alternate from block to
block, so the speed of any miner jumps up and down between odd and even blocks: that is normal (section 18).

The main network starts on **Wednesday 18 November 2026, 14:00 UTC** (17:00 Moscow time). A block takes about 60 seconds, the
reward is 20 NEB per block (5 % of it goes to the NEB project, the miner gets 19 NEB plus fees), and mined coins can be spent
after 100 blocks. ffgn is a native GPU miner for NVIDIA cards: it works with a pool or with your own node. It makes no promise
about the value of the coin or about any income.

## 2. What you need

- An NVIDIA GPU, **compute capability 6.1 or newer** (GTX 10 / Pascal up to RTX 50 / Blackwell are built into the GPU engine).
  The card is tested on RTX 30 (3070) and RTX 40 (4070 Laptop); older and newer families run the same code but have not been
  measured by us. Video memory: the miner needs less than 1 GB.
- The NVIDIA driver with CUDA 12 support (R525 or newer; RTX 50 needs R570 or newer). Windows tested on 576.57. You do **not**
  need to install CUDA.
- Windows 10/11 x64, or Linux x86_64 (glibc 2.30 or newer, e.g. Ubuntu 20.04 and later), or a HiveOS rig.
- The dataset file `neb-datasets.bin` (222 MB) is **inside the package**: nothing else to download for pool mining. The miner
  checks its BLAKE3 hash at every start.
- **The clock of the computer must be right** (automatic time synchronisation on). A NEB node refuses a block dated more
  than 120 seconds ahead of its clock; a drifting clock also hurts pool mining (stale shares).
- Clock locks and undervolt need administrator rights on Windows, root on Linux and HiveOS (HiveOS runs miners as root).
  Plain mining works without them.
- A normal internet connection. Solo mining also needs your own node (section 10): about 1 GB of disk and outgoing TCP port 3440.

## 3. Create a wallet

The official wallet `neb-wallet` is part of the NEB release (section 10 says where to get it and how to check it). It keeps
one or more keys in a file, derives the `neb1...` address from them and shows your balance.

```
neb-wallet --wallet wallet.txt new
```

On Windows PowerShell: `.\neb-wallet.exe --wallet wallet.txt new`. It asks for a password (twice) and prints your **address**
(`neb1...`). To print it again: `neb-wallet --wallet wallet.txt address`.

**Read this carefully: there is no seed phrase.**

- The wallet **file and its password are the only way to get your coins back.** Copy `wallet.txt` to another disk or a USB
  stick and write the password down in a safe place. A lost file or a lost password means lost coins; nobody can restore them.
- An empty password leaves the file unencrypted. Choose a password. To encrypt an existing plain file:
  `neb-wallet --wallet wallet.txt encrypt`.
- Never send the file or the password to anybody and never paste them into a chat. Nobody legitimate asks for them.
- The wallet file of the NEB test networks has the same format and holds the carried-over coins under the same address (the
  genesis block pays each test-network address a tenth of its test balance). If you used a test network, use the same file.

For mining in a pool the miner needs only the **address**, not the file: keep the file on your own computer.

| Command | What it does |
|---|---|
| `neb-wallet --wallet wallet.txt address` | prints your first address |
| `neb-wallet --wallet wallet.txt list` | prints all addresses of the file |
| `neb-wallet --wallet wallet.txt lock` | prints the 64-hex "lock" of your address: the value the node's `--miner-lock` takes (solo) |
| `neb-wallet lock-of neb1...` | the lock of any address (so a node can pay an address without having the wallet file) |
| `neb-wallet --rpc http://127.0.0.1:8432 --wallet wallet.txt balance` | confirmed and immature balance (needs a synchronised node) |
| `neb-wallet --rpc http://127.0.0.1:8432 --wallet wallet.txt send neb1... 1.5` | send coins |

Mined coins can be spent after **100 blocks** (about 100 minutes at 60 s per block); until then the balance shows them as
immature. Pools pay by their own rules (section 9).

## 4. The package and how to check it

| Package | File | Contents |
|---|---|---|
| Windows x64 | `ffgn-1.0.0-windows-x64.zip` | `ffgn.exe`, `nebgpu.dll`, `neb-datasets.bin`, `start.bat`, `start-suprnova.bat`, `start-solo.bat`, `start.ps1`, `install-autostart.bat`, `remove-autostart.bat`, README, MANUAL, LICENSE, NOTICE |
| Linux x86_64 | `ffgn-1.0.0-linux-x86_64.tar.gz` | `ffgn`, `libnebgpu.so`, `neb-datasets.bin`, docs |
| HiveOS | `ffgn-hive-1.0.0.tar.gz` | the custom miner `ffgn-hive`: `ffgn`, `libnebgpu.so`, `neb-datasets.bin`, `h-*.sh`, docs |

Release page: https://github.com/pepsykolya/FreeForGemNeb/releases (tag `v1.0.0`). The GPU engine (`nebgpu.dll` /
`libnebgpu.so`) is closed source and ships as a binary; everything else is described in NOTICE.

**Check the download** against `SHA256SUMS` of the release. Windows: `certutil -hashfile ffgn-1.0.0-windows-x64.zip SHA256`;
Linux: `sha256sum -c SHA256SUMS`. The dataset file `neb-datasets.bin` in the package is the file of the NEB 1.0.0 main
network, unchanged:

```
size     222320684 bytes
SHA-256  500846f682669743b1305776cfbf19edc11ad3db1c809bd5a4fe5b33cf196fa4
BLAKE3   fbabd0bdde0b43aef946e7031ff6e037c8a2a5d0ae2616b49b2b49b11af8010e
```

Self-test of the miner and your GPU (no network needed): `ffgn.exe --self-test`. It checks the built-in reference code
against the NEB test vectors, runs the GPU engine's own checks and compares what the GPU finds with the reference on the real
dataset. It ends with `NEB self-test passed`.

## 5. Pool or solo?

- **Pool.** The pool hands out the work and pays you by its rules (usually in proportion to the shares you sent). You need only
  your address. This is the easiest way and the steady one: you see accepted shares within minutes. Section 9.
- **Solo through your own node.** Your node builds the block jobs and the whole block reward (19 NEB plus fees) goes to the
  address you gave the node, but only when your own GPUs find a block: a random event that can take long. Section 10.
  A pool also has a "solo" port (the pool's solo mode pays the finder only), see section 9.
- **Rigs and solo.** The work port of a node listens on `127.0.0.1` only, so a rig in another place cannot reach it. Run an
  **ffgn-gate** beside the node (section 11) and the rigs connect to the gate.

## 6. Windows

1. Download `ffgn-1.0.0-windows-x64.zip`, check it (section 4) and unpack it into a folder **without spaces and without
   non-English letters**, for example `C:\FFGN`. Needs only the NVIDIA driver, no CUDA.
2. Pick the start file:

   | File | What it does |
   |---|---|
   | `start.bat` | pool Aria (`neb.ariabrain.com:3370`), Suprnova as the backup |
   | `start-suprnova.bat` | pool Suprnova (`neb.suprnova.cc:7804`), Aria as the backup |
   | `start-solo.bat` | solo through your own NEB node on this PC (`127.0.0.1:3441`) |

3. Right-click the file, **Edit**, and set `WALLET` to your `neb1...` address (not needed for `start-solo.bat`). Save.
   The other fields at the top of the file are optional:

   | Field | Meaning |
   |---|---|
   | `WORKER` | the worker (rig) name, default is the computer name; letters, digits, `_`, `-` |
   | `POOL` | pool address; several separated by commas = failover list, the first is the primary |
   | `NODE` | solo: the work service of your node, e.g. `127.0.0.1:3441` (then `POOL` and `WALLET` are not used) |
   | `DEVICES` | which GPUs: empty = all, or `0,1,3` (numbers of `nvidia-smi`) |
   | `PASSWORD` | pool password, normally empty |
   | `LOCK_CCLOCK`, `LOCK_MCLOCK`, `MOFFSET` | core clock lock, memory clock lock, memory offset (MHz); one value or a list in `DEVICES` order |
   | `AUTO_UV`, `UV_OFFSET` | `1` = find a safe undervolt per GPU; or a fixed offset in MHz |
   | `AUTO_LOCK` | `1` = choose the core clock lock of each GPU by measurement |
   | `TEMP_LIMIT` | keep the GPU at or below this temperature (laptops) |
   | `IDLE`, `IDLE_MINUTES`, `ACTIVE`, `ALWAYS_PAUSE` | mine only while the PC is idle (section 14) |
   | `EXTRA` | any other options from `ffgn.exe --help` |
   | `LOG` | `1` = also write the output to `logs\miner-<date>.log` (the window shows it anyway) |

4. Double-click the file. A window shows the miner. **Ctrl+C** stops it and puts your GPU settings back. If the miner exits
   by itself, the script restarts it after 15 seconds. Clocks, undervolt and `AUTO_LOCK` need the file started with
   **Run as administrator**.
5. Optional: `install-autostart.bat` (run it once as administrator) starts `start.bat` at every logon (a Task Scheduler task
   named "FFGN miner", with your normal rights); `install-autostart.bat start-suprnova.bat` starts another file. Set `WALLET`
   first: at logon nobody can answer a question. For clock locks / undervolt at logon move the folder to
   `C:\Program Files\FFGN` and run `install-autostart.bat start.bat highest` as administrator (the highest rights are only
   given to a folder that only administrators can change). `remove-autostart.bat` undoes it. Ctrl+C in the window asks
   "Terminate batch job (Y/N)?": answer Y.
   Edit the `.bat` first, then install the autostart.

From a command prompt in the folder, without the scripts:

```
ffgn.exe --pool neb.ariabrain.com:3370 --wallet neb1YOURADDRESS --worker rig1
```

Windows asks for permission for the first start of a new program (SmartScreen) and antivirus programs may flag mining
software; unblock it or add the folder to the exclusions if you trust the file (compare the SHA-256 first).

## 7. Linux

```bash
mkdir -p ~/ffgn && tar -xzf ffgn-1.0.0-linux-x86_64.tar.gz -C ~/ffgn && cd ~/ffgn/ffgn-1.0.0
export CUDA_DEVICE_ORDER=PCI_BUS_ID        # GPU numbers as in nvidia-smi
./ffgn --pool neb.ariabrain.com:3370 --wallet neb1YOURADDRESS --worker rig1
```

The dataset is found next to the binary. Run the miner as root (or `sudo`) if you want clock locks and undervolt. To keep it
running after you log out use `screen` or `tmux`, or a systemd unit:

```ini
[Unit]
Description=ffgn NEB miner
After=network-online.target
Wants=network-online.target

[Service]
Environment=CUDA_DEVICE_ORDER=PCI_BUS_ID
WorkingDirectory=/opt/ffgn
ExecStart=/opt/ffgn/ffgn --pool neb.ariabrain.com:3370 --wallet neb1YOURADDRESS --worker rig1
Restart=always
RestartSec=15

[Install]
WantedBy=multi-user.target
```

(put the unpacked folder in `/opt/ffgn`; the unit runs as root, so clocks and undervolt work).

## 8. HiveOS

Short version; the full flight sheet with a ready one to import is in [HIVEOS.md](HIVEOS.md).

Flight sheet: Coin `NEB`, your wallet (the `neb1...` address), Pool **Configure in miner**, Miner **Custom**, then *Setup Miner
Config*:

| Field | Value |
|---|---|
| Miner name | `ffgn-hive` (filled in from the URL) |
| Installation URL | `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.0/ffgn-hive-1.0.0.tar.gz` |
| Hash algorithm | `nebhash` |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | `neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804` (several = failover; the first is the primary) |
| Pass | `x` (or empty) |
| Extra config arguments | optional, e.g. `--devices 0,1 --auto-lock` |

Apply it to the rig. The dataset is inside the package. The miner uses every GPU of the rig. Log: `miner log`. The dashboard
shows speed per GPU (1 "hash" = 1 attempt, so kH/s = MA/s x 1000), temperature, fans, accepted/rejected shares, and the version
column carries the efficiency (`1.0.0 0.0240 MA/W`) and `need update!!` when a newer release exists.

- The clock of the rig must be right: check `timedatectl` (time synchronisation on).
- The miner's own clock and undervolt options win over the OC profile while it runs; your values are put back when it
  stops. Do not set a core offset in the OC profile when you use `--auto-uv` or `--auto-lock`.
- HiveOS caches archives by file name: after an update use the new Installation URL.
- Tuning results are kept in `/hive-config/ffgn` (survive reboots and updates).

## 9. Pools: Aria and Suprnova

Enter the address, a dot and a worker name as the login (`neb1YOURADDRESS.rig1`); the scripts and the HiveOS template do it
for you. The pool password is `x` or empty. The miner recognises the pool protocol by itself; a bare `HOST:PORT` is enough.

| Pool | Address | Mode |
|---|---|---|
| Aria | `neb.ariabrain.com:3370` | PPLNS |
| Aria, solo | `neb.ariabrain.com:3371` | the finder of a block gets the reward |
| Suprnova | `neb.suprnova.cc:7804` | PPLNS, vardiff for GPUs (start 500 000 000) |
| Suprnova, solo | `neb.suprnova.cc:7805` | solo, vardiff |

Other Suprnova ports (`7801` vardiff 20 000, `7802` fixed 20 000, `7803` fixed 2 000 000) work the same way. Fees and payout
rules are the pools': read them on https://pool.ariabrain.com/neb.html and https://neb.suprnova.cc (fees announced on
16 November: Aria 2 %, Suprnova 1 %; payouts from 1 NEB per hour on Aria and from 0.01 NEB on Suprnova). The pools were on the
NEB test network until the main network starts: **before you mine for real check on the pool's page that it works on the
main network** (a pool still on the test network pays test coins that are worth nothing).

Several pools in one command = failover: the first is the primary, the next ones are backups, and the miner goes back to the
primary by itself (section 15):

```
ffgn --pool neb.ariabrain.com:3370,stratum+tcp://neb.suprnova.cc:7804 --wallet neb1YOURADDRESS --worker rig1
```

Address forms: `HOST:PORT`, `stratum+tcp://HOST:PORT` (plain TCP), `stratum+ssl://HOST:PORT` (TLS). Check a pool without a GPU
and without sending anything: `ffgn --pool-check --pool neb.ariabrain.com:3370` (connects, logs in, waits for a job, exits).

Where to see your shares: the miner log (section 16) and the statistics page of the pool (enter your address). Aria does not
answer each share; its page and the miner's "sent" counters are the proof that shares arrive. Suprnova answers every share.

## 10. Solo through your own node

You run the official NEB node on the same computer; ffgn takes the work from its work port `127.0.0.1:3441`, and the block
reward goes to the address behind the node's `--miner-lock`.

**Get the node and the wallet** from the official NEB release page https://github.com/SilasEigen/neb/releases/tag/v1.0.0 :
the archive for your system (`neb-1.0.0-windows-x86_64.zip` or `neb-1.0.0-linux-x86_64.tar.gz`) and `SHA256SUMS`. The Linux
node needs glibc 2.34 or newer. **Check before you run anything:** compare the SHA-256 of the archive with `SHA256SUMS`
(PowerShell: `Get-FileHash -Algorithm SHA256 <file>`) and verify the signature of `SHA256SUMS` (`SHA256SUMS.asc`, key
`release-key.asc`):

```bash
gpg --import release-key.asc
gpg --fingerprint            # compare with: EBF3 A9A0 37BC D121 0464  2A30 D7FC 5241 9E25 BBC4 (get the fingerprint from a
                             # second independent source, e.g. the project's site, too)
gpg --verify SHA256SUMS.asc SHA256SUMS
```

Unpack it into a folder without spaces (`C:\neb` or `~/neb`). The dataset file is the same `neb-datasets.bin` as in the ffgn
package: copy it there (the node refuses a file with another hash).

**Run the node:**

```bash
cd ~/neb
./neb-node --run --dataset neb-datasets.bin --datadir data --miner-lock "$(./neb-wallet --wallet wallet.txt lock)"
```

Windows PowerShell:

```powershell
cd C:\neb
$lock = .\neb-wallet.exe --wallet wallet.txt lock
.\neb-node.exe --run --dataset neb-datasets.bin --datadir data --miner-lock $lock
```

(the wallet asks for the password to print the lock; you can also paste the 64 hex characters.) Make sure the lock is **yours**.

- The node prints its **genesis id**; on the main network it must be
  `fcf3b2bdbb0a3f7621c14db4e229459375a1614d0480b4aa87a9aa01d3d58f88` (`neb-node --genesis` prints it too).
- It dials the public relays `seed1.neuralenginebyte.org:3440` and `seed2.neuralenginebyte.org:3440` and finds more peers.
- **Before the start** (14:00 UTC on 18 November) the node sits at height 0 and says how long is left. That is normal. Start
  the node and the miner early: the miner connects at any time and gets its first job about two minutes before the start.
- Without `--miner-lock` the node refuses every miner. Keep the folder `data`; never reuse the data folder of a test node.
- Ports: 3440 (peers; outgoing is enough), 3441 (miners, **local only**), 8432 (JSON-RPC for the wallet, local only).

**Run the miner** on the same computer: `ffgn --node 127.0.0.1:3441` (Windows: `start-solo.bat`). A list of nodes
`--node A,B` is a failover list. No wallet or worker is needed: the node pays its own lock.
The miner sends every share and every block candidate to the node; the log shows `BLOCK submitted` and
`BLOCK ACCEPTED by the node` when you find a block. Finding a block alone is a random event: be patient.

## 11. Rigs and solo: ffgn-gate

The work port of the node is local only and has no password. To let rigs mine solo through a node, run the gate beside the
node. It is a small server inside `ffgn` that speaks the pool protocol to the rigs and the work protocol to the node, gives
every rig its own range of nonces, checks every share again and forwards it. The node stays local.

On the node's computer:

```bash
echo "a-long-secret-token-of-16+-characters" > gate-token.txt
./ffgn --gate --node 127.0.0.1:3441 --gate-token-file gate-token.txt --gate-listen 0.0.0.0:3442
```

On a rig (Windows `EXTRA`, Linux, or the HiveOS Pool URL `GATE-HOST:3442` with the token in **Pass**):

```bash
./ffgn --pool GATE-HOST:3442 --password a-long-secret-token-of-16+-characters --wallet neb1YOURADDRESS --worker rig1
```

The token is at least 16 characters. The login is free and does not choose who gets paid: **the block reward goes to the
node's `--miner-lock`.** The gate speaks plain TCP: use it in your own network or through a VPN, and do not open port 3442 to
the internet. `--gate-slots-bits B` allows at most 2^B rigs (default 6 = 64). At most 20 submits per second per rig are accepted
and a rig that sends more than 20 bad tickets is disconnected.

## 12. All options

Every option has a default; `ffgn --help` prints the full list of your version. Lists: one value for all GPUs or one per
GPU in `--devices` order (`-` = leave that GPU alone).

**Work source**

| Option | Meaning |
|---|---|
| `--pool HOST:PORT` | pool; `stratum+tcp://` plain, `stratum+ssl://` TLS; several (`A,B` or `--pool A --pool B`) = failover, the first is primary |
| `--wallet neb1...` | the address the pool pays (required with `--pool`) |
| `--worker NAME` | worker name (login `ADDRESS.NAME`; default none) |
| `--password TEXT` | pool password (default `x`); for a gate: the gate token |
| `--node HOST:PORT[,..]` | solo: work service of a NEB node (default port 3441; local only) |
| `--dataset FILE` | `neb-datasets.bin` (default: next to the binary, in the working folder, or the HiveOS folder) |
| `--gate`, `--gate-listen ADDR`, `--gate-token-file FILE`, `--gate-slots-bits B` | run the gate (section 11) |
| `--pool-check` | log in to every pool of the list, print the verdicts, exit (no GPU needed) |

**GPUs and running**

| Option | Meaning |
|---|---|
| `--devices 0,1,...` | GPUs to mine on (default all; honours `CUDA_VISIBLE_DEVICES`; numbering follows `CUDA_DEVICE_ORDER=PCI_BUS_ID`) |
| `--minutes N` | stop after N minutes (settings are put back) |
| `--blocks N` | stop after N accepted blocks (solo tests) |
| `--self-test` | reference vs the NEB vectors, the engine's self-test, device hits vs reference; exit |
| `--hash-bench SECS` | attempts per second per GPU for KEP1 and RET2, with watts; exit |
| `--backend cuda\|cpu`, `--cpu-threads N` | `cuda` (default) is the GPU engine; `cpu` is the reference on CPU threads, for tests only |
| `--gpu-state` | print the state of the GPUs (NVML) |
| `--no-color`, `--version`, `--help` | log colours off (also `NO_COLOR=1`); version; help |

**Clocks and power** (NVIDIA, administrator / root; your values are put back on exit)

| Option | Meaning |
|---|---|
| `--lock-cclock N` | lock the core clock to N MHz |
| `--lock-mclock N` | lock the memory clock to N MHz |
| `--keep-hive-mem` | keep a HiveOS OC profile memory lock below the card's maximum (by default ffgn replaces it with the maximum while mining, see section 13) |
| `--moffset N` | memory clock offset in MHz, may be negative (driver 555 or newer) |
| `--temp-limit C[,C]` | keep the GPU at or below C degrees (40..100); a duty-cycle controller, for laptops |
| `--temp-stop C2[,C2]` | hard stop at C2 until the GPU cools to the limit minus 6 (default limit + 10) |

On exit a lock that was there before is put back: the HiveOS OC profile lock or `$FFG_RESTORE_CCLOCK` / `$FFG_RESTORE_MCLOCK`;
otherwise it is reset.

**Undervolt and lock search** (section 13)

| Option | Meaning |
|---|---|
| `--auto-uv` | find and apply a safe undervolt (V/F offset) per GPU, every step checked bit for bit |
| `--uv-offset N` | a fixed offset of N MHz after a quick check, instead of tuning |
| `--uv-min N`, `--uv-max N`, `--uv-step N` | where the tuner starts, the highest offset, the step (default 15 MHz) |
| `--uv-coarse N` | climbing step (default 30), refined with `--uv-step` inside the failing interval |
| `--uv-keep-last` | after a failed step keep the last good step (default: one step lower) |
| `--uv-reval-hours H` | re-check the undervolt every H hours while mining (default 6, 0 = never) |
| `--uv-dir DIR` | where the profiles are kept (default next to the binary; `/hive-config/ffgn` on HiveOS) |
| `--uv-reset`, `--uv-reset-all` | forget the profile of the current clocks / every saved result |
| `--auto-uv-dry-run` | only log what `--auto-uv` would change |
| `--auto-lock` | choose the core clock lock of each GPU by measurement (turns on `--auto-uv`) |
| `--auto-lock-range A-B[:S]` or `L,L,..` | locks to try (default 1800-1200, step 100, capped by the card's maximum) |
| `--auto-lock-max-loss PCT` | allowed hash rate loss for efficiency (default 10) |
| `--auto-lock-goal eff\|speed` | best efficiency within the loss (default), or the top speed |
| `--auto-lock-reset` | forget the measured locks and sweep again |

Default maximum offsets: RTX 20 / GTX 16 225, RTX 30 / 40 / 50 495, CMP 50HX 330, CMP 30HX / 40HX 150 (`--uv-max`).

**Pool watchdogs** (section 15)

| Option | Meaning |
|---|---|
| `--failback-minutes N` | on a backup pool check the primary every N minutes (default 10, 0 = off) |
| `--noshare-minutes M` | no accepted share for M minutes although the GPUs work: reconnect, then the next pool (default 15, 0 = off) |
| `--reject-minutes K` | only rejected shares (at least 5) for K minutes: the same (default 5, 0 = off) |

**Idle mode (Windows)** (section 14)

| Option | Meaning |
|---|---|
| `--idle` | mine only while nobody uses the PC |
| `--idle-minutes N` | minutes without input before mining starts again (default 5) |
| `--active pause\|light:PCT` | while the PC is in use: pause (default) or mine PCT % of the time (1-99; `light` = 30) |
| `--idle-pause-for L` | programs that pause mining while they run, e.g. `cs2.exe,blender.exe` |
| `--idle-cpu PCT`, `--idle-gpu PCT` | another program over PCT % of the CPU (default 25) / of a GPU engine (default 10) counts as use; 0 = off |
| `--idle-ignore L` | programs whose load does not count as use |

**Monitoring and updates**

| Option | Meaning |
|---|---|
| `--api [HOST:]PORT` | read-only HTTP status: `/summary`, `/gpus`, `/metrics` (default host `127.0.0.1`) |
| `--no-update-check` | do not look for a newer release (default: one check about a minute after the start, then every 12 hours; only reported) |
| `--check-update` | check now and exit |

## 13. Clocks, undervolt, auto-lock: speed and watts

A miner at stock clocks draws all the power the card allows. Lowering the core clock lock loses little speed and saves a lot of
power, and an undervolt (a core clock offset: the same clocks at a lower voltage) saves more. The danger of an undervolt is
that an unstable card does not always crash: it can compute a **wrong** answer. ffgn therefore checks every step of
`--auto-uv` in a separate process: the NEB engine at full load and the GPU tickets compared **bit for bit** with the C
reference of NEB. Only a step that passes is kept. Results are saved per GPU, coin and clock lock; while one GPU is checked the
others keep mining; a GPU that crashes while mining is named in the log, remembered in `gpu-faults.json`, and gets a lower
offset, and a card with nothing left to lower is left out for an hour.

Simple recipes (Windows: `AUTO_UV`, `AUTO_LOCK` in the `.bat`, run as administrator):

- Do nothing: stock clocks, no risk.
- `--auto-uv`: the miner finds a safe undervolt at the clocks it runs. The first tuning takes some minutes per GPU while
  the other GPUs keep mining.
- `--lock-cclock 1500 --auto-uv`: a fixed lock and the undervolt for it. Profiles are per lock.
- `--auto-lock`: the miner mines at several locks (default from 1800 down to 1200 in steps of 100), each at its own tuned
  undervolt, measures 45 s of settling plus 120 s of speed and watts, and keeps the lock with the best efficiency within
  10 % of the top speed. The result is saved (`autolock.json`) and applied at once on later starts; `--auto-lock-reset`
  sweeps again. A GPU with its own `--lock-cclock` is not swept.

The speed alternates between KEP1 and RET2 blocks, so a measurement over a few minutes is noisy; the miner's efficiency
number is an average.

**RTX 3070, GPU engine 0.2.8** (core lock, power limit 200 W, memory at its stock 7001 MHz, no undervolt, the real dataset;
speed in million attempts per second, efficiency in million attempts per joule):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W | RET2, MA/s | RET2, W | RET2, MA/W |
|---|---|---|---|---|---|---|
| stock (limited by the 200 W) | 16.95 | 200 | 0.085 | 9.92 | 200 | 0.050 |
| 1800 | 16.96 | 200 | 0.085 | 9.92 | 200 | 0.050 |
| 1650 | 16.92 | 193 | 0.088 | 9.86 | 181 | 0.054 |
| 1500 | 15.55 | 160 | 0.097 | 9.61 | 161 | 0.060 |
| 1350 | 15.06 | 141 | 0.107 | 9.33 | 139 | 0.067 |
| 1200 | 14.24 | 130 | 0.110 | 8.59 | 128 | 0.067 |

**RTX 4070 Ti, GPU engine 0.2.8** (stock power limit 235 W, memory at its stock 10251 MHz, no undervolt, the real dataset):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W | RET2, MA/s | RET2, W | RET2, MA/W |
|---|---|---|---|---|---|---|
| stock (~2450, limited by the 235 W) | 37.59 | 235 | 0.160 | 17.76 | 235 | 0.076 |
| 2500 | 37.31 | 235 | 0.159 | 17.75 | 235 | 0.076 |
| 2250 | 34.44 | 190 | 0.181 | 17.58 | 212 | 0.083 |
| 2000 | 30.89 | 173 | 0.179 | 17.19 | 207 | 0.083 |
| 1800 | 27.72 | 156 | 0.177 | 16.28 | 198 | 0.082 |

On the RTX 3070 the best efficiency is near 1350 MHz and 1500 MHz is a balance (about 8 % less speed for 20 % less power); on
the RTX 4070 Ti 2250 MHz is the balance (about 4 % less speed for 10-19 % less power). KEP1 and RET2 blocks alternate, so the
average speed of a run lies between the two columns (RTX 3070 stock: about 12.5 MA/s, RTX 4070 Ti: about 24 MA/s). The
numbers are the engine benchmark on the real main-network dataset and test runs (the mining loop gives the same speed);
your card, cooling and silicon will give somewhat different numbers.

### Memory clock

NEB reads its tables from the video memory all the time: the hash rate follows the memory clock. A memory clock locked low (the
HiveOS OC profiles made for Pearl lock it to 810 MHz; MSI Afterburner profiles with a memory underclock) makes NEB several
times slower. Measured with engine 0.2.8: RTX 3070 at 5001 MHz instead of 7001 — KEP1 12.55 / RET2 7.40 MA/s (−26 % / −25 %),
at 810 MHz — 1.64 / 0.97 MA/s (−90 %); RTX 4070 Ti at 5001 MHz — RET2 8.85 MA/s (−50 %; KEP1 stays near 37.7, its table fits in
the cache), at 810 MHz — RET2 1.18 MA/s (−93 %). ffgn therefore:

- replaces a HiveOS OC profile memory lock below the card's maximum with the maximum while it mines, and puts the profile's
  value back on exit (`--keep-hive-mem` keeps the profile's value; `--lock-mclock N` sets your own);
- writes a WARNING in the log after the first minute when a card's memory runs below 60 % of its maximum.

## 14. Laptops, and mining only while you are away

**Temperature limit.** A laptop cannot set its fans. `--temp-limit 80` makes the miner lower the share of time the GPU works
so that the core stays about 2 degrees below the limit; mining starts soft (30 %) and ramps up; the shown speed is the real
one. `--temp-stop` is the hard stop.

**Idle mode (Windows).** `--idle` (`IDLE=1` in the `.bat`) mines only while nobody uses the PC: keyboard, mouse, a gamepad, a
fullscreen game, other programs working on the CPU (over 25 %) or on the GPU (over 10 %, a video also counts) mean "in use".
A locked screen counts as idle. While the PC is in use the GPUs get your clocks and offsets back and no undervolt check
starts. `--active light:30` mines 30 % of the time even while you work; `--idle-pause-for cs2.exe,blender.exe` pauses mining
whenever these programs run. Other mining programs are not counted as use.

## 15. Failover, watchdogs, restarts

- **Failover and failback.** With several pools the first is the primary. A dead pool sends the miner to the next one; while on
  a backup it tests the primary every `--failback-minutes` (10) on a separate connection (login and a job, nothing submitted)
  and goes back after two good tests in a row.
- **No-share watchdog.** The GPUs work, at least 10 shares were expected at the measured speed and the pool's share
  difficulty, but none of yours was accepted for `--noshare-minutes` (15): the pool is not crediting, so the miner reconnects;
  the same again moves to the next pool (a single pool: a reconnect with a warning). `--reject-minutes` (5) does the same for
  rejected shares. Pools that do not answer each share (Aria) can trigger the rejected rule only.
- **Stale guard.** Shares of a job that a new block voided, or of an old connection, are dropped and not sent.
- **Dead link.** 95 seconds of silence on the node or pool link = reconnect.
- **GPU faults.** A GPU that reports a wrong result or crashes (Xid) is handled alone: the offset steps down, a card with
  nothing left to lower is left out for 1 hour, the others keep mining. If every GPU stops, the miner restarts itself.
- **Your settings come back** after every kind of stop: Ctrl+C, `miner stop`, closing the console window, a crash (marker
  files restore the clocks at the next start).
- Windows `start.ps1` restarts a miner that exited after 15 seconds; HiveOS and systemd restart it too.

## 16. Reading the log, the API, the balance

Every 60 seconds one line (pool mode):

```
[  60s] GPU0 11.9 (139 W, 0.0856 MA/W)  GPU1 ... | total 23.8 MA/s, 278 W, 0.0856 MA/W | found 3 (bad 0) | user sub 3 acc 3 rej 0 (stale 0) | ...
```

Solo: `... | shares 3 | blocks sub 0 acc 0 rej 0 | ...`. **MA/s** = million attempts per second; `found` = what the GPUs found (each
one re-checked on the CPU), `bad` must stay 0. Lines you will meet:

| Line | Meaning |
|---|---|
| `[pool:NAME] share ACCEPTED (job ..., N ms)` | the pool accepted a share (green) |
| `REJECTED`, `ERROR` | a rejected share, a failure (red); some `stale` / "job not found" right after a new block are normal |
| `BLOCK candidate ... submitted` / `BLOCK ACCEPTED by the node` | a block found (solo) |
| `devfee: ...` | the dev fee slots (section 17), dark grey |
| `auto-uv`, `auto-lock` | tuning progress |

After the start: the GPUs, the dataset check (`BLAKE3 ... checked`), the connection, the first job, then the first accepted
shares within minutes: all fine.

**API.** `--api 4078` serves JSON on `http://127.0.0.1:4078/summary` (the rig), `/gpus` (per GPU: speed, watts, temperature,
fan, clocks, offsets, tuning state) and `/metrics` (Prometheus). Read-only; for other hosts use `--api 0.0.0.0:4078` and a
firewall. On HiveOS the API runs on `127.0.0.1:4078` automatically.

**Balance.** In a pool: the pool's page (immature rewards first, then payouts by the pool's rules). Solo: the node window shows
the height growing about once a minute; balance with
`neb-wallet --rpc http://127.0.0.1:8432 --wallet wallet.txt balance`; the node must be synchronised for it to be right.
Block explorer: https://neuralenginebyte.org/#/explorer .

## 17. Dev fee

ffgn contains a **1 %** dev fee: **one 60-second slot in every 100 minutes**, at a random moment inside the cycle (it does not
help to restart the miner). In the slot the GPUs mine for the developers, then the miner returns to you; the log says
`devfee: slot started` and `devfee: slot finished, back to the user`. The fee cannot be removed or changed (see
[LICENSE](LICENSE)).

- **Pool mode.** On pools that accept several logins in one session (Suprnova) the miner logs in a second time with the fee
  login (worker name `ffgn`) and sends the slot's shares under it. On Aria (one login per connection) it opens a second
  connection to the same pool shortly before the slot. If a pool refuses the fee login, the miner uses a connection of its
  own for the rest of the run, and if none works the slot is skipped: you are never blocked.
- **Solo.** The node pays its own `--miner-lock`, which the miner cannot touch, so during the slot the GPUs mine on a public
  pool (Aria, then Suprnova) under the fee login on a second connection. This needs an internet connection; without it the
  slot is skipped.
- **Through a gate.** Rigs behind an ffgn-gate pay the fee the same way as in solo: on a public pool.

The fee address (public, so you can check it on the pools):
`neb1fh4c7f95phnphxsh6tqv26cy2wzrf0ga4ljrs9uhc04mlyetr2xqw4nhfp` (worker name `ffgn`).

## 18. Troubleshooting and FAQ

| Symptom | What to check |
|---|---|
| The window closes at once | Run `start.bat` from a command prompt to read the message. Typical: `WALLET` is empty or wrong (`neb1...`, lower case); `nvidia-smi.exe not found` (install the NVIDIA driver); a file is missing (unpack the whole archive). |
| `dataset ... BLAKE3` error | `neb-datasets.bin` is damaged or not the main network's file. Unpack the package again and compare the SHA-256 (section 4). |
| `--self-test` fails or GPU errors | Update the NVIDIA driver; check cooling and power; try without clock and undervolt options (`--uv-reset-all` forgets old profiles). |
| No accepted shares | Check the pool address and port, the address (`neb1...`), and that the pool is on the main network. `ffgn --pool-check --pool HOST:PORT` tests the login. On Aria look at the pool's page: Aria does not answer shares one by one. Wait a few minutes: the share difficulty adapts to your speed. |
| Many stale / rejected shares | The clock is wrong (turn on time synchronisation: Windows `w32tm /resync`, Linux `timedatectl`), or the connection to the pool is slow or lossy. A few stale shares right after a new block are normal. |
| The speed jumps up and down | Normal: the blocks alternate between two networks. KEP1 blocks run about twice as fast in attempts per second as RET2 blocks; the average is what counts. |
| The speed is lower than in the table | Thermal or power limit, a lower clock lock, a laptop, another program on the GPU. Look at `--gpu-state` and the log (`--temp-limit`, idle mode). |
| The miner restarts itself | The message before the restart says why: a GPU fault (the undervolt offset is stepped down), a dead link, or the watchdog. Repeated GPU faults: use `--uv-reset-all` and a lower `--uv-max`, or no undervolt. |
| Solo: the miner gets no job | The node must run with `--miner-lock` on the **same** computer (port 3441 is local), and it must be synchronised. Before 14:00 UTC on 18 November the first job comes about two minutes before the start. |
| Solo: the node stays at height 0 | Normal before the start. After it: check the clock, the outgoing port 3440 and the genesis id (section 10). |
| Solo: valid blocks rejected | The clock of the node is wrong (blocks dated more than 120 s ahead are refused). |
| Balance is 0 / "immature" | Rewards mature after 100 blocks; a solo node must be synchronised. |
| HiveOS does not start the miner | The Installation URL of the right release; algorithm `nebhash`; wallet template `%WAL%.%WORKER_NAME%`; read `miner log`. |
| Windows: clocks / undervolt not applied | Start the `.bat` with Run as administrator. |
| Antivirus warning | Mining software is often flagged. Compare the SHA-256 with `SHA256SUMS`, then add an exclusion if you decide to trust it. |

Questions and bug reports: https://github.com/pepsykolya/FreeForGemNeb/issues, Telegram channel
[@freeforgem](https://t.me/freeforgem) and chat [@freeforgem_chat](https://t.me/freeforgem_chat). Please include the version
(`ffgn --version`), the card, the driver and the first 50 lines of the log.

## 19. Updating, removing, contact

- **Update.** The miner only reports a newer release (in the log and in the HiveOS version column); it never updates itself.
  Stop it, unpack the new archive over a fresh folder, copy your edited `.bat` and the state files (`uv.json`, `autolock.json`
  next to the binary) if you want to keep the tuning results. On HiveOS use the new Installation URL.
- **Remove.** Windows: run `remove-autostart.bat`, delete the folder. Linux / HiveOS: delete the folder (HiveOS: remove the
  flight sheet). Tuning results on HiveOS are in `/hive-config/ffgn`.
- Licence: [LICENSE](LICENSE) (free to use and redistribute unmodified; the 1 % dev fee must stay). Third-party components:
  [NOTICE](NOTICE).
