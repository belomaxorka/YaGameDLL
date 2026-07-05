# Форк belomaxorka/ReGameDLL_CS

Апстрим: https://github.com/rehlds/ReGameDLL_CS (мержит PR сквошем).

```
master   — зеркало rehlds/master, свои коммиты запрещены
fix/...  — тематические ветки от master; из них PR в rehlds
feat/...
release  — master + черри-пики тематических веток (моя сборка); история линейная, пушится с -f
```

Служебные файлы (FORK.md, .github/workflows/release.yml) живут только на release.

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

Отложено: stuck-фикс заложников — патч `hostage_stuck_fix.patch` на рабочем столе.
