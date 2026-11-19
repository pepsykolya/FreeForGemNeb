# FreeForGemNeb (ffgn) 1.0.6

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

**What changed in 1.0.6** (recommended for everyone). GPU engine 0.3.0: on the trained models of the chain many samples already give the target output (their error is 0), so their table rows are all zeros; both KEP1 kernels no longer read them. The results are the same bit for bit (NEB test vectors, the CPU reference on every share). The KEP1 kernel (dp4a or warp) is now chosen again at every new model, timed on its first mining launches (both kernels give the same results, so no attempt is lost); the log line `GPU0: KEP1 kernel ...` appears when the choice changes, the API field `kep1_kernel` shows the latest one. Measured (engine benchmark, the chain's models): RTX 3070 at a 1650 MHz lock and a 200 W limit: Kepler 16.5 -> 19.3 MA/s (+17 %), TESS 20.2 -> 25.9 MA/s (+29 %), it now mines with the warp kernel on both slots; RTX 4070 Laptop GPU: +8 to +15 %.

**What changed in 1.0.5** (recommended for everyone). **A new KEP1 kernel for RTX 30 / 40 / 50** (and RTX A-series): one attempt per GPU warp on the int8 tensor cores. The miner times both kernels on the first model of each dataset slot and keeps the faster one for that slot (`--kep1-kernel auto|dp4a|warp`; the log line `GPU0: KEP1 kernel ...` and the API field `kep1_kernel` show the choice). The results are the same bit for bit (NEB test vectors, the CPU reference on every share). Measured (engine benchmark, the real dataset): RTX 4070 Laptop GPU about 1.6x on both slots (Kepler 20.8 -> 33.4, TESS 20.7 -> 33.5 MA/s); RTX 3070 (its rate is set by the memory): no change, the miner keeps the old kernel there (on the initial model the new one was faster on TESS, 17.8 -> 20.3, but on the chain's trained models it is not: 19.8 vs 20.7). RTX 40 desktop and RTX 50 cards were not measured by us yet. `--auto-uv` is safe on big rigs with a weak CPU or power: it starts with 1 GPU at a time on 6+ GPUs (else 2) and grows to 2 and 4 only while the rig stays healthy (`--uv-parallel N` to fix it; the other GPUs mine meanwhile), every step that lowered the power is saved at once, and guards stop endless tuning: 3 failures in a row turn auto-uv off for a GPU for 24 h, 2 restarts or reboots during a tuning within an hour pause it on the rig (`--uv-unpause`), at most 10 min per GPU and 60 min per run, and after an unclean end the rig mines 7 min first. The power of a step is averaged after 10 s under load (25 s steps), a step that did not clearly lower it is measured again in the order best-step-best-step, and the log shows the spread (sd) of every measurement. Rigs of 4+ GPUs: after a pause the GPUs return to full load one by one (`--ramp-ms`, 500 ms apart), a new block's model is built 30 ms apart per GPU, and when all GPUs stop at the same moment the log says so (`all N GPUs stalled at once ...: likely a PCIe bus / power event`).

**What changed in 1.0.4**. `--auto-uv` climbs as long as every step lowers the power: a step is taken when its power is lower than the step before by more than the noise of a measurement (0.5 W), a step that does not beat the best power is not taken but the climb goes on (single readings move by 1-2 W), the climb stops when 3 steps in a row do not lower the power or it rises by more than 3 W, and the result is the highest stable offset that still lowered the power; no percentage thresholds any more (1.0.3 stopped an RTX 2060 SUPER at +165 although +195 and +225 saved 0.6 and 0.9 W each). Profiles saved by 1.0.3 are not tuned from +0: the card checks its saved offset and climbs on from it. The stats line, the API and HiveOS show the dataset of the current block (`slot 0 Kepler` / `slot 1 TESS`: the rate differs by slot) and a 5-minute average rate. A core lock that does not hold turns off auto-lock for that GPU too (not only auto-uv); a step whose clock runs above the lock gets the lock set again and is checked once more before it is refused. A gate or miner keeps its link to a node that is silent but alive (before the network starts). A memory lock of the OC profile that the driver does not let ffgn lift (RTX 2060 SUPER, driver 620) is a loud warning on every stats line, in the API and in HiveOS.

**What changed in 1.0.3** (for the NEB 1.0.1 main network). **Suprnova: fixed share accounting after the dev-fee slot (the dev fee now always uses a separate connection). Suprnova users must update.** Much less CPU: the engine waits for the GPU asleep (before, a whole CPU core per GPU; a 12-GPU rig on a 2-core CPU was at 100 %). `--auto-uv`: every check reads the real core clock: a step that runs above the core lock is refused (GeForce Turing), a lock dropped by an OC apply is set again, a lock that does not hold is reported loudly and auto-uv is off for that GPU; a step is taken only if the power did not rise, the result is saved only if it saves at least 3 %, at the lowest offset that gives it; after a crash the card resumes 2 steps below it, not from +0. `--password` goes only to the first pool or to an ffgn gate (backup pools get `x`). `--pool-check` checks the network of the job.

**What changed in 1.0.2.** ffgn 1.0.2 is required for the NEB 1.0.1 main network; ffgn 1.0.1 does not work on it. NEB 1.0.1
replaced NEB 1.0.0 before the start: new genesis id, a new dataset file `neb-datasets.bin` (about 16 MB instead of 222 MB; it is
inside every package) and both slots of the main network run KEP1, so every block has the KEP1 speed (RET2 is gone from the
main network). The pools (Aria, Suprnova) must run NEB 1.0.1 too; ffgn checks the pool's dataset hash and fails over to the
next pool if it differs. The GPU engine is unchanged (0.2.8), the dev fee is still 1 %.

The NEB main network starts on **18 November 2026, 14:00 UTC**. The miner makes no promise about the value of the coin or
about any income.

## Download

Release page: https://github.com/pepsykolya/FreeForGemNeb/releases/tag/v1.0.6

| package | file |
|---|---|
| HiveOS (custom miner `ffgn-hive`) | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.6/ffgn-hive-1.0.6.tar.gz |
| Linux x86_64 | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.6/ffgn-1.0.6-linux-x86_64.tar.gz |
| Windows x64 | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.6/ffgn-1.0.6-windows-x64.zip |
| checksums | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.6/SHA256SUMS |

The miner binary is `ffgn` (`ffgn.exe` on Windows). The dataset file of the NEB main network (`neb-datasets.bin`, about 16 MB) is
inside every package.

## Speed and power

Million attempts per second (MA/s) and million attempts per second per watt (MA/W), the real dataset, memory at its stock
clock, no undervolt. On the main network every block is KEP1 (slot 0 Kepler, slot 1 TESS: two datasets of different size).

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

**RTX 3070, engine 0.2.8 (ffgn 1.0.4)** (power limit 200 W):

| Core lock, MHz | KEP1, MA/s | KEP1, W | KEP1, MA/W |
|---|---|---|---|
| stock (limited by the 200 W) | 16.95 | 200 | 0.085 |
| 1800 | 16.96 | 200 | 0.085 |
| 1650 | 16.92 | 193 | 0.088 |
| 1500 | 15.55 | 160 | 0.097 |
| 1350 | 15.06 | 141 | 0.107 |
| 1200 | 14.24 | 130 | 0.110 |

**RTX 4070 Ti, engine 0.2.8 (ffgn 1.0.4; not measured with 0.2.9 yet)** (power limit 235 W):

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
   `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.6/ffgn-hive-1.0.6.tar.gz`, algorithm `nebhash`, wallet
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
