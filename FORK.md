# belomaxorka/ReGameDLL_CS fork

Upstream: https://github.com/rehlds/ReGameDLL_CS (merges PRs with squash).

```
master   — mirror of rehlds/master, own commits are forbidden
fix/...  — topic branches off master; PRs to rehlds are opened from them
feat/...
release  — master + cherry-picks of the topic branches (my build); linear history, force-pushed
```

Service files (FORK.md, .github/workflows/release.yml) live only on release.

## Versioning

The fork build is named YaGameDLL_CS (Yet Another GameDLL); the version
suffix is `yagd`.

Upstream numbers its builds as `MAJOR.MINOR.MAINT.<commit count>`; on release
the cherry-picks inflated the counter, so a build looked like a nonexistent
upstream dev build. The version scripts (regamedll/version/appversion.sh,
regamedll/msvc/PreBuild.bat) are therefore patched (release-only, keep them
when rebuilding the branch):

- base = commit count at the merge-base with origin/master (fallback: master) —
  exactly the upstream version of that point;
- suffix `+yagd.<N>` = number of fork commits on top.

Example: `5.30.0.814+yagd.17` = upstream `5.30.0.814` + 17 fork commits
(`game_version` prints this string; a trailing `+m` means uncommitted local
changes). Tagged CI builds are releases and carry no `-dev`; local and dev
builds keep it (`5.30.0.814-dev+yagd.17`) — the scripts check
`GITHUB_REF_TYPE == tag`. On master and on topic branches without own commits
the version matches upstream exactly (no suffix). The release changelog states
the upstream base on a separate line (version + commit + date) and links the
upstream diff since the previous release; the zip name and the release title
(`YaGameDLL_CS <version>`) carry the full fork version.

## Cutting a release

CI publishes releases only for pushed tags matching `v*` (release.yml).
Pushes to the release branch produce dev builds instead (dev-build.yml:
build + tests + artifacts, no publication).

1. Make sure `release` is pushed, in the desired state, and its Dev Build is
   green.
2. Find out the version: build locally and take it from the PreBuild output
   (`Updating appversion.h, new version is "5.30.0.814-dev+yagd.17"`) or from
   `game_version`. The tag mirrors it as `v<base>-yagd.<N>` (drop `-dev`,
   `+` is replaced with `-` — awkward in tag names):

   ```bash
   git tag v5.30.0.814-yagd.17 release
   git push origin v5.30.0.814-yagd.17
   ```

3. CI builds Win/Linux, runs unit tests and test demos, then publishes a
   release for the tag (marked Latest): zip with mp.dll/cs.so/configs and a
   changelog. The tagged build carries no `-dev` in the version. The changelog
   contains the upstream base (version + commit + date), a compare link with
   upstream changes since the previous release, and the list of fork commits
   ("Changelog").

To re-release the same tag: delete the release and the tag on GitHub, then
tag and push again. Old releases stay in the history — one release per tag.

## Release policy

- Tag a release after every meaningful, server-tested change-set: a new
  cvar/fix cherry-picked into release, or an upstream sync. Releases are
  cheap and each one is pinned to its tag forever.
- Sync with upstream (see below) when: starting a new topic branch, upstream
  merges one of our PRs, or upstream lands fixes we want. After a sync,
  test on the server, then tag.
- The version tells the story: base bump = upstream sync, `yagd.<N>` bump =
  fork changes. No separate changelog bookkeeping is needed.

## Syncing with upstream

```bash
git checkout master && git fetch upstream && git merge --ff-only upstream/master && git push origin master
git rebase master release && git push -f origin release
```

If `--ff-only` fails, own commits leaked into master; move them out to a branch.

## After rehlds accepts a PR

```bash
git branch -D fix/accepted && git push origin --delete fix/accepted
```

On the next rebase of release onto master the cherry-pick becomes a duplicate
of the upstream squash commit — git usually drops it automatically; on
conflict use `git rebase --skip`.

## CI (.github/workflows/, release branch only)

- release.yml — tag push `v*`: Win/Linux build + unit tests + test demos +
  release publication.
- dev-build.yml — push to release (and workflow_dispatch): same build + tests,
  artifacts only (README "Dev builds" points here).

No signing/GPG — those secrets belong to rehlds. The upstream build.yml fails
on mirror pushes of master; disable it once:

```powershell
Invoke-RestMethod -Method Put -Headers @{Authorization="token <PAT>"} `
  -Uri "https://api.github.com/repos/belomaxorka/ReGameDLL_CS/actions/workflows/build.yml/disable"
```

## Local ignore (`.git/info/exclude`, restore after a re-clone)

```
regamedll/msvc/ReGameDLL/
dep/cppunitlite/msvc/cppunitlite/
CLAUDE.md
```

## Release branch composition

- `fix/hostage-ai-stale-nav-on-mapchange` — CZ hostages freeze after a map change (PR #1162)
- `feat/knife-wall-sparks` — `mp_knife_wall_sparks` (PR #1163)
- `feat/show-bomb-timer` — `mp_show_bomb_timer` (PR #1164)
- `fix/sharedparse-bounds` — buffer overflow in SharedParse() (PR #1165)
- `feat/hint-messages-cvar` — `mp_show_hintmessages` (PR #1166)
- `fix/knife-hit-detection` — `mp_knife_hit_detection`, aim-based knife hit
  registration (issue #1154; PR to rehlds not opened yet)

Release-only commits (not from topic branches; preserve them via cherry-pick
when rebuilding the branch, like the service files):
- `docs: fix cvar defaults and document mp_chat_loc_fallback` — upstream doc
  bugs (sv_allchat=0, ff_damage_reduction_other=0.25, added
  mp_chat_loc_fallback). Could later become a separate PR to rehlds.
- `ci: fork-aware versioning` + follow-ups — appversion.sh / PreBuild.bat /
  release.yml / dev-build.yml patches (see Versioning / Cutting a release).
  Fork-specific, never goes to rehlds.
- `docs: fork README` — YaGameDLL_CS header, badges and Downloads links point
  to this fork. Fork-specific, never goes to rehlds.

Deferred: hostage stuck fix — `hostage_stuck_fix.patch` on the Desktop.
