# Kaizen Builds

Download host for Kaizen releases, and the place the Linux and Mac packages are built.

## Downloads

Each link always serves the newest version. The Linux and Mac links work from
the first release that includes them (after v2.13).

- **Windows:** [Kaizen-Setup.exe](https://github.com/KaizenMMO/kaizen-builds/releases/latest/download/Kaizen-Setup.exe)
- **Linux (x86-64):** [Kaizen-Linux-x86_64.tar.gz](https://github.com/KaizenMMO/kaizen-builds/releases/latest/download/Kaizen-Linux-x86_64.tar.gz).
  Extract it and run `Kaizen/Kaizen` from an X11 session, with `tesseract-ocr`
  and `tesseract-ocr-eng` installed (Kaizen shows the command for your system
  if they are missing). `README-LINUX.txt` inside has the details.
- **Mac (Apple Silicon, preview):** [Kaizen-Mac-AppleSilicon.dmg](https://github.com/KaizenMMO/kaizen-builds/releases/latest/download/Kaizen-Mac-AppleSilicon.dmg).
  M1 or newer, macOS 13 or later. Drag Kaizen onto Applications. It is not
  registered with Apple, so the first open needs System Settings > Privacy &
  Security > **Open Anyway**. Needs `brew install tesseract` and the Screen
  Recording permission. `README-MAC.txt` inside has the details.

## Linux and Mac packages for every release

`linux-release.yml` and `mac-release.yml` build Linux and Mac for a
**published** release and attach them to that same release. The source repo's
`packaging/publish_release.py` starts both after every Windows publish. To run
one yourself:

```sh
gh workflow run linux-release.yml --repo KaizenMMO/kaizen-builds \
  -f source_sha=<pushed commit in KaizenMMO/Kaizen> -f release_tag=v2.14
gh workflow run mac-release.yml --repo KaizenMMO/kaizen-builds \
  -f source_sha=<pushed commit in KaizenMMO/Kaizen> -f release_tag=v2.14
```

It checks the source's version matches the tag, runs the suite, compiles the
protected modules with Nuitka, builds on ubuntu-22.04 (old glibc, so the build
runs on every current distribution), runs `Kaizen --self-test` on the frozen
app, uploads `Kaizen-Linux-x86_64.tar.gz` and its `.sha256` under that fixed
name, and adds a Linux section to the release notes. The Mac workflow does the
same on macos-14 (Apple Silicon): `packaging/build_mac.py` builds `Kaizen.app`,
signs it ad-hoc (there is no Apple Developer account), runs the self-test, and
uploads `Kaizen-Mac-AppleSilicon.dmg`.

`-f tests_only=true` runs only the tests and touches no release (the
`release_tag` input is then ignored, but must still be given).

**Build logs here are public; the source is not.** The test step prints
failures as test names and one-line reasons only (`--tb=no`). Keep secrets and
source out of anything this workflow prints.

The source repo is private. The workflow reads it with a read-only deploy key
stored here as `KAIZEN_SOURCE_SSH_KEY`. To set it up or replace it:

```sh
ssh-keygen -t ed25519 -N "" -C "kaizen-builds CI" -f kz_deploy
gh repo deploy-key add kz_deploy.pub --repo KaizenMMO/Kaizen --title "kaizen-builds CI (read-only)"
gh secret set KAIZEN_SOURCE_SSH_KEY --repo KaizenMMO/kaizen-builds < kz_deploy
rm kz_deploy kz_deploy.pub
```

The Windows package is not built here. It is built on the developer's machine
and uploaded to the release by `packaging/publish_release.py`.
