# Releases (moults31/nhl24pc)

This fork publishes **tag-triggered GitHub Releases** with a ready-to-run Windows zip built in CI (`tools/make_release.ps1`). Upstream [HajDEO/nhl24pc](https://github.com/HajDEO/nhl24pc) does not use GitHub Releases; it relies on the in-repo `release/` folder and local assembly.

## What gets published

- **Asset:** `NHL-24-PC-vX.Y.Z.zip` — `launcher.exe`, `nhl24.exe`, VC++ runtime DLLs, launcher art, clean `user\` template (patches, default controls). **No PKG or extracted game.**
- **Checksum:** `SHA256SUMS` for the zip.
- **Traceability:** Release notes include the full git commit SHA for the tag.

## Semver: where it lives

The shipped version is **`NHLPORT_VERSION`** in `src/common/port_version.h`. It drives:

- Window title and launcher UI version
- Windows `FILEVERSION` / `PRODUCTVERSION` on `nhl24.exe`
- Release zip filename: `NHL-24-PC-vX.Y.Z.zip`

The git tag **must** match that string with a `v` prefix: tag `v0.8.4` ↔ `#define NHLPORT_VERSION "0.8.4"`. The release workflow fails if they differ.

## How to choose the next version

Use [semver](https://semver.org/) on **`NHLPORT_VERSION`**, aligned with upstream when you are tracking their line:

| Change | Bump | Example |
|--------|------|---------|
| Upstream tagged the same logical port release you merged | Match upstream **patch** (or minor if they did) | Upstream stays on 0.8.3 → you stay on **0.8.3** only for the **first** release cut from that commit; see fork note below |
| Bugfix or small fork-only fix after a release | **PATCH** | 0.8.3 → **0.8.4** |
| New user-visible feature (launcher option, new title ID, meaningful gameplay fix) | **PATCH** (port uses 0.x.y; treat like product semver) | 0.8.4 → **0.8.5** |
| Large milestone (maintainer would call it 0.9 / 1.0) | **MINOR** / **MAJOR** | Rare; coordinate with upstream naming in commit messages |

**Fork-only commits after upstream `v0.8.3`:** If upstream never tags but you have extra fixes (EUR PKG, 1.6 patches, README), bump **patch** on each **published** zip you want Chonxy to serve, even if upstream still says 0.8.3 in source. Example: first GitHub Release from fork at 0.8.3+fork fixes → use **0.8.4** (or **0.8.3-moults31.1** is awkward for the tag check; prefer bumping patch in `port_version.h`).

## Cut a release

1. **Set version** in `src/common/port_version.h`:
   - `NHLPORT_VERSION_MAJOR` / `MINOR` / `PATCH`
   - `#define NHLPORT_VERSION "X.Y.Z"`
2. **Merge** the version bump (and any changes) to `main`.
3. **Tag** on `main` (annotated tags are fine):
   ```bash
   git tag vX.Y.Z
   git push origin vX.Y.Z
   ```
   For this fork, `origin` is usually `moults31` (`git push moults31 vX.Y.Z`).
4. **Wait** for the [Release (Windows package)](https://github.com/moults31/nhl24pc/actions) workflow on that tag.
5. **Download** from the GitHub Release page (or `gh release download -R moults31/nhl24pc vX.Y.Z`).
6. **Host on Unraid** (Chonxy): copy the zip to `/mnt/user/media/chonxy/production/archives/` with the **same filename** as the asset, then point the NHL 24 Extras page at `/downloads/archives/NHL-24-PC-vX.Y.Z.zip`.

## Upstream version awkwardness

| Situation | Practical approach |
|-----------|-------------------|
| Upstream has **no git tags / no GitHub Releases** | Your tags on `moults31/nhl24pc` are the only official release objects for your household; semver in `port_version.h` may match upstream **commit messages** (e.g. “v0.8.3”) without a upstream tag to point at. |
| Same `NHLPORT_VERSION` as upstream, different code | Do **not** re-tag `v0.8.3` if upstream might tag later; bump to **0.8.4** (or next patch) for your first published Release. |
| You merge upstream `main` | Reconcile `port_version.h` with upstream’s bump if they changed it; then tag your next release from the merged `main`. |
| Upstream ships a new `0.8.4` in source only | After merging, set `port_version.h` to **0.8.4**, tag **`v0.8.4`** on **your** fork’s merge commit (your artifact ≠ theirs; traceability is commit SHA). |
| `release/README.txt` still mentions HajDEO / old update guidance | CI package uses `make_release.ps1` text; fix upstream or fork README in source when guidance changes—Release body on GitHub is generated separately. |

GPL: you distribute binaries built from this fork; keep **source** available via the repo and cite [HajDEO/nhl24pc](https://github.com/HajDEO/nhl24pc) lineage in release notes or project README as you already do in docs.
