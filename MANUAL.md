# FreeForGemNeb (ffgn) 1.0.12 — manual for beginners

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
that the chain itself owns. On the main network (NEB 1.0.1) both dataset slots run **KEP1**, a network on NASA Kepler transit
signals (slot 0: Kepler transit signals, 13,404 samples; slot 1: TESS signals, 7,089 samples, eclipse or junk), so every block
has the same speed (section 18). The retinal-scan network RET2 is gone from the main network; it exists only on the old test
network.

**What changed in 1.0.2.** ffgn 1.0.2 is required for the NEB 1.0.1 main network; ffgn 1.0.1 does not work on it. NEB 1.0.1
(https://github.com/SilasEigen/neb/releases/tag/v1.0.1) replaced NEB 1.0.0 before the start (the start time is unchanged): new
genesis id, a new dataset file (about 16 MB instead of 222 MB). The pools (Aria, Suprnova) must run NEB 1.0.1 too; ffgn checks
the pool's dataset hash and fails over to the next pool if it differs. The GPU engine is unchanged (0.2.8), the dev fee is 1 %.

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
- The dataset file `neb-datasets.bin` (about 16 MB) is **inside the package**: nothing else to download for pool mining. The miner
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
| Windows x64 | `ffgn-1.0.2-windows-x64.zip` | `ffgn.exe`, `nebgpu.dll`, `neb-datasets.bin`, `start.bat`, `start-suprnova.bat`, `start-solo.bat`, `start.ps1`, `install-autostart.bat`, `remove-autostart.bat`, README, MANUAL, LICENSE, NOTICE |
| Linux x86_64 | `ffgn-1.0.2-linux-x86_64.tar.gz` | `ffgn`, `libnebgpu.so`, `neb-datasets.bin`, docs |
| HiveOS | `ffgn-hive-1.0.2.tar.gz` | the custom miner `ffgn-hive`: `ffgn`, `libnebgpu.so`, `neb-datasets.bin`, `h-*.sh`, docs |

Release page: https://github.com/pepsykolya/FreeForGemNeb/releases (tag `v1.0.2`). The GPU engine (`nebgpu.dll` /
`libnebgpu.so`) is closed source and ships as a binary; everything else is described in NOTICE.

**Check the download** against `SHA256SUMS` of the release. Windows: `certutil -hashfile ffgn-1.0.2-windows-x64.zip SHA256`;
Linux: `sha256sum -c SHA256SUMS`. The dataset file `neb-datasets.bin` in the package is the file of the NEB 1.0.1 main
network (it was 222 MB for NEB 1.0.0; ffgn 1.0.2 refuses that old file and tells what to download). Source:
https://github.com/SilasEigen/neb/releases/download/v1.0.1/neb-datasets.bin ; it holds two NMD2 datasets, both of the KEP1 network:

