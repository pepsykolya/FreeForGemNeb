# FreeForGemNeb (ffgn) 1.0.2 on HiveOS

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
{"name":"ffgn NEB Aria","isFavorite":false,"items":[{"coin":"NEB","pool_ssl":false,"pool_urls":["neb.ariabrain.com:3370"],"dpool_ssl":false,"miner":"custom","miner_alt":"ffgn-hive","miner_config":{"url":"neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804","algo":"nebhash","pass":"x","miner":"ffgn-hive","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.2/ffgn-hive-1.0.2.tar.gz","user_config":""},"pool_geo":[]}]}
```

3. Pick your `NEB` wallet in the red-marked field (the import carries no wallet), **Create flight sheet**, apply it to the rig.

If HiveOS refuses the import, use the manual way: the result is the same.

### Flight sheet: manual way

Coin `NEB`, Wallet: yours, Pool: *Configure in miner*, Miner: **Custom**, *Setup Miner Config*:

| Field | Value |
|---|---|
| Miner name | `ffgn-hive` (filled in automatically from the URL) |
| Installation URL | `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.2/ffgn-hive-1.0.2.tar.gz` |
| Hash algorithm | `nebhash` |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | see the table above |
| Pass | `x` (or empty) |
| Extra config arguments | optional: `--auto-lock`, `--auto-uv`, `--lock-cclock 1500`, `--devices 0,2`, ... |

How the fields become the command line (`h-config.sh`): the part of the template before the first dot is the wallet, the rest
is the worker name (`--wallet`, `--worker`); each Pool URL becomes a `--pool`; a `node://HOST:PORT` URL becomes `--node`; Pass
becomes `--password` (unless it is `x`); `--api 127.0.0.1:4078` is added; Extra config arguments are appended at the end.

The miner uses every GPU of the rig by itself (`--devices` narrows the choice). Log: `miner log`, or
`/var/log/miner/ffgn-hive/ffgn-hive.log`. The dashboard shows speed per GPU (1 "hash" = 1 attempt; kH/s = MA/s x 1000),
temperature, fans and accepted / rejected shares; the version column carries the efficiency (`1.0.2 0.0240 MA/W`), and
`need update!!` when a newer release is out. ffgn 1.0.2 is required for the NEB 1.0.1 main network (1.0.1 does not work on it); on the main network every block is KEP1,
so the speed is steady (MANUAL section 18).

**OC profile.** With `--lock-cclock`, `--lock-mclock`, `--moffset`, `--auto-uv`, `--auto-lock` the miner sets clocks and the
undervolt itself and puts your values back when it stops (the lock and the core clock offset of the OC profile, or reset). Keep
the OC profile for fans and the power limit; do not set a core offset in it together with `--auto-uv` / `--auto-lock`. If
you apply an OC profile in the web UI while the miner runs, the miner sets its own values again after a quick check. `miner
stop` puts the profile's values back. Undervolt and lock results are kept in `/hive-config/ffgn` (they survive reboots and
updates); `--uv-reset-all` forgets them.

**Updating.** HiveOS caches archives by file name: after a new release use the new Installation URL. The miner only reports a
newer release (`need update!!`); `--no-update-check` hides it.

**Solo on a rig.** The node's work port is local only. Either run the node on the rig (`node://127.0.0.1:3441`), or run an
**ffgn-gate** beside a node somewhere else and give the rigs `GATE-HOST:3442` with the token in Pass (MANUAL section 11). The
reward of a block goes to the node's `--miner-lock`.

### Troubleshooting

| Symptom | What to check |
|---|---|
| The miner does not start | The Installation URL of the right release, algorithm `nebhash`, the wallet template `%WAL%.%WORKER_NAME%`; `miner log`. |
| "Wallet template is empty" / "Pool URL is empty" in the log | Fill the flight sheet fields. |
| No accepted shares | Pool address, `neb1...` address, the pool works on the main network; a wrong rig clock. |
| Speed lower than expected | The OC profile or the power limit; temperature; compare with MANUAL section 13. |


### Memory clock (important)

