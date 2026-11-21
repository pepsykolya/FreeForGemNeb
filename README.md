# FreeForGemNeb (ffgn) 1.0.13

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

**What changed in 1.0.13** (recommended for everyone). **1.0.12 was withdrawn; please use 1.0.13.** 1.0.12 had a bug: after an auto-uv fault a healthy GPU could stay DEAD, and 1.0.13 fixes it. Everything from 1.0.12 is in 1.0.13 (below). GPU engine 0.8.1, unchanged. **GPU fault recovery:** after a GPU fault the miner now brings the card to a safe state first and starts the new engine only then. The safe state is the run's core lock, memory policy and power limit, and an offset two steps below the one that crashed (never above the last confirmed one, never below the base). The driver's reset is waited out in a separate probe process, so the miner itself is not exposed to it. A healthy card is no longer marked DEAD because the new engine started while the faulty offset was still applied; DEAD / LOST now means the device cannot be opened or reached at all. A card that keeps faulting at a safe offset is paused for 10 minutes, then gets a new engine, and no reboot is asked. A busy memory-lock answer no longer switches `--table-mclock` off. **Late pool answers:** the miner prints a WARNING when the pool's answers come seconds late (a lossy or slow network link makes shares stale); it shows the median delay and the stale shares so far. **New option `--debug-timing`**: logs per share the time from the scan to the CPU check to the socket, and per job the time from the socket to the dispatch, to diagnose stale shares. **Fewer stale shares:** the CPU check runs off the GPU and pool path, hits of a job replaced by a new clean job are dropped at once, and after "job not found" the rest of that job is not sent. Results, models, tickets and the dataset are unchanged bit for bit; the CPU reference still re-checks every share.

**What changed in 1.0.12** (withdrawn; all of it is in 1.0.13). GPU engine 0.8.1. **Partial L=3 table for TESS, automatic:** the digest table of the TESS slot can now also hold attempts with three live rows, as many as fit in the free VRAM. The miner builds them in slices while the pair table already mines and sizes them per model from two measured numbers: the card's build speed and how long TESS models last. When the models change too quickly for the build to pay back, it builds none. With the triples built, the TESS rate rises by +17 to +18 % on an RTX 4070 Ti or RTX 3080 Ti (12 GB) and by +9.5 % on an RTX 3070 (8 GB), at the same power. `--tbig off|auto|on`, `--tbig-gb G`. **New option `--power-limit N` (`--pl N`)**: one value or one per GPU, put back on exit and set again after an OC apply. **New option `--reboot-on-lost-gpu MIN`** (off by default): reboots the rig when a GPU stays lost (fallen off the bus) for MIN minutes; and when every GPU is down for 5 min the miner exits with code 3, so the HiveOS watchdog can act; a lost GPU reports 0. **auto-uv:** no 24 h pause without a real verdict (in 1.0.11 one card's fault could pause the others at +0); after a restart a card with recent crashes resumes below its saved offset and climbs again; after an Xid 79 the whole rig tunes cautiously for 6 h; RTX 20 / GTX 16 climb at most to +240 by default (`--uv-max` overrides). **The digest table no longer stays off after one slow measurement** (for example during a driver recovery on another GPU). The path line says why a GPU mines without the table. **Fixed (Windows):** when an undervolt step went too far and the driver had to recover the card, the miner itself could close a moment later. Now the miner pauses its NVML calls for that GPU until its new engine process mines again, restarts only that GPU and steps the offset down as usual. Results, models, tickets and the dataset are unchanged bit for bit; the CPU reference still re-checks every share.

