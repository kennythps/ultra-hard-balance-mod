# Ultra Hard Balance Mod — THUG Pro

**[Русский](#русский) · [English](#english)**

---

## Русский

Мод для **Tony Hawk's Underground Pro**, который делает полоску баланса по-настоящему злой.
Мануалы, гринды, липы и скитч перестают прощаться — на высоких значениях их приходится
держать руками, а не «залипать» на центр.

Мод меняет **один параметр в одном файле**. Это не сборник правок и не перепаханная физика:
всё остальное в игре остаётся ровно таким, каким его сделали разработчики THUG Pro.

### Что именно меняется

Ускоряется наклон баланс-метра — параметр `$Lean_Acc$`. Их четыре, по одному на каждый тип баланса:

| Структура | Стат, от которого зависит | Ваниль | Мод | Во сколько раз |
|---|---|---:|---:|---:|
| `ManualParams` | `STATS_MANUAL` | 10 | **47** | **4.7×** |
| `LipParams` | `STATS_LIPBALANCE` | 10 | **47** | **4.7×** |
| `SkitchParams` | — (без стата) | 8 | **50** | **6.25×** |
| `GrindParams` | `STATS_RAILBALANCE` | 10 | **90** | **9×** |

Гринд наказан жёстче всех — 9×. Скитч, наоборот, самый мягкий, но только в абсолюте:
у него и база была ниже (8 против 10).

### Что такое `Lean_Acc` и почему «hard» = «больше число»

`$Lean_Acc$` — это **ускорение наклона** баланс-метра: насколько резко полоска уезжает от центра.
Чем больше значение, тем быстрее метка уходит в сторону и тем меньше у вас времени на возврат.
Поэтому «сложнее» здесь — это «выше число», и мод честно называется *Ultra Hard*.

Значение не применяется «как есть» — THUG Pro домножает его на два коэффициента из того же файла:

```
$LEAN_ACC_DIFF$   = %vec2(0.750000, 1.000000)   // 0.75× при нулевом стате → 1.0× при максимальном
$standard_switch$ = %vec2(0.900000, 1.000000)   // 0.90× в свитче
```

Отсюда важное следствие: **множитель из таблицы выше одинаков на любом стате и в свитче**.
Оба коэффициента присутствуют в обеих версиях файла, поэтому отношение «мод / ваниль» —
константа. Мод не «наказывает новичков сильнее», он увеличивает сложность ровно во столько раз,
сколько написано в таблице, для любого персонажа.

Для наглядности, гриндовый баланс на персонаже с нулевым статом в свитче:

```
ваниль:  10 × 0.75 × 0.9 ≈  6.8
мод:     90 × 0.75 × 0.9 ≈ 60.8
```

### Установка

1. Скачайте архив мода из раздела **Releases**.
2. Сделайте бэкап папки `data\pre\` вашей установки THUG Pro.
3. Скопируйте `data\pre\thugpro_qb.prx` в `<THUG Pro>\data\pre\`, согласившись на замену.
4. Запустите игру.

`thugpro_qb.prx` — это контейнер со скриптами игры. Мод заменяет его **целиком**,
а не патчит один файл внутри, поэтому шаг 2 не формальность.

### Про `updt.dll`

В пакете лежат два варианта этой библиотеки — это D3D9-прокси, который THUG Pro использует
как загрузчик обновлений.

| Файл | Что делает |
|---|---|
| `updt.dll` (в корне пакета) | Адрес манифеста обновлений заменён заглушкой — апдейтер не может получить манифест и **не перезапишет мод** |
| `turn on thug pro update system\updt.dll` | Настоящий апдейтер (`dl.thugpro.com`) — для тех, кто хочет вернуть автообновление |

Без первого варианта автообновление при запуске вернуло бы ванильные скрипты, и мод бы просто исчез.
Это не вирус и не что-то скрытое — обычная защита мода от затирания. Если сомневаетесь,
не копируйте `updt.dll` вовсе: мод будет работать, но обновления игры смогут его снести.

### Совместимость — прочитайте перед установкой

Мод собран на базе THUG Pro **образца 2022 года** (архив 1 655 748 байт, `physics.qb` 27 808 байт).
С тех пор игра обновлялась, и её `thugpro_qb.prx` сильно изменился.

Проверка на актуальной установке (архив 1 619 808 байт, `physics.qb` 30 155 байт):
из 210 общих скриптов **108 совпадают байт в байт, 102 отличаются**, и один скрипт
(`qb\thugpro\thugpro_menu.qb`) в свежей версии появился, а в моде его нет.

Практический вывод: **установка мода откатывает не только баланс, но и все те 102 скрипта
к версии 2022 года.** Для одиночной игры и локальных сессий это обычно не проблема,
но перед установкой сделайте бэкап `data\pre\`, а если вы играете в мультиплеере на актуальной
версии — имейте в виду, что расхождение скриптов может привести к несовпадению с хостом.

Мод не трогает ни одного значения, кроме четырёх `Lean_Acc`. Всё остальное в `physics.qb`
версии 2022 года — ванильное.

### Как это было собрано

Скрипты THUG Pro лежат в `.prx`-архиве в скомпилированном виде (QB-байткод), поэтому правка
делается в три шага:

1. **Декомпиляция.** `physics.qb` из архива → текстовый исходник `physics.txt`.
   Использовался `roqq.exe` (`roqq.exe -d physics.qb`); рядом должен лежать
   `physics.qb_table.qbi` — таблица QB-ключей, без неё читаются только хеши вместо имён.
2. **Правка.** В `physics.txt` меняются четыре строки `$Lean_Acc$` — в `ManualParams`,
   `LipParams`, `SkitchParams` и `GrindParams`. Больше ничего не трогается.
3. **Сборка.** `roqq.exe -c physics.txt` → `physics.qb`, затем `PreTool.exe` открывает
   `thugpro_qb.prx`, находит `qb\game\skater\physics.qb`, делает `Replace` и сохраняет архив.

Проверки, которые подтверждают, что в релизе лежит именно этот исходник:

- `roqq.exe -d physics.qb` воспроизводит `physics.txt` **байт в байт** — исходник полный, ничего не потеряно.
- `roqq.exe -c physics.txt` даёт `.qb`, у которого **первые 15 131 байт совпадают** с релизным
  `physics.qb`; дальше в файле дописана таблица QB-ключей.
- Внутренняя запись архива сообщает несжатый размер **27 808** — ровно размер релизного `physics.qb`.
- Упаковка релизного `physics.qb` тем же алгоритмом (LZSS), что использует `PreTool.exe`,
  даёт **ровно те 11 974 байта**, что лежат в `.prx` — 0 отличий. Обратная распаковка возвращает
  файл байт в байт.

Сверка с ванилью того же билда (THUG Pro, `physics.qb` 27 809 байт) через ту же таблицу ключей
даёт **794 строки в обеих версиях и ровно 4 различающиеся строки** — те самые четыре `Lean_Acc`.
Это и есть весь мод.

### Кредиты

- **Matrix717** — подбор значений баланса. Числа 47 / 47 / 50 / 90 найдены вместе с ним.
- **kennythps** — декомпиляция, правка, сборка и упаковка.

Инструменты, которыми это собиралось, написаны не нами и в репозиторий не входят:
`PreTool.exe` (Alabaster, Inc., 2011) — редактор `.prx`/PRE; `roq cmd.exe` (2007) — GUI-обёртка;
`roqq.exe` (2017) — компилятор/декомпилятор QB. Спасибо их авторам: без них модов на THUG Pro
просто не существовало бы. Если будете собирать сами — берите эти инструменты у их авторов
и проверяйте на свой страх и риск.

### Файлы в репозитории

```
physics.txt             исходник скрипта целиком (794 строки) — главный артефакт
physics.qb_table.qbi    таблица QB-ключей, нужна для компиляции и декомпиляции
README.md               этот файл
LICENSE                 MIT
```

Скомпилированный `physics.qb` и готовый `thugpro_qb.prx` в git не хранятся — они
прикреплены к релизу. Так репозиторий остаётся лёгким, а диффы — читаемыми.

### Лицензия

MIT — см. [LICENSE](LICENSE). Берите, меняйте, форкайте, выкладывайте свои сборки.
Исходные скрипты THUG Pro принадлежат их правообладателям; здесь распространяется
только изменённый параметр и текст, описывающий его.

---

## English

A mod for **Tony Hawk's Underground Pro** that makes the balance meter genuinely punishing.
Manuals, grinds, lips and skitches stop being free — at high values you have to hold them
instead of parking the marker in the middle.

The mod changes **one parameter in one file**. It is not a compilation of tweaks and not a
reworked physics model: everything else in the game is exactly what THUG Pro's developers shipped.

### What changes

The lean acceleration of the balance meter — `$Lean_Acc$`. There are four of them, one per balance type:

| Struct | Stat it scales with | Vanilla | Mod | Multiplier |
|---|---|---:|---:|---:|
| `ManualParams` | `STATS_MANUAL` | 10 | **47** | **4.7×** |
| `LipParams` | `STATS_LIPBALANCE` | 10 | **47** | **4.7×** |
| `SkitchParams` | — (no stat) | 8 | **50** | **6.25×** |
| `GrindParams` | `STATS_RAILBALANCE` | 10 | **90** | **9×** |

Grinds are punished hardest at 9×. Skitch is the most forgiving in absolute terms, but only
because its base value was lower to begin with (8 instead of 10).

### What `Lean_Acc` is, and why "hard" means "higher number"

`$Lean_Acc$` is the **lean acceleration** of the balance meter — how sharply the marker drifts
away from centre. The higher the value, the faster it tips and the less time you have to bring it
back. So "harder" here really is "higher number", which is why the mod is called *Ultra Hard*.

The value is not applied raw — THUG Pro multiplies it by two coefficients from the same file:

```
$LEAN_ACC_DIFF$   = %vec2(0.750000, 1.000000)   // 0.75× at zero stat → 1.0× at max stat
$standard_switch$ = %vec2(0.900000, 1.000000)   // 0.90× when riding switch
```

Which leads to a useful property: **the multiplier in the table is identical at every stat level
and in switch.** Both coefficients are present in both versions of the file, so the mod/vanilla
ratio is constant. The mod does not punish beginners disproportionately — it scales difficulty by
exactly the stated factor for every character.

For illustration, grind balance on a zero-stat skater riding switch:

```
vanilla:  10 × 0.75 × 0.9 ≈  6.8
mod:      90 × 0.75 × 0.9 ≈ 60.8
```

### Installation

1. Download the mod archive from **Releases**.
2. Back up your THUG Pro `data\pre\` folder.
3. Copy `data\pre\thugpro_qb.prx` into `<THUG Pro>\data\pre\`, overwriting.
4. Launch the game.

`thugpro_qb.prx` is the game's script container. The mod replaces it **as a whole** rather than
patching a single file inside, so step 2 is not a formality.

### About `updt.dll`

The package ships two builds of this library — it is a D3D9 proxy that THUG Pro uses as its
updater stub.

| File | Behaviour |
|---|---|
| `updt.dll` (package root) | The update manifest host is stubbed out, so the updater cannot fetch a manifest and **will not overwrite the mod** |
| `turn on thug pro update system\updt.dll` | The real updater (`dl.thugpro.com`) — for anyone who wants automatic updates back |

Without the first one, the auto-updater would restore vanilla scripts on launch and the mod would
simply vanish. Nothing hidden or malicious about it — it is ordinary protection against being
overwritten. If you would rather not use it, skip `updt.dll` entirely: the mod still works, but a
game update may wipe it.

### Compatibility — please read before installing

The mod was built against the **2022-era** THUG Pro (archive 1,655,748 bytes, `physics.qb`
27,808 bytes). The game has been updated since, and its `thugpro_qb.prx` has changed a lot.

Measured against a current install (archive 1,619,808 bytes, `physics.qb` 30,155 bytes):
of 210 shared scripts, **108 are byte-identical and 102 differ**, and one script
(`qb\thugpro\thugpro_menu.qb`) exists in the newer build but not in the mod.

Practical consequence: **installing the mod rolls back not just the balance but all 102 of those
scripts to their 2022 versions.** For single player and local sessions that is usually fine, but
back up `data\pre\` first, and if you play multiplayer on a current version, be aware that the
script mismatch can put you out of sync with the host.

The mod touches no value other than the four `Lean_Acc` entries. Everything else in the 2022
`physics.qb` is vanilla.

### How it was built

THUG Pro's scripts live inside a `.prx` archive as compiled QB bytecode, so editing takes three steps:

1. **Decompile.** `physics.qb` from the archive → the text source `physics.txt`, using
   `roqq.exe -d physics.qb`. `physics.qb_table.qbi` (the QB key table) has to sit next to it —
   without it you get raw hashes instead of names.
2. **Edit.** Four `$Lean_Acc$` lines in `physics.txt` — in `ManualParams`, `LipParams`,
   `SkitchParams` and `GrindParams`. Nothing else is touched.
3. **Build.** `roqq.exe -c physics.txt` → `physics.qb`, then `PreTool.exe` to open
   `thugpro_qb.prx`, locate `qb\game\skater\physics.qb`, `Replace` it and save the archive.

Checks confirming the release really contains this source:

- `roqq.exe -d physics.qb` reproduces `physics.txt` **byte for byte** — the source is complete.
- `roqq.exe -c physics.txt` yields a `.qb` whose **first 15,131 bytes match** the released
  `physics.qb`; the remainder of the file is the appended QB key table.
- The archive's internal record reports an uncompressed size of **27,808** — exactly the size of
  the released `physics.qb`.
- Packing the released `physics.qb` with the same LZSS algorithm `PreTool.exe` uses produces
  **exactly the 11,974 bytes** stored in the `.prx` — zero differences. Unpacking them returns the
  file byte for byte.

Diffing against vanilla from the same build (THUG Pro, `physics.qb` 27,809 bytes) through the same
key table gives **794 lines in both and exactly 4 differing lines** — the four `Lean_Acc` entries.
That is the entire mod.

### Credits

- **Matrix717** — balance tuning. The 47 / 47 / 50 / 90 values were found together with him.
- **kennythps** — decompiling, editing, compiling and packaging.

The tools used here were written by other people and are not part of this repository:
`PreTool.exe` (Alabaster, Inc., 2011) for `.prx`/PRE editing, `roq cmd.exe` (2007) as a GUI wrapper,
and `roqq.exe` (2017) as the QB compiler/decompiler. Credit to their authors — without them there
would be no THUG Pro modding at all. If you want to build this yourself, get those tools from their
authors and use them at your own risk.

### Repository layout

```
physics.txt             the full script source (794 lines) — the main artifact
physics.qb_table.qbi    QB key table, required to compile and decompile
README.md               this file
LICENSE                 MIT
```

The compiled `physics.qb` and the ready-to-use `thugpro_qb.prx` are not tracked in git — they are
attached to the release. That keeps the repository light and the diffs readable.

### License

MIT — see [LICENSE](LICENSE). Take it, change it, fork it, publish your own builds.
The original THUG Pro scripts belong to their respective rights holders; what is distributed here
is the modified parameter and the text describing it.