NEB is memory bound. An OC profile that locks the memory low (Pearl profiles: 810 MHz) makes NEB about 10x slower. ffgn replaces
such a lock with the card's maximum while it mines and puts the profile's value back when it stops (log: `WARNING the HiveOS OC
profile locks the memory to 810 MHz ...`). `--keep-hive-mem` in Extra config keeps the profile's value. Simpler: give the NEB
flight sheet an OC profile without a memory lock.

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
{"name":"ffgn NEB Aria","isFavorite":false,"items":[{"coin":"NEB","pool_ssl":false,"pool_urls":["neb.ariabrain.com:3370"],"dpool_ssl":false,"miner":"custom","miner_alt":"ffgn-hive","miner_config":{"url":"neb.ariabrain.com:3370 stratum+tcp://neb.suprnova.cc:7804","algo":"nebhash","pass":"x","miner":"ffgn-hive","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.2/ffgn-hive-1.0.2.tar.gz","user_config":""},"pool_geo":[]}]}
```

3. Выберите кошелёк `NEB` в поле, отмеченном красным (импорт кошелёк не переносит), **Create flight sheet**, примените к ригу.

Если HiveOS отказывается импортировать, пользуйтесь ручным способом: результат тот же.

### Полётный лист: вручную

Coin `NEB`, Wallet: ваш, Pool: *Configure in miner*, Miner: **Custom**, *Setup Miner Config*:

| Поле | Значение |
|---|---|
| Miner name | `ffgn-hive` (заполняется автоматически из URL) |
| Installation URL | `https://github.com/pepsykolya/FreeForGemNeb/releases/download/v1.0.2/ffgn-hive-1.0.2.tar.gz` |
| Hash algorithm | `nebhash` |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | см. таблицу выше |
| Pass | `x` (или пусто) |
| Extra config arguments | по желанию: `--auto-lock`, `--auto-uv`, `--lock-cclock 1500`, `--devices 0,2`, ... |

Как поля превращаются в командную строку (`h-config.sh`): часть шаблона до первой точки — кошелёк, остальное — имя воркера
(`--wallet`, `--worker`); каждый Pool URL становится `--pool`; URL вида `node://HOST:PORT` становится `--node`; Pass — это
`--password` (если не `x`); добавляется `--api 127.0.0.1:4078`; Extra config arguments дописываются в конец.

Майнер сам использует все видеокарты рига (`--devices` сужает выбор). Лог: `miner log` или
`/var/log/miner/ffgn-hive/ffgn-hive.log`. Дашборд показывает скорость по картам (1 «хэш» = 1 попытка; кХ/с = MA/s x 1000),
температуру, кулеры и принятые / отклонённые шары; в колонке версии — эффективность (`1.0.2 0.0240 MA/W`) и `need update!!`,
если вышел новый релиз. ffgn 1.0.2 нужен для основной сети NEB 1.0.1 (1.0.1 на ней не работает); в основной сети каждый блок — KEP1, поэтому скорость ровная (MANUAL.ru.md, раздел 18).

**OC-профиль.** С `--lock-cclock`, `--lock-mclock`, `--moffset`, `--auto-uv`, `--auto-lock` майнер сам выставляет частоты и
андервольт и при остановке возвращает ваши значения (лок и смещение ядра OC-профиля либо сброс). Оставьте OC-профилю кулеры и
лимит мощности; не задавайте в нём смещение ядра вместе с `--auto-uv` / `--auto-lock`. Если вы применяете OC-профиль в
веб-интерфейсе, пока майнер работает, майнер после быстрой проверки снова выставит свои значения. `miner stop` возвращает
значения профиля. Результаты подбора андервольта и лока лежат в `/hive-config/ffgn` (переживают перезагрузки и обновления);
`--uv-reset-all` их забывает.

**Обновление.** HiveOS кэширует архивы по имени файла: после нового релиза берите новый Installation URL. Майнер только сообщает
о новом релизе (`need update!!`); `--no-update-check` это скрывает.

**Соло на риге.** Рабочий порт ноды только локальный. Либо запустите ноду на риге (`node://127.0.0.1:3441`), либо поставьте
**ffgn-gate** рядом с нодой в другом месте и дайте ригам `GATE-HOST:3442` с токеном в Pass (MANUAL.ru.md, раздел 11). Награда за
блок идёт на `--miner-lock` ноды.

### Проблемы и решения

| Симптом | Что проверить |
|---|---|
| Майнер не запускается | Installation URL нужного релиза, алгоритм `nebhash`, шаблон кошелька `%WAL%.%WORKER_NAME%`; `miner log`. |
| В логе «Wallet template is empty» / «Pool URL is empty» | Заполните поля полётного листа. |
| Нет принятых шар | Адрес пула, адрес `neb1...`, пул работает в основной сети; неверные часы рига. |
| Скорость ниже ожидаемой | OC-профиль или лимит мощности; температура; сравните с MANUAL.ru.md, раздел 13. |

### Частота памяти (важно)

NEB упирается в память. OC-профиль с низким локом памяти (профили для Pearl: 810 МГц) замедляет NEB примерно в 10 раз. ffgn на
время майнинга заменяет такой лок на максимум карты и возвращает значение профиля при остановке (в логе: `WARNING the HiveOS OC
profile locks the memory to 810 MHz ...`). `--keep-hive-mem` в Extra config оставляет значение профиля. Проще: дайте полётному
листу NEB OC-профиль без лока памяти.