**What changed in 1.0.11** (recommended for everyone; RTX 20 / GTX 16 / CMP Turing and RTX 30 / 40). GPU engine 0.7.0. **Automatic L=3 on low-P models:** when the chain's Kepler model has few attempts with at most two live rows (below 1.3 %), the engine now also evaluates cheap attempts with three live rows from the digest table (exact, by inclusion-exclusion) and decides by a short trial per model. On such a model: RTX 3070 +28 % rate and +18 % per watt (60.0 against 47.0 MA/s), CMP 50HX (Turing) +15 % rate and +13 to +15 % per watt. Models above 1.3 % are unchanged. **Faster table build:** the pair section of the digest table is built 3.1 to 4.2 times faster (byte-identical), so each new Kepler model costs about 0.7 s less mining (RTX 3070: 1.0-1.2 s -> 0.31-0.34 s; CMP 50HX: 0.9-1.1 s -> 0.29-0.32 s). **Faster screen on RTX 30 / 40** (sm_86 / sm_89): +1.0 % (Kepler) and +2.25 % (TESS) rate on an RTX 3070. **New option `--table-mclock N`** (e.g. 810): the memory is locked to N MHz only while a GPU mines on the pure digest-table path, which barely uses the memory; the miner switches the memory back to the full clock by itself for every table build, the L=3 path and every other path. It only helps the pure table path: there an RTX 3070 drew about 10 W less at the same rate. Needs root / Administrator; off by default. **One engine process per GPU:** a fault on one GPU restarts only that GPU (about 5-10 s), the other GPUs keep mining; `--single-process` restores the old behaviour; about 60 MB extra RAM per GPU. **Fixed:** in 1.0.10 a start (or restart) on a low-P model could switch the digest table off after 3 s and mine on a slower path until the next model (RTX 3070: 40 MA/s at 173-181 W instead of about 47 MA/s at 127 W). auto-uv now compares the power of a step only on the same model and path, including its memory clock. Engine hardening for big rigs (an ordering fix of two counters, a check of the table's row map before upload). Results, models, tickets and the dataset are unchanged bit for bit; the CPU reference still re-checks every share.

