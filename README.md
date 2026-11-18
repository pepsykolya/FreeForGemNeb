# FreeForGemNeb (ffgn) 1.0.3

Native GPU miner for **NEB** (Neural Engine Byte), a proof-of-work coin whose mining attempt is one integer training step of a
small neural network (on the main network, the Kepler transit-signal network KEP1 in both dataset slots).
NVIDIA GPUs; HiveOS, Windows and Linux packages. Dev fee **1%**. Русская версия — [README.ru.md](README.ru.md).

- Every share and block found by the GPU is computed again on the CPU with the reference code of NEB before it is sent;
  new models are checked on the device, one batch in 64 is audited.
- Mining through a pool (Aria, Suprnova) or solo through your own NEB node; **ffgn-gate** lets rigs mine solo through one node.
- Built-in auto-undervolt checked bit for bit against the reference, `--auto-lock` that picks the core clock lock of each
  GPU by measurement, clock locks, temperature limit for laptops, idle mode on Windows.
- Pool failover with the return to the primary pool, watchdogs for pools without credited shares, API with Prometheus
  metrics, your GPU settings put back after every kind of stop, Windows autostart, HiveOS stats, a notice when a new version is out.

**What changed in 1.0.3** (for the NEB 1.0.1 main network). **Suprnova: fixed share accounting after the dev-fee slot (the dev fee now always uses a separate connection). Suprnova users must update.** Much less CPU: the engine waits for the GPU asleep (before, a whole CPU core per GPU; a 12-GPU rig on a 2-core CPU was at 100 %). `--auto-uv`: every check reads the real core clock: a step that runs above the core lock is refused (GeForce Turing), a lock dropped by an OC apply is set again, a lock that does not hold is reported loudly and auto-uv is off for that GPU; a step is taken only if the power did not rise, the result is saved only if it saves at least 3 %, at the lowest offset that gives it; after a crash the card resumes 2 steps below it, not from +0. `--password` goes only to the first pool or to an ffgn gate (backup pools get `x`). `--pool-check` checks the network of the job.

**What changed in 1.0.2.** ffgn 1.0.2 is required for the NEB 1.0.1 main network; ffgn 1.0.1 does not work on it. NEB 1.0.1
replaced NEB 1.0.0 before the start: new genesis id, a new dataset file `neb-datasets.bin` (about 16 MB instead of 222 MB; it is
inside every package) and both slots of the main network run KEP1, so every block has the KEP1 speed (RET2 is gone from the
main network). The pools (Aria, Suprnova) must run NEB 1.0.1 too; ffgn checks the pool's dataset hash and fails over to the
next pool if it differs. The GPU engine is unchanged (0.2.8), the dev fee is still 1 %.

The NEB main network starts on **18 November 2026, 14:00 UTC**. The miner makes no promise about the value of the coin or
about any income.

## Download

Release page: https://github.com/pepsykolya/FreeForGemNeb/releases/tag/v1.0.3

| package | file |
|---|---|
| HiveOS (custom miner `ffgn-hive`) | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.3/ffgn-hive-1.0.3.tar.gz |
| Linux x86_64 | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.3/ffgn-1.0.3-linux-x86_64.tar.gz |
| Windows x64 | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.3/ffgn-1.0.3-windows-x64.zip |
| checksums | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.3/SHA256SUMS |

The miner binary is `ffgn` (`ffgn.exe` on Windows). The dataset file of the NEB main network (`neb-datasets.bin`, about 16 MB) is
inside every package.

## Speed and power

GPU engine 0.2.8, the real dataset, memory at its stock clock, no undervolt; million attempts per second (MA/s) and million
attempts per second per watt (MA/W). On the main network every block is KEP1, so the KEP1 figures are the ones that count.

**RTX 3070** (power limit 200 W):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W |
|---|---|---|---|
| stock (limited by the 200 W) | 16.95 | 200 | 0.085 |
| 1800 | 16.96 | 200 | 0.085 |
| 1650 | 16.92 | 193 | 0.088 |
| 1500 | 15.55 | 160 | 0.097 |
| 1350 | 15.06 | 141 | 0.107 |
| 1200 | 14.24 | 130 | 0.110 |

**RTX 4070 Ti** (power limit 235 W):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W |
|---|---|---|---|
| stock (~2450, limited by the 235 W) | 37.59 | 235 | 0.160 |
| 2500 | 37.31 | 235 | 0.159 |
| 2250 | 34.44 | 190 | 0.181 |
| 2000 | 30.89 | 173 | 0.179 |
| 1800 | 27.72 | 156 | 0.177 |

