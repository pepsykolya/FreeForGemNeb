# FreeForGemNeb (ffgn) 1.0.12 on HiveOS

[English](#english) · [Русский](#русский)

## English

Custom miner **ffgn-hive** for NEB (Neural Engine Byte). The package already holds the miner, the GPU engine and the dataset
file of the NEB 1.0.1 main network (`neb-datasets.bin`, about 16 MB, checked by BLAKE3 at every start): nothing else is downloaded.

Requirements: a HiveOS image with glibc 2.30 or newer (every current image), an NVIDIA driver with CUDA 12 support (R525 or
newer; RTX 50 needs R570 or newer) selected in the rig settings, a GPU of compute capability 6.1 or newer, and **a correct rig
clock** (`timedatectl`: time synchronisation on).

### Pool URL: what to enter

| Mode | Pool URL in the flight sheet | Pass |
|---|---|---|
| Aria (PPLNS) | `neb.ariabrain.com:3370` | `x` |
| Aria, solo | `neb.ariabrain.com:3371` | `x` |
| Suprnova (PPLNS) | `stratum+tcp://neb.suprnova.cc:7804` | `x` |
| Suprnova, solo | `stratum+tcp://neb.suprnova.cc:7805` | `x` |
| failover | several URLs separated by spaces, the first is the primary, e.g. `neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804` | `x` |
| your node on the rig | `node://127.0.0.1:3441` (the official NEB node runs on the rig with its `--miner-lock`; the node needs glibc 2.34) | empty |
| an ffgn-gate beside your node | `GATE-HOST:3442` | the gate token |

TLS: `stratum+ssl://HOST:PORT` where a pool offers it. Before you mine for real check on the pool's page that the pool is on
the NEB **main** network. Pools are described in MANUAL section 9, the node and the gate in sections 10 and 11.

### Flight sheet: quick way (Import from Clipboard)

1. **Wallets**: add a wallet, coin `NEB` (type the name if HiveOS does not know the coin), your `neb1...` address.
2. **Flight Sheets**, **Import from Clipboard**, paste:

```json
{"name":"ffgn NEB Aria","isFavorite":false,"items":[{"coin":"NEB","pool_ssl":false,"pool_urls":["neb.ariabrain.com:3370"],"dpool_ssl":false,"miner":"custom","miner_alt":"ffgn-hive","miner_config":{"url":"neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804","algo":"nebhash","pass":"x","miner":"ffgn-hive","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.12/ffgn-hive-1.0.12.tar.gz","user_config":""},"pool_geo":[]}]}
```

3. Pick your `NEB` wallet in the red-marked field (the import carries no wallet), **Create flight sheet**, apply it to the rig.

If HiveOS refuses the import, use the manual way: the result is the same.

### Flight sheet: manual way

Coin `NEB`, Wallet: yours, Pool: *Configure in miner*, Miner: **Custom**, *Setup Miner Config*:

| Field | Value |
|---|---|
| Miner name | `ffgn-hive` (filled in automatically from the URL) |
| Installation URL | `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.12/ffgn-hive-1.0.12.tar.gz` |
| Hash algorithm | `nebhash` |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | see the table above |
| Pass | `x` (or empty) |
| Extra config arguments | optional: `--auto-lock`, `--auto-uv`, `--lock-cclock 1500`, `--pl 140`, `--devices 0,2`, ... |

**Rigs with many GPUs on a weak CPU** (2-4 CPU threads, a few GB of RAM, e.g. 8-12 cards on a Pentium / Celeron): ffgn 1.0.5
tunes `--auto-uv` one GPU at a time on rigs of 6+ GPUs and grows to 2 and 4 only while the rig stays healthy (the others
mine); give `--uv-parallel 1` in Extra config to keep it at one. A rig that restarted twice within an hour during a tuning
pauses auto-uv (`AUTO-UV PAUSED` in the version column) until `--uv-unpause` is given once. Since 1.0.10 auto-uv tunes while the cards mine (each card keeps its MA/s and shares), all GPUs at once; the limits above apply to the old check process, used only for a card that does not mine within 240 s or with `FFGN_UV_CHILD=1`. The GPUs return to full load one by one after a pause (`--ramp-ms 500` by default on 4+ GPUs; more, e.g. `--ramp-ms 1000`,
for a weak PSU or risers). A line `all N GPUs stalled at once ... likely a PCIe bus / power event` in the log points at risers,
cables or the PSU (`dmesg | grep -i xid`: Xid 79).

How the fields become the command line (`h-config.sh`): the part of the template before the first dot is the wallet, the rest
is the worker name (`--wallet`, `--worker`); each Pool URL becomes a `--pool`; a `node://HOST:PORT` URL becomes `--node`; Pass
becomes `--password` (unless it is `x`); `--api 127.0.0.1:4078` is added; Extra config arguments are appended at the end.

The miner uses every GPU of the rig by itself (`--devices` narrows the choice). Log: `miner log`, or
`/var/log/miner/ffgn-hive/ffgn-hive.log`. The dashboard shows speed per GPU (1 "hash" = 1 evaluated attempt, since 1.0.9; kH/s = MA/s x 1000),
temperature, fans and accepted / rejected shares; the version column carries the efficiency (`1.0.5 0.0240 MA/W, slot 1 TESS`; 1.0.4: the hash rate is the 5-minute average), and
`need update!!` when a newer release is out. ffgn 1.0.2 is required for the NEB 1.0.1 main network (1.0.1 does not work on it); on the main network every block is KEP1,
so the speed is steady (MANUAL section 18).

**OC profile.** With `--lock-cclock`, `--lock-mclock`, `--moffset`, `--auto-uv`, `--auto-lock` the miner sets clocks and the
undervolt itself and puts your values back when it stops (the lock and the core clock offset of the OC profile, or reset). Keep
the OC profile for fans and the power limit; do not set a core offset in it together with `--auto-uv` / `--auto-lock`. If
you apply an OC profile in the web UI while the miner runs, the miner sets its own values again after a quick check. `miner
stop` puts the profile's values back. Undervolt and lock results are kept in `/hive-config/ffgn` (they survive reboots and
updates); `--uv-reset-all` forgets them.

