# belomaxorka/YaGameDLL (ReGameDLL_CS fork)

Upstream: https://github.com/rehlds/ReGameDLL_CS (merges PRs with squash).
The repo was renamed from ReGameDLL_CS — old URLs redirect.

```
master   — mirror of rehlds/master, own commits are forbidden
fix/...  — topic branches off master; PRs to rehlds are opened from them
feat/...
release  — master + cherry-picks of the topic branches (my build); linear history, force-pushed
```

`release` is the default branch of the repo (front page, new clones, PRs into
the fork). PRs to rehlds are unaffected — their base is always rehlds/master.

Service files (FORK.md, .github/workflows/release.yml) live only on release.

## Versioning

The fork build is named YaGameDLL (Yet Another GameDLL); the version
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
(`YaGameDLL <version>`) carry the full fork version.

## Cutting a release

CI publishes releases only for pushed tags matching `v*` (release.yml).
Pushes to the release branch produce dev builds instead (dev-build.yml:
build + tests + artifacts, no publication).

1. Make sure `release` is pushed, in the desired state, and its Dev Build is
   green.
2. Add an entry for the new version to the fork section at the top of
   CHANGELOG.md (Added / Fixed / Infrastructure; state the upstream base).
3. Find out the version: build locally and take it from the PreBuild output
   (`Updating appversion.h, new version is "5.30.0.814-dev+yagd.17"`) or from
   `game_version`. The tag mirrors it as `v<base>-yagd.<N>` (drop `-dev`,
   `+` is replaced with `-` — awkward in tag names):

   ```bash
   git tag v5.30.0.814-yagd.17 release
   git push origin v5.30.0.814-yagd.17
   ```

4. CI builds Win/Linux, runs unit tests and test demos, then publishes a
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

NEVER use the "Sync fork" button in the GitHub UI on the release branch — for
a diverged branch it offers to discard our commits. Sync only via the commands
above.

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
  -Uri "https://api.github.com/repos/belomaxorka/YaGameDLL/actions/workflows/build.yml/disable"
```

## Local ignore (`.git/info/exclude`, restore after a re-clone)

```
regamedll/msvc/ReGameDLL/
dep/cppunitlite/msvc/cppunitlite/
CLAUDE.md
```

## Inspecting the release branch

Do not keep a manual list of cherry-picked branches in this file — the
actual composition is always:

```bash
git log --oneline master..release
```

Conventions that make that log self-describing:

- Topic-branch cherry-picks keep their original commit messages; the branch
  and PR they came from are recorded in the commit body / GitHub.
- Release-only commits are prefixed `docs:` / `ci:` and never go to rehlds
  (generic doc fixes may later be re-done as a separate PR from master).
  A regular rebase onto master carries them along automatically; when
  rebuilding the branch from scratch, cherry-pick them like the service
  files.