**What changed in 1.0.10** (recommended for everyone; faster on RTX 20 / GTX 16 / CMP Turing cards and on RTX 30). GPU engine 0.6.1 has a faster nonce screen on the digest-table path (the step that picks out the attempts with at most two live rows before they are looked up in the model's table). It is on by default on every card that takes the table path (CC 7.5 and newer: RTX 20 / GTX 16 / CMP Turing, RTX 30 / 40 / 50); GTX 10 and Volta keep the full path as before. Everything else is as in 1.0.9: the engine still chooses the path for every model by measurement, results, tickets, models and the dataset are unchanged bit for bit and the CPU reference still re-checks every share. Engine benchmark on the same recorded live models (1.0.9 engine vs 1.0.10 engine in one process): RTX 3070 at 1680 MHz / +210 / 200 W +28.5 to +28.9 % rate and +8.6 to +11.5 % per watt; CMP 50HX (Turing) at a 1300 MHz core lock +15 to +21 % rate and +10 to +14 % per watt. Live mining on the same RTX 3070: +25 % (Kepler) and +27 % (TESS) screened nonces per second, +4.8 % and +7.6 % per watt; five CMP 50HX at 1300 MHz: +13 % (Kepler) and +3 % (TESS), +6.1 % and +1.6 % per watt. The faster screen keeps the GPU busier, so the card draws more power (about +17 % on the RTX 3070 in the benchmark) and the gain per watt is smaller than the gain in rate. **auto-uv tunes while mining.** A card under `--auto-uv` now keeps mining on the path the engine chose, sends shares and counts its MA/s while its steps are measured (in 1.0.9 a card under tuning stood at 0 MA/s for its whole tuning). All GPUs are tuned at once. Each step's power is compared with the best offset so far on the same model and path, and a step ends as soon as the difference is clear. Measured on an RTX 3070: a fresh tuning took 120 s including one fault and an automatic restart of the miner (about 17 s per step); a start from a saved profile needs one 17-s check. The card ended at +240 with 124-126 W against 129-130 W at a fixed +210, at the same rate (about -3 %). A hard Xid at the edge costs one self-restart of the miner of about 10 s. A whole rig tunes in roughly 2-5 min, all cards in parallel (estimate). The rules are unchanged: climb while every step lowers the power by more than 0.5 W, resume 2 steps below a failure. Tuning lines in the log are yellow, failures and roll-backs orange. `FFGN_UV_CHILD=1` brings back the old separate check process. **Note:** all GPUs of a rig run in one miner process, so a fault on one GPU (usually an overclock or undervolt too far for that card) restarts the miner on every GPU; it recovers by itself in about a minute. The log names the GPU with the driver fault: lower that card's offset or use `--auto-uv`. Auto-lock and your OC settings are not changed by this release.

**What changed in 1.0.9** (recommended for everyone; big gains on RTX 20 / GTX 16 / CMP Turing cards and on RTX 30). GPU engine 0.6.0 chooses the fastest exact path automatically for every model of the chain. Most attempts of a model touch only a few live table rows: (1) the **digest-table path** looks up the attempts with at most two live rows in a table built for each model (about 0.2 s for TESS and 1.1 s for Kepler, often less when only part of the model changes), after a fast screen of the nonces; (2) the **cheap-attempt filter** evaluates only the attempts with at most k live rows (k = 3..6, chosen by measuring speed, and power where the card reports it). On every new model the engine measures the full path first and builds the table only when its expected rate is at least 0.8 x the measured full-path rate (the table path draws much less power); if the table is slower in practice it is turned off for that model. GTX 10 and Volta keep the full path. **The hashrate is now the number of attempts actually evaluated** (MA/s); the screened nonces are shown separately (`screened` in the stats line and the API, metric `ffgn_gpu_screened_total`). Every evaluated attempt has the same chance of a share as before; results, tickets, models and the dataset are unchanged bit for bit and the CPU reference still re-checks every share. The host sizes its scan calls by the path (larger calls on the table path) and starts the next header itself when a GPU has used one up. Measured on RTX 3070 at 1680 MHz / +210 / 200 W, live mining on one pool: Kepler 40.05 MA/s at 0.2172 MA/W (1.0.8) -> 65.8-83.1 MA/s at 0.59-0.74 MA/W (1.0.9), TESS 40.21 MA/s at 0.2474 MA/W -> 343-358 MA/s at 2.98-3.09 MA/W. On a CMP 50HX (Turing) at 1500 MHz on the same models: Kepler x3.8-x5.1 and TESS x10.5 per watt. The rate now depends on the model of the moment (see the table below). Auto-lock, auto-uv and your OC settings are not changed by this release.

**What changed in 1.0.8** (recommended for everyone, especially RTX 30). GPU engine 0.5.1 makes the KEP1 warp kernel faster where the table does not fit in the GPU's L2 cache: wide table rows take the fast path too, the tail of each row is prefetched into L2, more warps run per SM, and the work around the kernel is smaller (the attempt keys are written in one coalesced pass). On a card whose live table is larger than its L2 cache the engine can also run the attempts sorted by table row: it times sorted against unsorted on the first launches of every new model and keeps the sort only where it is faster on that card. Results, models, tickets and the dataset are unchanged bit for bit; the CPU reference still re-checks every share. RTX 3070 at 1680 MHz / +210 / 200 W, measured in the same session as 1.0.7: Kepler 31.37 -> 38.22 MA/s (+21.8 %) and 0.1905 -> 0.2064 MA/W (+8.4 %), TESS 36.81 -> 41.67 MA/s (+13.2 %) and 0.2463 -> 0.2524 MA/W (+2.5 %). On RTX 4070 Laptop GPU there is no clear change (within the noise of its thermal limit), so no gain is claimed for it. Auto-lock, auto-uv and your OC settings are not changed by this release.

**What changed in 1.0.7** (recommended for everyone). GPU engine 0.4.0 skips the convolution reads of zero table rows too (1.0.6 skipped only their big-linear reads), and the KEP1 warp kernel now uses one attempt per warp/block. Results, models, tickets and the dataset are unchanged bit for bit; the CPU reference still re-checks every share. On the chain's trained models an RTX 3070 reaches 31.4 MA/s Kepler and 37.9 MA/s TESS (about +63 % and +46 % from 1.0.6). TESS reaches 40.49 MA/s at a 1850 MHz lock; the best measured efficiency in the clock sweep is at 1680 MHz, not at the maximum-speed setting. The one-warp setting was also faster than the old one on RTX 4070 Laptop GPU.

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

Release page: https://github.com/pepsykolya/FreeForGemNeb/releases/tag/v1.0.13

| package | file |
|---|---|
| HiveOS (custom miner `ffgn-hive`) | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.13/ffgn-hive-1.0.13.tar.gz |
| Linux x86_64 | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.13/ffgn-1.0.13-linux-x86_64.tar.gz |
| Windows x64 | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.13/ffgn-1.0.13-windows-x64.zip |
| checksums | https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.13/SHA256SUMS |

The miner binary is `ffgn` (`ffgn.exe` on Windows). The dataset file of the NEB main network (`neb-datasets.bin`, about 16 MB) is
inside every package.

## Speed and power

Million attempts per second (MA/s) and million attempts per second per watt (MA/W), the real dataset, memory at its stock
clock, no undervolt. On the main network every block is KEP1 (slot 0 Kepler, slot 1 TESS: two datasets of different size).

**GPU engine 0.8.1 (ffgn 1.0.13)**, measured 21.11.2026, values per card (power = mean of the card's own readings under
load; MA/s = evaluated attempts).

Partial L=3 table for TESS (engine benchmark: one resident table, the same TESS models of the chain, alternating 8-s segments
with the triples ignored and used):

| Card, clocks (per card) | TESS, pairs only (as 1.0.11) | TESS, 1.0.13 with the triples | Rate | MA/W | Triples, full build |
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
   `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.13/ffgn-hive-1.0.13.tar.gz`, algorithm `nebhash`, wallet
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