**Power limit in Extra config.** `--power-limit N` (or `--pl N`) sets the power limit to N W while the miner runs: one value for
all GPUs, or one per GPU in `--devices` order (`--pl 140,115,140`; `-` = keep that GPU's limit from the OC profile). It is
clamped to the card's range, and the OC profile's power limit is put back when the miner stops (also after a crash, at the next
start). If HiveOS applies the OC profile while the miner runs, the miner sets its limit again and says so in the log. The stats
line shows `PL 140 W` per GPU (`capped` while the limit holds the clocks down); auto-uv tunes with the limit in force.

**Updating.** HiveOS caches archives by file name: after a new release use the new Installation URL. The miner only reports a
newer release (`need update!!`); `--no-update-check` hides it.

**Solo on a rig.** The node's work port is local only. Either run the node on the rig (`node://127.0.0.1:3441`), or run an
**ffgn-gate** beside a node somewhere else and give the rigs `GATE-HOST:3442` with the token in Pass (MANUAL section 11). The
reward of a block goes to the node's `--miner-lock`.

### Watchdog and a lost GPU (recommended)

A GPU can fall off the PCIe bus (Xid 79: a riser, its power, the card); the NVIDIA driver gets it back only after a reboot of
the rig. Two settings make a rig recover by itself instead of being found the next morning not mining:

1. **HiveOS Watchdog** (rig settings, *Watchdog*): enable it; *Minimal hashrate* about **50 %** of the rig's normal total for
   `nebhash` (the dashboard shows kH/s = MA/s x 1000: 120 MA/s normal -> 60000); *Restart miner after* **10-15 min**; *Reboot
   after* **20-30 min**.
2. **`--reboot-on-lost-gpu 10`** in *Extra config arguments*: a GPU lost for 10 minutes (Xid 79, its engine cannot reopen the
   device = `DEAD`, or NVML no longer sees it) makes the miner reboot the rig with HiveOS `sreboot`. Never in the first 10 min
   of the miner, at most once per 60 min (`/hive-config/ffgn/reboot-on-lost-gpu.json`), so a card still lost after the reboot
   does not reboot the rig in a loop; the Watchdog and the dashboard still show it. A `WARNING: REBOOTING THE RIG ...` line
   comes first in `miner log`. `FFGN_NO_REBOOT=1` (or `--single-process`) turns it off.

Always, without options: a lost GPU shows **0** on the dashboard (and `GPU LOST GPU1` next to the version), so the total is
the true one and the minimal hashrate rule fires; when every GPU is down or not mining for 5 min although there is work, the
miner exits with code 3 and HiveOS starts it again.

### Troubleshooting

| Symptom | What to check |
|---|---|
| The miner does not start | The Installation URL of the right release, algorithm `nebhash`, the wallet template `%WAL%.%WORKER_NAME%`; `miner log`. |
| "Wallet template is empty" / "Pool URL is empty" in the log | Fill the flight sheet fields. |
| No accepted shares | Pool address, `neb1...` address, the pool works on the main network; a wrong rig clock. |
| Speed lower than expected | The OC profile or the power limit; temperature; compare with MANUAL section 13. |
| `GPU LOST` / `REBOOT NEEDED` next to the version | A GPU fell off the bus (Xid 79) or cannot be opened: reboot the rig, then check that GPU's riser and power. `--reboot-on-lost-gpu 10` does the reboot by itself (see above). |


### Memory clock (important)

NEB is memory bound. An OC profile that locks the memory low (Pearl profiles: 810 MHz) makes NEB about 10x slower. ffgn replaces
such a lock with the card's maximum while it mines and puts the profile's value back when it stops (log: `WARNING the HiveOS OC
profile locks the memory to 810 MHz ...`). `--keep-hive-mem` in Extra config keeps the profile's value. Simpler: give the NEB
flight sheet an OC profile without a memory lock. Since 1.0.12 the TESS triples (`--tbig auto`, the default) need nothing in Extra config. Optional since 1.0.11: `--table-mclock 810` in Extra config locks the memory low only while the pure digest-table path runs (the full clock comes back by itself for every build and every other path).

## Русский

Кастомный майнер **ffgn-hive** для NEB (Neural Engine Byte). В пакете уже есть майнер, GPU-движок и файл датасета основной
сети NEB 1.0.1 (`neb-datasets.bin`, около 16 МБ, проверяется по BLAKE3 при каждом запуске): ничего больше не скачивается.

Требования: образ HiveOS с glibc 2.30 или новее (любой текущий), драйвер NVIDIA с поддержкой CUDA 12 (R525 или новее; для
RTX 50 — R570 или новее), выбранный в настройках рига, видеокарта с compute capability 6.1 или выше и **верные часы рига**
(`timedatectl`: синхронизация времени включена).

### Pool URL: что вписать

| Режим | Pool URL в полётном листе | Pass |
|---|---|---|
| Aria (PPLNS) | `neb.ariabrain.com:3370` | `x` |
| Aria, соло | `neb.ariabrain.com:3371` | `x` |
| Suprnova (PPLNS) | `stratum+tcp://neb.suprnova.cc:7804` | `x` |
| Suprnova, соло | `stratum+tcp://neb.suprnova.cc:7805` | `x` |
| резервирование | несколько URL через пробел, первый основной, например `neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804` | `x` |
| ваша нода на риге | `node://127.0.0.1:3441` (на риге работает официальная нода NEB со своим `--miner-lock`; ноде нужна glibc 2.34) | пусто |
| ffgn-gate рядом с вашей нодой | `GATE-HOST:3442` | токен шлюза |

TLS: `stratum+ssl://HOST:PORT`, где пул его предлагает. Прежде чем майнить всерьёз, убедитесь по странице пула, что пул работает
в **основной** сети NEB. Пулы описаны в MANUAL.ru.md, раздел 9, нода и шлюз — в разделах 10 и 11.

### Полётный лист: быстрый способ (Import from Clipboard)

1. **Wallets**: добавьте кошелёк, монета `NEB` (впишите название, если HiveOS не знает монету), ваш адрес `neb1...`.
2. **Flight Sheets**, **Import from Clipboard**, вставьте:

```json
{"name":"ffgn NEB Aria","isFavorite":false,"items":[{"coin":"NEB","pool_ssl":false,"pool_urls":["neb.ariabrain.com:3370"],"dpool_ssl":false,"miner":"custom","miner_alt":"ffgn-hive","miner_config":{"url":"neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804","algo":"nebhash","pass":"x","miner":"ffgn-hive","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.12/ffgn-hive-1.0.12.tar.gz","user_config":""},"pool_geo":[]}]}
```

3. Выберите кошелёк `NEB` в поле, отмеченном красным (импорт кошелёк не переносит), **Create flight sheet**, примените к ригу.

Если HiveOS отказывается импортировать, пользуйтесь ручным способом: результат тот же.

### Полётный лист: вручную

Coin `NEB`, Wallet: ваш, Pool: *Configure in miner*, Miner: **Custom**, *Setup Miner Config*:

| Поле | Значение |
|---|---|
| Miner name | `ffgn-hive` (заполняется автоматически из URL) |
| Installation URL | `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.12/ffgn-hive-1.0.12.tar.gz` |
| Hash algorithm | `nebhash` |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | см. таблицу выше |
| Pass | `x` (или пусто) |
| Extra config arguments | по желанию: `--auto-lock`, `--auto-uv`, `--lock-cclock 1500`, `--pl 140`, `--devices 0,2`, ... |

**Риги с множеством карт на слабом CPU** (2–4 потока CPU, несколько ГБ RAM, например 8–12 карт на Pentium / Celeron): ffgn 1.0.5
подбирает `--auto-uv` по одной карте на ригах от 6 карт и увеличивает до 2 и 4, только пока риг здоров (остальные майнят);
`--uv-parallel 1` в Extra config оставит по одной. Риг, дважды за час перезапустившийся во время подбора, ставит auto-uv на
паузу (`AUTO-UV PAUSED` в колонке версии) до однократного `--uv-unpause`. С 1.0.10 auto-uv подбирает, пока карты майнят (каждая карта сохраняет MA/s и шары), все карты одновременно; ограничения выше относятся к прежнему проверочному процессу, который используется только для карты, не начавшей майнить за 240 с, или при `FFGN_UV_CHILD=1`.
После паузы карты выходят на полную нагрузку по одной (`--ramp-ms 500` по умолчанию на 4+ картах; при слабом блоке питания или
райзерах — больше, например `--ramp-ms 1000`). Строка `all N GPUs stalled at once ... likely a PCIe bus / power event` в логе
указывает на райзеры, кабели или блок питания (`dmesg | grep -i xid`: Xid 79).

Как поля превращаются в командную строку (`h-config.sh`): часть шаблона до первой точки — кошелёк, остальное — имя воркера
(`--wallet`, `--worker`); каждый Pool URL становится `--pool`; URL вида `node://HOST:PORT` становится `--node`; Pass — это
`--password` (если не `x`); добавляется `--api 127.0.0.1:4078`; Extra config arguments дописываются в конец.

Майнер сам использует все видеокарты рига (`--devices` сужает выбор). Лог: `miner log` или
`/var/log/miner/ffgn-hive/ffgn-hive.log`. Дашборд показывает скорость по картам (1 «хэш» = 1 вычисленная попытка, с 1.0.9; кХ/с = MA/s x 1000),
температуру, кулеры и принятые / отклонённые шары; в колонке версии — эффективность (`1.0.5 0.0240 MA/W, slot 1 TESS`; 1.0.4: хешрейт — среднее за 5 минут) и `need update!!`,
если вышел новый релиз. ffgn 1.0.2 нужен для основной сети NEB 1.0.1 (1.0.1 на ней не работает); в основной сети каждый блок — KEP1, поэтому скорость ровная (MANUAL.ru.md, раздел 18).

**OC-профиль.** С `--lock-cclock`, `--lock-mclock`, `--moffset`, `--auto-uv`, `--auto-lock` майнер сам выставляет частоты и
андервольт и при остановке возвращает ваши значения (лок и смещение ядра OC-профиля либо сброс). Оставьте OC-профилю кулеры и
лимит мощности; не задавайте в нём смещение ядра вместе с `--auto-uv` / `--auto-lock`. Если вы применяете OC-профиль в
веб-интерфейсе, пока майнер работает, майнер после быстрой проверки снова выставит свои значения. `miner stop` возвращает
значения профиля. Результаты подбора андервольта и лока лежат в `/hive-config/ffgn` (переживают перезагрузки и обновления);
`--uv-reset-all` их забывает.

**Лимит мощности в Extra config.** `--power-limit N` (или `--pl N`) ставит лимит мощности N Вт на время работы майнера: одно
значение на все карты или по одному на карту в порядке `--devices` (`--pl 140,115,140`; `-` — оставить этой карте лимит из
OC-профиля). Значение подрезается до диапазона карты, а при остановке майнера возвращается лимит из OC-профиля (и после сбоя —
при следующем запуске). Если HiveOS применит OC-профиль во время майнинга, майнер снова выставит свой лимит и напишет об этом в
лог. В строке статистики у каждой карты видно `PL 140 W` (`capped` — пока лимит сдерживает частоты); auto-uv подбирает с
действующим лимитом.

**Обновление.** HiveOS кэширует архивы по имени файла: после нового релиза берите новый Installation URL. Майнер только сообщает
о новом релизе (`need update!!`); `--no-update-check` это скрывает.

**Соло на риге.** Рабочий порт ноды только локальный. Либо запустите ноду на риге (`node://127.0.0.1:3441`), либо поставьте
**ffgn-gate** рядом с нодой в другом месте и дайте ригам `GATE-HOST:3442` с токеном в Pass (MANUAL.ru.md, раздел 11). Награда за
блок идёт на `--miner-lock` ноды.

### Watchdog и потерянная карта (рекомендуем)

Карта может отвалиться от шины PCIe (Xid 79: райзер, его питание, сама карта); драйвер NVIDIA возвращает её только после
перезагрузки рига. Две настройки дают ригу восстановиться самому, а не стоять до утра без майнинга:

1. **Watchdog HiveOS** (настройки рига, *Watchdog*): включить; *Minimal hashrate* — около **50 %** обычной общей скорости рига для
   `nebhash` (панель показывает kH/s = MA/s x 1000: обычно 120 MA/s -> 60000); *Restart miner after* — **10–15 мин**; *Reboot
   after* — **20–30 мин**.
2. **`--reboot-on-lost-gpu 10`** в *Extra config arguments*: карта, потерянная 10 минут (Xid 79, её движок не может заново открыть
   устройство = `DEAD`, или её больше не видит NVML), — майнер перезагружает риг командой HiveOS `sreboot`. Никогда в первые 10 мин
   работы майнера, не чаще раза в 60 мин (`/hive-config/ffgn/reboot-on-lost-gpu.json`), так что карта, потерянная и после
   перезагрузки, не уводит риг в цикл перезагрузок; Watchdog и панель всё равно её покажут. Сначала в `miner log` пишется строка
   `WARNING: REBOOTING THE RIG ...`. `FFGN_NO_REBOOT=1` (или `--single-process`) выключает перезагрузку.

Всегда, без опций: потерянная карта показывает на панели **0** (и `GPU LOST GPU1` рядом с версией), общая скорость честная, и
правило minimal hashrate срабатывает; если все карты не работают или не майнят 5 мин при наличии задания, майнер выходит с кодом 3,
и HiveOS запускает его снова.

### Проблемы и решения

| Симптом | Что проверить |
|---|---|
| Майнер не запускается | Installation URL нужного релиза, алгоритм `nebhash`, шаблон кошелька `%WAL%.%WORKER_NAME%`; `miner log`. |
| В логе «Wallet template is empty» / «Pool URL is empty» | Заполните поля полётного листа. |
| Нет принятых шар | Адрес пула, адрес `neb1...`, пул работает в основной сети; неверные часы рига. |
| Скорость ниже ожидаемой | OC-профиль или лимит мощности; температура; сравните с MANUAL.ru.md, раздел 13. |
| `GPU LOST` / `REBOOT NEEDED` рядом с версией | Карта отвалилась от шины (Xid 79) или не открывается: перезагрузите риг, затем проверьте райзер и питание этой карты. `--reboot-on-lost-gpu 10` перезагрузит сам (см. выше). |

### Частота памяти (важно)

NEB упирается в память. OC-профиль с низким локом памяти (профили для Pearl: 810 МГц) замедляет NEB примерно в 10 раз. ffgn на
время майнинга заменяет такой лок на максимум карты и возвращает значение профиля при остановке (в логе: `WARNING the HiveOS OC
profile locks the memory to 810 MHz ...`). `--keep-hive-mem` в Extra config оставляет значение профиля. Проще: дайте полётному
листу NEB OC-профиль без лока памяти. С 1.0.12 тройкам TESS (`--tbig auto`, по умолчанию) ничего в Extra config не нужно. С 1.0.11 по желанию: `--table-mclock 810` в Extra config фиксирует память низко только пока идёт чистый путь таблицы дайджестов (на каждую сборку и любой другой путь полная частота возвращается сама).
