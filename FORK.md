# Форк belomaxorka/ReGameDLL_CS

Апстрим: https://github.com/rehlds/ReGameDLL_CS (мержит PR сквошем).

```
master   — зеркало rehlds/master, свои коммиты запрещены
fix/...  — тематические ветки от master; из них PR в rehlds
feat/...
release  — master + черри-пики тематических веток (моя сборка); история линейная, пушится с -f
```

Служебные файлы (FORK.md, .github/workflows/release.yml) живут только на release.

## Версионирование

Имя сборки форка — YaGameDLL_CS (Yet Another GameDLL), суффикс версии — `yagd`.

Апстрим нумерует билды как `MAJOR.MINOR.MAINT.<число коммитов>`; на release
черри-пики раздували счётчик, и версия выглядела как несуществующий апстримовский
dev-билд. Поэтому скрипты версии (regamedll/version/appversion.sh,
regamedll/msvc/PreBuild.bat) пропатчены (release-only, беречь при пересборке):

- база = число коммитов до merge-base с origin/master (fallback: master) —
  в точности апстримовская версия этой точки;
- суффикс `+yagd.<N>` — число наших коммитов поверх.

Пример: `5.30.0.814-dev+yagd.15` = апстрим `5.30.0.814-dev` + 15 коммитов форка
(`game_version` показывает эту строку; `+m` в конце — незакоммиченные правки).
На master и в тематических ветках без своих коммитов версия совпадает
с апстримовской (суффикс не добавляется). Ченжлог релиза указывает
апстрим-базу отдельной строкой (версия + коммит + дата), имя zip и заголовок
релиза (`YaGameDLL_CS <версия>`) содержат полную версию форка.

## Синхронизация с апстримом

```bash
git checkout master && git fetch upstream && git merge --ff-only upstream/master && git push origin master
git rebase master release && git push -f origin release
```

Если `--ff-only` не прошёл — в master попали свои коммиты; унести их в ветку.

## Когда rehlds принял PR

```bash
git branch -D fix/принятая && git push origin --delete fix/принятая
```

При следующем rebase release на master свой черри-пик станет дублем сквош-коммита —
git обычно сам его выкидывает; если конфликт — `git rebase --skip`.

## CI (Release, .github/workflows/release.yml)

Пуш в release: сборка Win/Linux + тесты + перевыпуск релиза с тегом `latest`
(zip с mp.dll/cs.so/конфигами и ченжлогом — коммиты поверх master).
Подписи/GPG нет — секреты rehlds. Апстримовый build.yml упадёт на первом
зеркальном пуше master; после этого отключить его:

```powershell
Invoke-RestMethod -Method Put -Headers @{Authorization="token <PAT>"} `
  -Uri "https://api.github.com/repos/belomaxorka/ReGameDLL_CS/actions/workflows/build.yml/disable"
```

## Локальный ignore (`.git/info/exclude`, восстановить после переклона)

```
regamedll/msvc/ReGameDLL/
dep/cppunitlite/msvc/cppunitlite/
CLAUDE.md
```

## Состав release

- `fix/hostage-ai-stale-nav-on-mapchange` — CZ-заложники замирают после смены карты (PR #1162)
- `feat/knife-wall-sparks` — `mp_knife_wall_sparks` (PR #1163)
- `feat/show-bomb-timer` — `mp_show_bomb_timer` (PR #1164)
- `fix/sharedparse-bounds` — переполнение буфера в SharedParse() (PR #1165)
- `feat/hint-messages-cvar` — `mp_show_hintmessages` (PR #1166)
- `fix/knife-hit-detection` — `mp_knife_hit_detection`, регистрация ударов ножа
  по прицелу (issue #1154; PR в rehlds пока не открыт)

Release-only коммиты (не из тематических веток; при пересборке сохранять черри-пиком,
как служебные файлы):
- `docs: fix cvar defaults and document mp_chat_loc_fallback` — баги доков апстрима
  (sv_allchat=0, ff_damage_reduction_other=0.25, добавлен mp_chat_loc_fallback).
  Позже можно оформить отдельным PR в rehlds.
- `ci: fork-aware versioning` — патч appversion.sh/PreBuild.bat/release.yml
  (см. «Версионирование»). В rehlds не пойдёт — специфично для форка.

Отложено: stuck-фикс заложников — патч `hostage_stuck_fix.patch` на рабочем столе.