Balance points (KEP1): RTX 3070 1500 MHz, RTX 4070 Ti 2250 MHz. The memory clock matters: a memory locked to 810 MHz (HiveOS profiles
made for Pearl) makes NEB about 10x slower; ffgn replaces such a lock while it mines (see the
[manual](MANUAL.md#13-clocks-undervolt-auto-lock-speed-and-watts)).

How it was measured: the engine benchmark on the real NEB main-network dataset (a model at its initial weights), each point
after the clocks settled, speeds and watts from the driver (NVML). Your card, cooling and silicon will give somewhat different
numbers; the speed in a long run also depends on how far the chain's models have trained.

## Quick start

Full step-by-step guide for beginners (wallet, Windows, HiveOS, Linux, pools, solo, options, troubleshooting):
**[MANUAL.md](MANUAL.md)** (Русский: [MANUAL.ru.md](MANUAL.ru.md)) · HiveOS flight sheet with a ready one to import:
[HIVEOS.md](HIVEOS.md).

1. Create a wallet with the official `neb-wallet` (part of the NEB release, check its hashes and signature first) and keep the
   file and the password (there is no seed phrase): `neb-wallet --wallet wallet.txt new`. It prints your `neb1...` address.
2. Windows: unpack the zip into a folder without spaces, edit `start.bat` (right-click, Edit): set `WALLET`, run it. It mines on
   pool Aria with Suprnova as the backup; `start-suprnova.bat` swaps them; `start-solo.bat` is for your own node.
3. Linux:
   `./ffgn --pool neb.ariabrain.com:3370 --wallet neb1YOURADDRESS --worker rig1`
4. HiveOS: custom miner `ffgn-hive`, Installation URL
   `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.3/ffgn-hive-1.0.3.tar.gz`, algorithm `nebhash`, wallet
   template `%WAL%.%WORKER_NAME%`, Pool URL `neb.ariabrain.com:3370`.
5. Solo through your own node: `neb-node --run --dataset neb-datasets.bin --datadir data --miner-lock "$(neb-wallet --wallet wallet.txt lock)"`
   and `ffgn --node 127.0.0.1:3441` on the same computer (details in the manual).

## Requirements

- NVIDIA GPU, compute capability 6.1 or newer (GTX 10 up to RTX 50); tested on RTX 30 and RTX 40; the miner needs less than 1 GB
  of video memory.
- NVIDIA driver with CUDA 12 support (R525 or newer; RTX 50: R570 or newer). No CUDA toolkit is needed.
- Windows 10/11 x64, Linux x86_64 (glibc 2.30 or newer) or HiveOS.
- The correct clock of the computer (a NEB node refuses blocks dated more than 120 s ahead of its clock).
- Clock locks and undervolt need administrator (Windows) or root (Linux, HiveOS).

## Main network parameters

| | |
|---|---|
| start | 18 November 2026, 14:00 UTC |
| block time | 60 s (ASERT) |
| reward | 20 NEB per block, halving every 450,000 blocks; 5 % of each reward goes to the NEB project |
| supply | about 18 million NEB; the genesis block also pays a tenth of the test-network balances |
| maturity | mined coins are spendable after 100 blocks |
| ports | peers 3440; the miner port 3441 and the RPC port 8432 are local only |
| genesis id | `da2f5af7bb4c5e2bcc8d558b2f4f01d4c88557a943e6cff9cd035638fe9439ae` |
| dataset | `neb-datasets.bin`, 16,087,037 bytes, SHA-256 `57e22a4b8aacecb2ebae62f76307ddccb04198ed5c4ba15c1dee90ca7a7c2174` |

More: https://github.com/SilasEigen/neb (code, specification, release), https://neuralenginebyte.org .

## Pools

| Pool | Address | Mode |
|---|---|---|
| Aria | `neb.ariabrain.com:3370` | PPLNS |
| Aria, solo | `neb.ariabrain.com:3371` | solo |
| Suprnova | `neb.suprnova.cc:7804` | PPLNS |
| Suprnova, solo | `neb.suprnova.cc:7805` | solo |

Several pools in one command = failover (`--pool A,B`). Fees and payout rules are the pools'; check on the pool's page that it
works on the main network.

## Dev fee

1 % of the mining time: one 60-second slot in every 100 minutes, at a random moment. In the slot the miner works for the
developers' login on the same pool (solo: on a public pool, because the node pays its own address) and then returns to yours;
the log shows each slot. It cannot be removed or changed (see [LICENSE](LICENSE)); how it works: MANUAL section 17.

## What is in the package

`ffgn` / `ffgn.exe` (host program), `libnebgpu.so` / `nebgpu.dll` (GPU engine, binary only), `neb-datasets.bin`, start scripts
(HiveOS `h-*.sh`, Windows `start*.bat`, `start.ps1`, autostart scripts), README, MANUAL, LICENSE, NOTICE (HiveOS package: also
HIVEOS.md). The NEB node and wallet come from the official NEB release (MANUAL section 10).

## License

Free to use and redistribute in unmodified form; the 1 % dev fee must stay intact. See [LICENSE](LICENSE); third-party
components and data: [NOTICE](NOTICE).