```
size     16087037 bytes
SHA-256  57e22a4b8aacecb2ebae62f76307ddccb04198ed5c4ba15c1dee90ca7a7c2174
BLAKE3   790b57fbd830f3dcbe7de9d1146aa89a0a20799118e30dd48a343a5c98f19e72
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

1. Download `ffgn-1.0.2-windows-x64.zip`, check it (section 4) and unpack it into a folder **without spaces and without
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
mkdir -p ~/ffgn && tar -xzf ffgn-1.0.2-linux-x86_64.tar.gz -C ~/ffgn && cd ~/ffgn/ffgn-1.0.2
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
| Installation URL | `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.2/ffgn-hive-1.0.2.tar.gz` |
| Hash algorithm | `nebhash` |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | `neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804` (several = failover; the first is the primary) |
| Pass | `x` (or empty) |
| Extra config arguments | optional, e.g. `--devices 0,1 --auto-lock` |

Apply it to the rig. The dataset is inside the package. The miner uses every GPU of the rig. Log: `miner log`. The dashboard
shows speed per GPU (1 "hash" = 1 attempt, so kH/s = MA/s x 1000), temperature, fans, accepted/rejected shares, and the version
column carries the efficiency (`1.0.2 0.0240 MA/W`) and `need update!!` when a newer release exists.

- The clock of the rig must be right: check `timedatectl` (time synchronisation on).
- The miner's own clock and undervolt options win over the OC profile while it runs; your values are put back when it
  stops. Do not set a core offset in the OC profile when you use `--auto-uv` or `--auto-lock`.
- HiveOS caches archives by file name: after an update use the new Installation URL.
- Tuning results are kept in `/hive-config/ffgn` (survive reboots and updates).
- Recommended: the HiveOS Watchdog (minimal hashrate about 50 % of normal, restart the miner after 10-15 min, reboot after
  20-30 min) and `--reboot-on-lost-gpu 10` in Extra config: a GPU that fell off the bus gets the rig rebooted (section 15).

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
and without sending anything: `ffgn --pool-check --pool neb.ariabrain.com:3370` (connects, logs in, waits for a job, checks that the job is of the network of
your dataset file, exits; a pool on another network is `FAILED: WRONG NETWORK`).

Where to see your shares: the miner log (section 16) and the statistics page of the pool (enter your address). Aria does not
answer each share; its page and the miner's "sent" counters are the proof that shares arrive. Suprnova answers every share.

## 10. Solo through your own node

You run the official NEB node on the same computer; ffgn takes the work from its work port `127.0.0.1:3441`, and the block
reward goes to the address behind the node's `--miner-lock`.

**Get the node and the wallet** from the official NEB release page https://github.com/SilasEigen/neb/releases/tag/v1.0.2 :
the archive for your system (`neb-1.0.1-windows-x86_64.zip` or `neb-1.0.1-linux-x86_64.tar.gz`) and `SHA256SUMS`. The Linux
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
  `da2f5af7bb4c5e2bcc8d558b2f4f01d4c88557a943e6cff9cd035638fe9439ae` (`neb-node --genesis` prints it too).
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
| `--password TEXT` | pool password (default `x`); for a gate: the gate token. 1.0.3: sent only to the first pool of `--pool` or to an ffgn gate; backup pools get `x` |
| `--node HOST:PORT[,..]` | solo: work service of a NEB node (default port 3441; local only) |
| `--dataset FILE` | `neb-datasets.bin` (default: next to the binary, in the working folder, or the HiveOS folder). A job is mined only with the dataset file of its own network; for the old test network pass the testnet file of NEB 0.3.1 here |
| `--gate`, `--gate-listen ADDR`, `--gate-token-file FILE`, `--gate-slots-bits B` | run the gate (section 11) |
| `--pool-check` | log in to every pool of the list, print the verdicts, exit (no GPU needed) |

**GPUs and running**

| Option | Meaning |
|---|---|
| `--devices 0,1,...` | GPUs to mine on (default all; honours `CUDA_VISIBLE_DEVICES`; numbering follows `CUDA_DEVICE_ORDER=PCI_BUS_ID`) |
| `--minutes N` | stop after N minutes (settings are put back) |
| `--blocks N` | stop after N accepted blocks (solo tests) |
| `--self-test` | reference vs the NEB vectors, the engine's self-test, device hits vs reference; exit |
| `--hash-bench SECS` | attempts per second per GPU for KEP1 (and RET2 of the test network), with watts; exit |
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
| `--table-mclock N` | NEB: lock the memory to N MHz (e.g. 810) only while a GPU mines on the pure digest-table path; the full memory clock comes back by itself for every table build, the L=3 path and every other path (same list syntax as `--lock-mclock`; root / Administrator) |
| `--tbig MODE` | NEB: partial L=3 digest table of the TESS slot (1.0.12): `auto` (default) builds the triples while mining and sizes them per model from the measured build speed and model lifetime (none when they would not pay back); `on` = all the free VRAM at each model start; `off` |
| `--tbig-gb G` | NEB: at most G GB for the TESS triples (default: the free VRAM minus a reserve for the Kepler table) |
| `--power-limit N` (alias `--pl N`) | set the GPU power limit to N W (like `nvidia-smi -pl`; root / Administrator). One value for all GPUs, or one per GPU in `--devices` order: `--pl 140,115,140` (`-` = leave that GPU's limit as it is, e.g. the HiveOS one; a list needs one value per GPU, like `--uv-max`). The value is clamped to the card's range (the log says so). The limit from before is put back on exit, on Ctrl-C and at the next start after a crash (one file per GPU, `power-limit-restore-<bus>.json` in the state folder; on HiveOS the OC profile's power limit); an OC apply that changes it while mining is undone (one log line; at most 4 times an hour per GPU). auto-uv measures with the limit in force; when the limit holds the clocks down under load (power cap) the log says so once. Stats line: `GPU0 47.1 (139 W, PL 140 W capped, 0.3388 MA/W)`; API `/gpus`: `power_limit_w`, `power_limit_ours`, `power_limit_set_w`, `power_limit_binds` |
| `--single-process` | run every GPU's engine in the miner process (before 1.0.11); default: one engine process per GPU, so a fault on one GPU restarts only that GPU |
| `--reboot-on-lost-gpu MIN` | opt-in: reboot the rig when a GPU is lost (Xid 79 / `DEAD` / gone from NVML) for MIN minutes (recommended on rigs: `10`): HiveOS `sreboot`, else `systemctl reboot`; Windows `shutdown /r /t 60`. Never in the first 10 min of the miner, at most once per 60 min; off with `--single-process` or `FFGN_NO_REBOOT=1`; root / Administrator. See section 15 |
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
| `--uv-parallel N` | 1.0.5: GPUs tuned (or re-checked from a saved profile) at the same time, the others mine meanwhile; default auto: 1 on rigs of 6+ GPUs, else 2, then 1 -> 2 -> 4 after 2 GPUs tuned without trouble (never above the free CPU threads nor the RAM of the checks); trouble -> 1, remembered for the rig |
| `--uv-unpause` | 1.0.5: forget the guards of auto-uv (a rig paused after restarts during a tuning, GPUs off for 24 h, the remembered caution); also `--uv-reset-all` |
| `--ramp-ms N` | 1.0.5: after a pause the GPUs go back to full load one by one, N ms apart (default 500 on rigs of 4+ GPUs, else 0); such rigs also build a new block's model 30 ms apart per GPU |
| `--kep1-kernel auto\|dp4a\|warp` | 1.0.5: the KEP1 attempt kernel (default auto; 1.0.6: both timed at every new model on its first mining launches, the faster kept; `warp` needs RTX 30 / A-series or newer) |
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
offset, and a card with nothing left to lower is left out for an hour. After a crash or a failed check (1.0.3) the card does
not tune again from +0: it resumes 2 steps (30 MHz) below the failed offset, never above the last confirmed one, after a
30 s check, and mines; if that fails too, 2 steps lower again, down to the base.

**1.0.3: the lock and the watts are checked.** Every check also reads the real core clock under load. A step whose clock runs
above the core lock (+45 MHz, 3 clock bins) is refused: on GeForce Turing (RTX 20xx) an offset with a lock can raise the clock
instead of lowering the voltage. At the start offset above the lock = the lock was dropped (HiveOS `nvidia-oc` resets it when an
OC profile is applied): the miner sets it again, also while mining (`set again` in the log). If it still does not hold:
`WARNING GPUn: the core clock lock N MHz does not hold (runs at M MHz): auto-uv is off for this GPU; use the power limit of the
OC profile`: the offset goes back to +0, the API (`core_lock_holds`, `warnings`) and the HiveOS version field show it. A step is
taken only if the power did not rise. **1.0.4 (replaces the 3 % rules of 1.0.3):** a step is taken when its power is lower than
the step it climbs from by more than the noise of a measurement (0.5 W: repeated 15 s checks of an RTX 3070 at one offset
differ by ~0.4 W); every step is compared with the best (lowest) power of the climb; a step that does not beat it is not taken,
but the climb goes on: single readings between offsets move by 1-2 W (an RTX 3070 @1650 swept 0..+270 fell from 152 to 129 W,
~1-1.5 W per 15 MHz, yet +60 once read 2 W above +30); 3 steps in a row that do not lower the power, or a rise by more than
3 W, end the climb; a step that did not lower it while the card
warmed by 2 C or more is measured once more. The result is the highest stable offset that
still lowered the power, without any percentage threshold (1 W per step is a real saving). A profile saved by an earlier
version is not tuned from +0: its offset is checked (30 s) and the climb goes on upward from it (never above a failed offset
of the profile, nor into an offset that crashed while mining). Unchanged: the clock against the lock, the bit-exact check,
a crash -> 2 steps below it, `--uv-max`. A step whose clock runs above the lock (1.0.4): the lock is set again and the step
checked once more, only then refused. A lock that does not hold turns off `--auto-lock` for that GPU too (saved in
`autolock.json`; `--auto-lock-reset` tries again).

**1.0.10: tuning while mining.** The steps are measured on the card's own mining load: the card keeps mining on the path the engine chose for the model, sends shares and counts its MA/s. A step alternates the candidate offset (5 s) with the best offset so far (3.5 s) and compares their power only under the same model and path, so a new block or a Kepler / TESS switch does not look like a saving. A step ends as soon as the difference is decided against the 0.5 W rule (at least 8 s at the candidate, at most `--uv-step-secs`, 25 s). A step is stable when the card had no Xid, its worker stayed alive and none of its results was wrong (every hit is checked on the CPU). Time without work is not counted. On the mining load the climbing step is +60 (refined by +15 steps near a failure; `--uv-coarse 30` for a finer climb), and the 60 s confirmation runs once, at the result. All GPUs are tuned at once; `--uv-parallel N` still limits the number. Measured on an RTX 3070: a fresh tuning took 120 s including one fault and an automatic restart of the miner (about 17 s per step); a start from a saved profile needs one 17-s check. The card ended at +240 with 124-126 W against 129-130 W at a fixed +210, at the same rate (about -3 %). A hard Xid at the edge costs one self-restart of the miner of about 10 s. A whole rig tunes in roughly 2-5 min, all cards in parallel (estimate). The old separate check process (the card's mining paused while it is checked, the rules below) is used only for a card that does not mine within 240 s, or for every card with `FFGN_UV_CHILD=1`. auto-uv and auto-lock lines are yellow in the log, their failures and roll-backs orange (`--no-color` turns colours off).

**1.0.5-1.0.9 (and the fallback check process since 1.0.10): rigs with a weak CPU, measurement.** A check process is a whole NEB engine at full load (its own CUDA context, the
dataset tables, the CPU reference). On a rig with many GPUs and a small CPU (12x RTX 2060 SUPER on a 2-thread Pentium with
7.6 GB of RAM) twelve of them at once hung the whole rig in 1.0.4, and the power supply and the risers are invisible to
software. Since 1.0.5 the tuning starts small and grows only while the rig stays healthy: 1 GPU at a time on rigs of 6+ GPUs
(else 2), 2 and then 4 after every 2 GPUs tuned without trouble, never more than the free CPU threads nor the RAM of the checks
(1.2 GB each). Trouble during a tuning (a GPU lost from the bus, a check that hangs, all GPUs stalled at once, a mining GPU that
computes wrong) brings it back to 1 at once and is remembered for the rig (`uv-parallel.json`: later starts stay at 1 until
`--uv-unpause` / `--uv-reset-all`). `--uv-parallel N` sets a fixed number. The log says how many and why
(`auto-uv: parallel tuning 1 -> 2: 2 GPUs tuned without trouble ...`). The GPUs that wait mine at their current offset, and a
GPU's own mining pauses while it is checked. Every step that lowered the power is saved at once (as "tuning in progress"):
after a restart the climb goes on from it instead of from the start (a failure in the middle of a check still counts: 2 steps
below it).

**1.0.5: no endless tuning, no reboot loops.** Each GPU is tuned once; a saved result (also one in progress) is used; it is
tuned again only after a failure on that GPU (2 steps below the failure, no climb above it) or as the short re-check every
`--uv-reval-hours`. Guards (`uv-guard.json`):
- a GPU that starts with a failure at its tuned offset 3 times in a row gets no auto-uv for 24 h and mines at its own offset
  (`WARNING auto-uv: GPU3 ... auto-uv OFF for this GPU for 24 h`, `[AUTO-UV GPU3 OFF 24h]` in the stats line and in HiveOS,
  `warnings` in the API);
- the miner writes a mark before a GPU's tuning and removes it when the GPU mines; a start that finds it (the miner or the rig
  stopped in the middle: a restart, a reboot, ffgn's own restart) first mines 7 minutes and tunes on only if the rig stayed
  calm; 2 such starts within an hour pause auto-uv on the rig (`[AUTO-UV PAUSED]`) until `--uv-unpause` / `--uv-reset-all` or
  a new miner version: it mines with the saved results of this version, else at the cards' own offsets;
- a run spends at most 10 minutes per GPU and 60 minutes in all on tuning; a GPU over its time mines at its last taken step and
  goes on climbing at the next start. The check processes start at least 3 s apart and wait (up to 90 s) for 700 MB of free RAM. The power of
a step is now the mean of the NVML samples after 10 s under load (steps of 25 s instead of 15 s; a short check of an RTX 2060
SUPER read 76.7 W and then 67.4 W at the same offset), the log gives its spread (`sd 0.4 W, 30 samples`), and a step that did
not clearly lower the power is measured once more in the order best-step-best-step (a warming card drifts both alike) before
it counts.

**1.0.5: power steps of a rig.** Rigs of 4+ GPUs: after a pause (pool or node lost, no job, idle mode, an auto-uv check) the
GPUs go back to full load one by one, 500 ms apart (`--ramp-ms`), and a new block's model is built 30 ms apart per GPU; the
start was already staggered (1.5 s per GPU). When all GPUs stop making attempts in the same few seconds while there is work,
the log says so once: `ERROR: all 12 GPUs stalled at once at 03:12:45 UTC (...): likely a PCIe bus / power event (risers, PSU,
cables), not the pool` (look for Xid 79 "GPU has fallen off the bus" in `dmesg`). On every new block's model the log says how long
each GPU stood idle and how far apart the GPUs went back to full load (`model change: GPU idle GPU0 41 ms, GPU1 44 ms; back at
work within 31 ms of each other`, API: `model_change_idle_ms`); the CPU check of a new model runs beside the mining (the GPU no
longer waits for it). The clock lock keeper only reads the clocks (every 5 s) and sets a lock again only when the card is seen
running off it.

**1.0.3: CPU.** The engine waits for the GPU asleep (it used to spin a whole CPU core per GPU): a 12-GPU rig on a 2-core CPU is
no longer at 100 % CPU.

Simple recipes (Windows: `AUTO_UV`, `AUTO_LOCK` in the `.bat`, run as administrator):

- Do nothing: stock clocks, no risk.
- `--auto-uv`: the miner finds a safe undervolt at the clocks it runs. The first tuning takes some minutes per GPU while
  the other GPUs keep mining.
- `--lock-cclock 1500 --auto-uv`: a fixed lock and the undervolt for it. Profiles are per lock.
- `--auto-lock`: the miner mines at several locks (default from 1800 down to 1200 in steps of 100), each at its own tuned
  undervolt, measures 45 s of settling plus 120 s of speed and watts, and keeps the lock with the best efficiency within
  10 % of the top speed. The result is saved (`autolock.json`) and applied at once on later starts; `--auto-lock-reset`
  sweeps again. A GPU with its own `--lock-cclock` is not swept.

On the main network every block is KEP1, so the speed is steady; the figures below are KEP1. A measurement over a few minutes
can still be noisy; the miner's efficiency number is an average.

Million attempts per second (MA/s) and million attempts per second per watt (MA/W), the real dataset, memory at its stock
clock, no undervolt. On the main network every block is KEP1 (slot 0 Kepler, slot 1 TESS: two datasets of different size).

**GPU engine 0.8.1 (ffgn 1.0.12)**, measured 21.11.2026, values per card (power = mean of the card's own readings under
load; MA/s = evaluated attempts).

Partial L=3 table for TESS (engine benchmark: one resident table, the same TESS models of the chain, alternating 8-s segments
with the triples ignored and used):

| Card, clocks (per card) | TESS, pairs only (as 1.0.11) | TESS, 1.0.12 with the triples | Rate | MA/W | Triples, full build |
|---|---:|---:|---:|---:|---:|
| RTX 4070 Ti 12 GB, 2250 MHz, PL 190 W | 415.1 MA/s, 178 W | **488.9 MA/s**, 179 W | +17.5 to +17.8 % | +17.2 to +18.0 % | 316 M in 2.69 s |
| RTX 3080 Ti 12 GB, default clocks | 404.0 MA/s | **472.3 MA/s** | +16.9 to +17.0 % | | 301 M in 2.35 s |
| RTX 3070 8 GB, 1680 MHz / +210 / 200 W | 238.9 MA/s, 133 W | **261.7 MA/s**, 133 W | +9.4 to +9.6 % | +9.4 to +9.8 % | 189 M in 2.72 s |
| RTX 4070 Laptop 8 GB | 198-218 MA/s, 75-85 W | **214-232 MA/s** | +6.1 to +8.3 % | +8.3 to +9.3 % | 169 M in 2.2-2.3 s |

These are steady-state numbers with the triples built. Every new TESS model needs its own triples. The miner builds them while
it mines and sizes them per model from the measured build speed and the measured TESS mining time per model. At 41 s of TESS
mining per model this is worth about +9 to +10 % per model on the 12 GB cards and about +2 % on the RTX 3070. When the models
are short-lived it builds none: live, an RTX 3070 measured 13 s per model and kept the pair table only, the 1.0.11 path.
The triples are exact: byte-compared with the full fold on every card, slice by slice.

Live, RTX 3070 (1680 MHz / +210 / 200 W, `--table-mclock 810`), FFGN 1.0.12 against 1.0.11 on a second RTX 3070 of the same
machine in the same minutes: Kepler rate x0.999. Power per watt was 1.7-2.4 % lower than the spread between the two cards on
1.0.11 alone, in one 10-minute window. 0 bad results. `--power-limit 150` on that card: 150 W while mining, the previous 200 W
again after the miner's exit.

These are measurements, not a recommendation to change your clocks. Auto-lock and existing OC settings are unchanged.

**GPU engine 0.7.0 (ffgn 1.0.11)**, measured 21.11.2026 (power = mean of the GPU's own readings under load; MA/s = evaluated
attempts). P = the model's share of attempts with at most two live rows.

Automatic L=3 on low-P Kepler models (engine benchmark, same recorded models, one process; table alone against the path the
engine chooses):

| Card, core lock | Model (P) | Table alone | 1.0.11 (table + L=3) | Rate | MA/W |
|---|---|---:|---:|---:|---:|
| RTX 3070, 1680 MHz / +210 / 200 W | live Kepler (0.73 %) | 47.0 MA/s, 127.4 W, 0.369 MA/W | **60.0 MA/s**, 137.5 W, **0.436 MA/W** | +28 % | +18 % |
| RTX 3070, 1680 MHz / +210 / 200 W | Kepler (0.65 %) | 42.3 MA/s, 130.6 W, 0.324 MA/W | **60.0 MA/s**, 138.3 W, **0.434 MA/W** | +42 % | +34 % |
| CMP 50HX (Turing), 1500 MHz | live Kepler (0.73 %) | 47.5 MA/s, 141.3 W, 0.336 MA/W | **54.3 MA/s** (L=3 at 14k-20k: 137-143 W, 0.381-0.387 MA/W) | +15 % | +13 to +15 % |
| CMP 50HX (Turing), 1500 MHz | Kepler (0.65 %) | 42.7 MA/s, 142.0 W, 0.301 MA/W | **54.3-55.0 MA/s**, 139.5-143.2 W, 0.384-0.390 MA/W | +27 to +29 % | +28 to +29 % |

On models with P 1.56 % (Kepler) and 3.92 % (TESS) the L=3 path is slower (x0.76 / x0.49 on the CMP 50HX), so the engine keeps it
off above 1.3 %. Live Kepler models of the last hours had P 0.62-2.25 %.

Digest-table build, full build of one model (byte-identical tables): RTX 3070 Kepler 1003-1192 ms -> 311-341 ms, TESS 227 -> 61 ms;
CMP 50HX Kepler 892-1063 ms -> 287-316 ms, TESS 196 -> 47 ms. Screener on RTX 30 / 40 (RTX 3070, interleaved, 3 x 15 s): Kepler
+1.0 % rate (+1.0 to +1.5 % per watt), TESS +2.25 % (+1.0 %).

`--table-mclock 810` on the pure table path (RTX 3070, engine benchmark, same minutes): rate -0.4 to -0.6 %, power 131 -> 121 W,
MA/W +7.9 % (Kepler) and +10.2 % (TESS). At 810 MHz every path that reads the memory is several times slower, which is why the
miner gives the full memory clock back for builds and for every other path.

Live on one RTX 3070 (1680 MHz / +210 / 200 W, same pool), 1.0.10 (46 min) against 1.0.11 with `--table-mclock 810` (47 min), Kepler blocks: screened nonces 6.31 -> 6.45 G/s (+2.1 %), power 132.2 -> 120.7 W, screened nonces per watt +11.8 %. The chain served no Kepler model below 1.3 % in these minutes, so the automatic L=3 stayed off, as it should; the TESS blocks were too short for a figure. 1.0.11: 150 shares found, 0 bad, at the pool 132 accepted and 2 stale, no Xid.

These are measurements, not a recommendation to change your clocks. Auto-lock and existing OC settings are unchanged.

**GPU engine 0.6.1 (ffgn 1.0.10)**, measured 21.11.2026 (power = mean of the GPU's own readings under load; MA/s = evaluated
attempts, G/s = screened nonces). On the table path the evaluated rate = screened nonces x the model's share of attempts with at
most two live rows, so the screened rate and the screened nonces per watt compare the two versions even when the chain's model
has changed between the runs.

Engine benchmark, 1.0.9 engine (0.6.0) against 1.0.10 engine (0.6.1) on the same recorded live models, in one process with the
same digest table (RTX 3070: 20-s segments; CMP 50HX: 12-s segments x 2 rounds):

| Card, core lock | Model (share of attempts with <= 2 live rows) | 1.0.9 engine | 1.0.10 engine | Rate | MA/W |
|---|---|---:|---:|---:|---:|
| RTX 3070, 1680 MHz / +210 / 200 W | Kepler (1.12 %) | 4.97 G/s, 55.8 MA/s, 110.3 W, 0.506 MA/W | **6.41 G/s, 71.9 MA/s**, 127.5 W, **0.564 MA/W** | +28.9 % | +11.5 % |
| RTX 3070, 1680 MHz / +210 / 200 W | Kepler (1.56 %) | 4.93 G/s, 77.1 MA/s, 112.0 W, 0.688 MA/W | **6.34 G/s, 99.1 MA/s**, 130.0 W, **0.762 MA/W** | +28.6 % | +10.8 % |
| RTX 3070, 1680 MHz / +210 / 200 W | TESS (3.92 %) | 4.72 G/s, 185.3 MA/s, 113.0 W, 1.640 MA/W | **6.07 G/s, 238.2 MA/s**, 133.8 W, **1.781 MA/W** | +28.5 % | +8.6 % |
| CMP 50HX (Turing), 1300 MHz | Kepler (1.12 %) | 4.64 G/s, 52.06 MA/s, 111.8 W, 0.466 MA/W | **5.60 G/s, 62.85 MA/s**, 118.0 W, **0.533 MA/W** | +20.7 % | +14 % |
| CMP 50HX (Turing), 1300 MHz | TESS (3.92 %) | 4.38 G/s, 171.66 MA/s, 112.8 W, 1.522 MA/W | **5.03 G/s, 197.48 MA/s**, 117.9 W, **1.676 MA/W** | +15.0 % | +10.1 % |

Live mining on one pool, screened nonces per second and per watt (the chain's models differ between the runs):

| Where | Slot | 1.0.9 | 1.0.10 | Rate | Per watt |
|---|---|---:|---:|---:|---:|
| RTX 3070, 1680 MHz / +210 / 200 W | Kepler | 4.95 G/s at 111.4 W | **6.18 G/s** at 132.8 W | +25 % | +4.8 % |
| RTX 3070, 1680 MHz / +210 / 200 W | TESS | 4.55 G/s at 117.5 W | **5.81 G/s** at 139.6 W | +27 % | +7.6 % |
| 5x CMP 50HX (Turing), 1300 MHz, rig total | Kepler | 23.43 G/s at 567 W | **26.51 G/s** at 605 W | +13.1 % | +6.1 % |
| 5x CMP 50HX (Turing), 1300 MHz, rig total | TESS | 21.09 G/s at 596 W | **21.73 G/s** at 604 W | +3.0 % | +1.6 % |

RTX 3070, 1.0.10, the first 16 min: Kepler (model share 1.93 %) 186.6 MA/s at 132.8 W (1.405 MA/W), TESS (8.62 %) 428.9 MA/s at
139.6 W (3.07 MA/W); 63 shares found, 62 accepted, 1 stale ("job not found" right after a job change), 0 bad. Five CMP 50HX,
1.0.10, 20 min: Kepler (2.77 %) 739.4 MA/s at 605 W for the rig (1.222 MA/W), 146.2-149.5 MA/s per card; TESS (9.13 %)
1981.5 MA/s at 604 W (3.276 MA/W), 389.5-404.1 MA/s per card; 117 shares found, 0 bad, at the pool 84 accepted and 1 stale.
On TESS at a high model share much of the GPU time goes to the attempts themselves, so the faster screen helps less there.
The faster screen draws more power, so the gain per watt is a third to a half of the gain in rate. We did not measure a desktop
RTX 40 / 50 with 0.6.1, and the RTX 4070 Laptop GPU's speed follows its thermal limit, so no figure is claimed for them.

These are measurements, not a recommendation to change your clocks. Auto-lock and existing OC settings are unchanged.

**GPU engine 0.6.0 (ffgn 1.0.9)**, measured 21.11.2026 (power = mean of the GPU's own readings under load; MA/s = evaluated
attempts). The rate of the new paths is proportional to the share of a model's attempts with at most two live rows, so it moves
with the chain's model: the share was 1.25-1.68 % for the Kepler models and 7.6-8.2 % for the TESS models during the live runs.

Live mining on one pool, RTX 3070, 1680 MHz / +210 / 200 W limit (1.0.8: the live lock sweep of 20.11.2026; 1.0.9: 20 min,
11 models, 61 shares accepted, 0 rejected, 0 bad):

| Slot | 1.0.8 (engine 0.5.1) | 1.0.9 (engine 0.6.0) | Change |
|---|---:|---:|---:|
| Kepler | 40.05 MA/s, 184.4 W, 0.2172 MA/W | **65.8-83.1 MA/s**, 110-113 W, **0.59-0.74 MA/W** | x1.6-2.1 speed, x2.7-3.4 MA/W |
| TESS | 40.21 MA/s, 162.5 W, 0.2474 MA/W | **343-358 MA/s**, 115-116 W, **2.98-3.09 MA/W** | x8.5-8.9 speed, x12 MA/W |

Engine benchmark on three recorded live models (one process per model, automatic path, 20 s after the choice):

| Card, core lock | Model (share of attempts with <= 2 live rows) | 1.0.8 path | 1.0.9 (digest table chosen) | MA/W change |
|---|---|---:|---:|---:|
| RTX 3070, 1680 MHz / +210 | Kepler (1.12 %) | — | **55.64 MA/s**, 111.0 W, **0.501 MA/W** | |
| RTX 3070, 1680 MHz / +210 | Kepler (1.56 %) | — | **76.81 MA/s**, 111.2 W, **0.691 MA/W** | |
| RTX 3070, 1680 MHz / +210 | TESS (3.92 %) | — | **184.32 MA/s**, 112.7 W, **1.636 MA/W** | |
| CMP 50HX (Turing), 1500 MHz | Kepler (1.12 %) | 18.60 MA/s, 154.4 W, 0.1205 MA/W | **59.80 MA/s**, 132.1 W, **0.453 MA/W** | x3.8 |
| CMP 50HX (Turing), 1500 MHz | Kepler (1.56 %) | 18.67 MA/s, 152.6 W, 0.1223 MA/W | **82.25 MA/s**, 133.1 W, **0.618 MA/W** | x5.1 |
| CMP 50HX (Turing), 1500 MHz | TESS (3.92 %) | 18.73 MA/s, 135.4 W, 0.1383 MA/W | **196.85 MA/s**, 135.2 W, **1.455 MA/W** | x10.5 |

(The CMP 50HX 1.0.8 numbers were measured on the same card and lock on 20.11.2026.) Live mining on a rig of five CMP 50HX at a
1300 MHz core lock (the best per-watt lock of the table path we measured on this card), 20 min, 5 models: Kepler 58.3-65.4 MA/s
per card at 109-119 W (0.49-0.59 MA/W), TESS 343.8-346.2 MA/s per card at 116-127 W (2.71-3.00 MA/W); 80 shares found, 79
accepted, 1 stale ("job not found" after a 1.5-s job), 0 bad. RTX 40 / 50 cards take the same paths; we did not measure a
desktop RTX 40 / 50 with 0.6.0, and the RTX 4070 Laptop GPU's speed follows its thermal limit, so no figure is claimed for them.

These are measurements, not a recommendation to change your clocks. Auto-lock, auto-uv and existing OC settings are unchanged.

**GPU engine 0.5.1 (ffgn 1.0.8)**, engine benchmark on the chain's trained models (20.11.2026, RTX 3070, 200 W limit,
0.4.0 and 0.5.1 measured in the same session, power = mean under full load, automatic sort choice):

| Core lock / offset | slot | 0.4.0 (1.0.7) | 0.5.1 (1.0.8) | change |
|---|---|---:|---:|---:|
| 1680 MHz / +210 | Kepler | 31.37 MA/s, 164.7 W, 0.1905 MA/W | **38.22 MA/s**, 185.2 W, **0.2064 MA/W** | +21.8 % speed, +8.4 % MA/W |
| 1680 MHz / +210 | TESS | 36.81 MA/s, 149.5 W, 0.2463 MA/W | **41.67 MA/s**, 165.1 W, **0.2524 MA/W** | +13.2 % speed, +2.5 % MA/W |
| 1620 MHz / +240 | Kepler | 30.73 MA/s, 152.0 W, 0.2022 MA/W | **37.23 MA/s**, 168.1 W, **0.2215 MA/W** | +21.2 % speed, +9.5 % MA/W |
| 1620 MHz / +240 | TESS | 35.49 MA/s, 137.5 W, 0.2581 MA/W | **40.20 MA/s**, 146.1 W, **0.2752 MA/W** | +13.3 % speed, +6.6 % MA/W |

The best measured efficiency of 0.5.1 on this card is at 1620 MHz / +240: Kepler 0.2215 MA/W and TESS 0.2752 MA/W, +16.3 %
and +11.7 % over 0.4.0 at 1680 MHz / +210, and still faster than 0.4.0 there. The power rises with the speed (the GPU waits
less between kernels), so the per-watt gain is smaller than the speed gain. On RTX 4070 Laptop GPU (32 MB L2 cache, both
tables fit in it, the engine does not sort) there is no clear change in speed or efficiency: its clocks follow its thermal
limit and move the speed more than the difference, so no gain is claimed for it. RTX 40 desktop and RTX 50 cards were not measured by us with 0.5.1 yet.

These are measurements, not a recommendation to change your clocks. Auto-lock, auto-uv and existing OC settings are unchanged.

**Stability note (added 20.11.2026):** in a later long live run on the same RTX 3070, a fixed V/F offset of +240 at 1620 MHz was not stable while mining the Kepler slot with the sorted attempt order (bad results caught by the miner's checks, then a GPU fault); +240 at 1560 MHz and +210 were stable. The best offset differs from card to card: use `--auto-uv`, which checks every step against the CPU reference and steps back after a failure, instead of copying an offset.

**Live mining with 1.0.8, core lock sweep** (20.11.2026, RTX 3070, 200 W limit; measured on one card, 3.5 min per point;
per slot: the mean of the miner's 10 s rate lines while that slot was mined, power = mean of nvidia-smi on that slot):

| Core lock / offset | Kepler MA/s | W | MA/W | TESS MA/s | W | MA/W |
|---|---:|---:|---:|---:|---:|---:|
| 1500 MHz / +210 | 36.00 | 146.3 | **0.2460** | 35.92 | 130.2 | **0.2760** |
| 1560 MHz / +210 | 37.44 | 161.4 | 0.2320 | 36.87 \* | 142.2 | 0.2593 |
| 1620 MHz / +240 | 38.84 | 169.5 | 0.2291 | 38.55 \* | 146.0 | 0.2641 |
| 1680 MHz / +210 | 40.05 | 184.4 | 0.2172 | 40.21 \* | 162.5 | 0.2474 |
| 1680 MHz / +240 | 40.13 \* | 182.1 | 0.2204 | 40.31 | 158.6 | 0.2542 |
| 1740 MHz / +210 | 41.29 | 199.6 | 0.2069 | 41.48 \* | 178.2 | 0.2328 |

\* only 1-3 rate lines on that slot in the window. At 1740 MHz Kepler reached the 200 W limit and the clock fell to
1710-1731 MHz. Every point ran stable with no errors. The best measured efficiency is at the lower lock: 1500 MHz / +210
(Kepler 0.2460, TESS 0.2760 MA/W), about 13 % more MA/W on Kepler than 1680 MHz / +210 at about 10 % less speed.
This is one card and short windows, not a recommendation: the miner's settings (auto-lock, auto-uv, OC) are unchanged.

**GPU engine 0.4.0 (ffgn 1.0.7)**, engine benchmark on the chain's trained models (20.11.2026, auto kernel choice):

| GPU | slot | 1.0.6, MA/s | 1.0.7, MA/s |
|---|---|---:|---:|
| RTX 3070 | Kepler | 19.33 | 31.42 |
| RTX 3070 | TESS | 25.93 | 37.92 |
| RTX 4070 Laptop GPU (hot, two 8 s rounds) | Kepler | — | 36.02-38.00 |
| RTX 4070 Laptop GPU (hot, two 8 s rounds) | TESS | — | 35.34-38.28 |

The RTX 3070 comparison uses nearby core locks (1650 MHz for 1.0.6, 1680 MHz for 1.0.7) and a 200 W limit. RTX 4070 Laptop
GPU power varied with temperature, so no efficiency claim is made from that A/B. RTX 40 desktop and RTX 50 cards were not
measured by us with 0.4.0 yet.

TESS on RTX 3070, engine 0.4.0 clock sweep:

| Core lock | Speed | Mean power | Efficiency |
|---|---:|---:|---:|
| **1680 MHz (best measured efficiency)** | 36.81 MA/s | 135.7 W | **0.271 MA/W** |
| 1750 MHz | 38.33 MA/s | 148.2 W | 0.259 MA/W |
| 1800 MHz | 39.27 MA/s | 157.2 W | 0.250 MA/W |
| **1850 MHz (maximum measured speed)** | **40.49 MA/s** | 170.4 W | 0.238 MA/W |

These are measurements, not a recommendation to raise your clocks. Auto-lock, auto-uv and existing OC settings are unchanged.

**GPU engine 0.3.0 (ffgn 1.0.6)**, engine benchmark on the chain's trained models (19.11.2026, 8 s x 2 rounds, auto choice):

| GPU | slot | 1.0.5, MA/s | 1.0.6, MA/s | 1.0.6, W | 1.0.6, MA/W | kernel |
|---|---|---|---|---|---|---|
| RTX 3070 (core lock 1650 MHz, power limit 200 W) | Kepler | 16.1-16.5 | 19.33 | 160 | 0.121 | warp |
| RTX 3070 (core lock 1650 MHz, power limit 200 W) | TESS | 20.17 | 25.93 | 172 | 0.151 | warp |
| RTX 4070 Laptop GPU (hot, 86 C, ~45-50 W) | Kepler | 29.7 | 32.7 | ~51 | 0.64 | warp |
| RTX 4070 Laptop GPU (hot, 86 C, ~45-50 W) | TESS | 26.2 | 30.1 | ~45 | 0.67 | warp |

RTX 3070: 1.0.5 (engine 0.2.9) measured both kernels once, on the first model of a slot, and kept dp4a or warp for good; on the
chain's models of today both read every row and the card is limited by its memory. 1.0.6 skips the zero rows and picks warp on
both slots. RTX 4070 Laptop GPU: the 1.0.5 column is engine 0.3.0 with the zero rows read (`NEBGPU_DEADSKIP=0`, the same kernels
as 0.2.9). RTX 40 desktop and RTX 50 cards were not measured by us with 0.3.0 yet.

**GPU engine 0.2.9 (ffgn 1.0.5): both KEP1 kernels**, engine benchmark, the same conditions for both, measured 19.11.2026:

| GPU | slot | dp4a kernel, MA/s | warp kernel, MA/s | W | dp4a, MA/W | warp, MA/W | auto picks |
|---|---|---|---|---|---|---|---|
| RTX 4070 Laptop GPU (cool, at its ~115 W cap) | Kepler | 20.8 | 33.4 | ~115 | 0.18 | 0.29 | warp |
| RTX 4070 Laptop GPU (cool, at its ~115 W cap) | TESS | 20.7 | 33.5 | ~115 | 0.18 | 0.29 | warp |
| RTX 4070 Laptop GPU (hot, 88 C, ~64 W) | Kepler | 14.97 | 23.61 | 64 | 0.232 | 0.368 | warp |
| RTX 3070 (power limit 200 W, stock clocks) | Kepler | 16.73 | 16.56 | 200 | 0.084 | 0.083 | dp4a |
| RTX 3070 (power limit 200 W, stock clocks) | TESS | 17.75 | 20.25 | 200 | 0.089 | 0.101 | warp (initial model); dp4a on the chain's models |

On the RTX 3070 the rate is set by the memory (its 4 MB cache cannot hold the tables): the warp kernel gains on the smaller
TESS slot with the initial model only; on the trained models of the chain (19.11.2026, mining on Suprnova) its own timing gave
Kepler dp4a 17.34 vs warp 16.87 and TESS dp4a 20.65 vs warp 19.76, so it mines with the old kernel (16.8 / 18.4 MA/s at 200 W,
as 1.0.4). RTX 4070 Laptop GPU on the same pool: warp 30.7 vs dp4a 18.9 MA/s on Kepler and 23.5 vs 18.6 on TESS, all shares
accepted. RTX 40 desktop and RTX 50 cards were not measured by us with 0.2.9 yet.

**RTX 3070, GPU engine 0.2.8** (core lock, power limit 200 W, memory at its stock 7001 MHz, no undervolt, the real dataset;
speed in million attempts per second, efficiency in million attempts per second per watt):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W |
|---|---|---|---|
| stock (limited by the 200 W) | 16.95 | 200 | 0.085 |
| 1800 | 16.96 | 200 | 0.085 |
| 1650 | 16.92 | 193 | 0.088 |
| 1500 | 15.55 | 160 | 0.097 |
| 1350 | 15.06 | 141 | 0.107 |
| 1200 | 14.24 | 130 | 0.110 |

**RTX 4070 Ti, GPU engine 0.2.8** (stock power limit 235 W, memory at its stock 10251 MHz, no undervolt, the real dataset):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W |
|---|---|---|---|
| stock (~2450, limited by the 235 W) | 37.59 | 235 | 0.160 |
| 2500 | 37.31 | 235 | 0.159 |
| 2250 | 34.44 | 190 | 0.181 |
| 2000 | 30.89 | 173 | 0.179 |
| 1800 | 27.72 | 156 | 0.177 |

On the RTX 3070 the best efficiency is near 1350 MHz and 1500 MHz is a balance (about 8 % less speed for 20 % less power); on
the RTX 4070 Ti 2250 MHz is the balance (about 4 % less speed for 10-19 % less power). Every main-network block is KEP1, so
the speed of a run is the KEP1 figure of the table. The numbers are the engine benchmark on the real main-network dataset and test runs (the mining loop gives the same speed);
your card, cooling and silicon will give somewhat different numbers.

### Memory clock

NEB reads its tables from the video memory all the time: the hash rate follows the memory clock. A memory clock locked low (the
HiveOS OC profiles made for Pearl lock it to 810 MHz; MSI Afterburner profiles with a memory underclock) makes NEB several
times slower. Measured with engine 0.2.8 (RET2 figures are for the test network only): RTX 3070 at 5001 MHz instead of 7001 — KEP1 12.55 / RET2 7.40 MA/s (−26 % / −25 %),
at 810 MHz — 1.64 / 0.97 MA/s (−90 %); RTX 4070 Ti at 5001 MHz — RET2 8.85 MA/s (−50 %; KEP1 stays near 37.7, its table fits in
the cache), at 810 MHz — RET2 1.18 MA/s (−93 %). ffgn therefore:

- replaces a HiveOS OC profile memory lock below the card's maximum with the maximum while it mines, and puts the profile's
  value back on exit (`--keep-hive-mem` keeps the profile's value; `--lock-mclock N` sets your own);
- writes a WARNING in the log after the first minute when a card's memory runs below 60 % of its maximum.
- 1.0.4: when the driver does not let ffgn lift the profile's lock (RTX 2060 SUPER, driver 620: "memory clock lock not
  supported") and the memory runs at that lock under load, every stats line, the API (`warnings`) and the HiveOS version field
  say `NEB runs ~6x slower: memory locked to 810 MHz by the OC profile — remove the memory lock`: remove the memory lock (and
  the memory clock) from the OC profile.

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
- **Lost GPU** (1.0.12). A GPU that fell off the PCIe bus (Xid 79), whose engine cannot reopen the device after the retries
  (`DEAD` on the stats line) or that NVML no longer sees is LOST: it shows `0` and `LOST` / `DEAD` on the stats line, `lost: true`
  in the API `/gpus` (its average is 0 as well), and `GPU LOST GPU1` next to the version in HiveOS, so the total is the true one.
  The NVIDIA driver gets such a card back only after a reboot. `--reboot-on-lost-gpu MIN` (off by default; recommended on rigs:
  `10`) reboots the rig when a GPU stays lost for MIN minutes: HiveOS `sreboot` (`/hive/sbin/sreboot`), else `systemctl reboot`;
  Windows `shutdown /r /t 60 /c "FFGN: GPU lost"` (`shutdown /a` cancels it within the 60 s). Never in the first 10 minutes of
  the miner and at most once per 60 minutes (the time is kept in `reboot-on-lost-gpu.json` in the state folder, across reboots:
  a card that is still lost after the reboot does not make the rig reboot in a loop); a `WARNING: REBOOTING THE RIG: GPU1 (bus 02)
  lost for 10.0 min (...); running ...` line comes first, also in the API warnings. With `--single-process` or the environment
  variable `FFGN_NO_REBOOT=1` it never reboots, it only logs what it would do. Needs root / Administrator (HiveOS: root).
- **Never standing still** (1.0.12). When every GPU is down or not mining for 5 minutes although there is work (the driver is
  gone, every engine fails), the miner exits with code **3**: HiveOS (its miner restart and Watchdog), `start.ps1` and systemd
  start it again. A pool or node that is down, a pause (idle mode, an auto-uv check, a temperature stop) does not count.
- **HiveOS Watchdog (recommended).** Rig settings, *Watchdog*: enable it, *Minimal hashrate* about 50 % of the rig's normal
  total for `nebhash` (e.g. 120 MA/s normal -> 60000 kH/s), *Restart miner after* 10-15 minutes, *Reboot after* 20-30 minutes;
  plus `--reboot-on-lost-gpu 10` in Extra config. A lost GPU counts 0, so the rule sees the real hashrate.
- Windows `start.ps1` restarts a miner that exited after 15 seconds; HiveOS and systemd restart it too.

## 16. Reading the log, the API, the balance

Every 60 seconds one line (pool mode):

```
[  60s] GPU0 11.9 (139 W, 0.0856 MA/W)  GPU1 ... | total 23.8 MA/s, 278 W, 0.0856 MA/W | found 3 (bad 0) | user sub 3 acc 3 rej 0 (stale 0) | ...
```

1.0.4: after `total` come the average of the last 5 minutes (`| avg 5 min 23.9 MA/s (11.9, 12.0)`, total and per GPU; HiveOS shows
this average) and the dataset of the current block (`| slot 1 TESS`: on the main network slot 0 is Kepler, slot 1 TESS; the
rate differs by slot, an RTX 3070 @1650 ~16.9 vs ~18.6 MA/s, so a slot change is not a regression). API `/summary`:
`attempts_per_s_avg`, `slot`, `dataset`; `/gpus`: `attempts_per_s_avg`, `memory_locked_by_oc_mhz`, `power_limit_w` (the limit the card is held to; with `--power-limit` also `power_limit_ours`, `power_limit_set_w`, `power_limit_binds`). Solo: `... | shares 3 | blocks sub 0 acc 0 rej 0 | ...`. **MA/s** = million attempts per second; `found` = what the GPUs found (each
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

- **Pool mode.** Shortly before the slot the miner opens a **second connection** to the same pool with the fee login (worker
  name `ffgn`) and mines the slot there; your own connection and login are never touched. If it does not work, the slot is
  skipped: you are never blocked. 1.0.3: Suprnova share accounting after the dev-fee slot fixed (the fee always uses a
  separate connection); Suprnova users must update.
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
| `dataset ... BLAKE3` error | `neb-datasets.bin` is damaged or is not the NEB 1.0.1 main-network file (the 222 MB file of NEB 1.0.0 is refused by ffgn 1.0.2). Unpack the package again and compare the SHA-256 (section 4). |
| `--self-test` fails or GPU errors | Update the NVIDIA driver; check cooling and power; try without clock and undervolt options (`--uv-reset-all` forgets old profiles). |
| No accepted shares | Check the pool address and port, the address (`neb1...`), and that the pool is on the main network. `ffgn --pool-check --pool HOST:PORT` tests the login. On Aria look at the pool's page: Aria does not answer shares one by one. Wait a few minutes: the share difficulty adapts to your speed. |
| Many stale / rejected shares | The clock is wrong (turn on time synchronisation: Windows `w32tm /resync`, Linux `timedatectl`), or the connection to the pool is slow or lossy. A few stale shares right after a new block are normal. |
| The speed jumps up and down | On the main network every block is KEP1, so the speed should be steady. Short dips come from the pool (new jobs), the clock or thermal limits. |
| The speed is lower than in the table | Thermal or power limit, a lower clock lock, a laptop, another program on the GPU. Look at `--gpu-state` and the log (`--temp-limit`, idle mode). |
| The miner restarts itself | The message before the restart says why: a GPU fault (the undervolt offset is stepped down), a dead link, or the watchdog. Repeated GPU faults: use `--uv-reset-all` and a lower `--uv-max`, or no undervolt. |
| Solo: the miner gets no job | The node must run with `--miner-lock` on the **same** computer (port 3441 is local), and it must be synchronised. Before 14:00 UTC on 18 November the first job comes about two minutes before the start. |
| Solo: the node stays at height 0 | Normal before the start. After it: check the clock, the outgoing port 3440 and the genesis id (section 10). |
| Solo: valid blocks rejected | The clock of the node is wrong (blocks dated more than 120 s ahead are refused). |
| Balance is 0 / "immature" | Rewards mature after 100 blocks; a solo node must be synchronised. |
| HiveOS does not start the miner | The Installation URL of the right release; algorithm `nebhash`; wallet template `%WAL%.%WORKER_NAME%`; read `miner log`. |
| Windows: clocks / undervolt not applied | Start the `.bat` with Run as administrator. |
| Antivirus warning | Mining software is often flagged. Compare the SHA-256 with `SHA256SUMS`, then add an exclusion if you decide to trust it. |
| One GPU restarts by itself (`FAILED in its own engine process`) | Since 1.0.11 each GPU runs in its own engine process: a fault on one GPU (usually an overclock or undervolt too far for that card) restarts only that GPU in about 5-10 s, the others keep mining. The log names the GPU with the driver fault: lower that card's offset or use `--auto-uv`. With `--single-process` (or before 1.0.11) a fault restarts the miner on every GPU. |

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
