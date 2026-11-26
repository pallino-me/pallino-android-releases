# Pallino for Android: releases

The Android app of **Pallino**, an offline-first notes and tasks app in Markdown, with a self-hostable backend that
syncs all your devices.

This repo only holds the builds: every stable version of the app is published here as a GitHub Release, with its APK
and its notes. The app's code is not here.

## Download

**[Latest version →](https://github.com/pallino-me/pallino-android-releases/releases/latest)**

Every release has:

| File | What it is |
| --- | --- |
| `pallino-X.Y.Z-buildN.apk` | The app, signed. |
| `pallino-demo.json` | Optional demo data, to import from **Profile** in the app. |

The *Source code* archives GitHub adds to each release are of this repo (this README), not of the app.

## Install

1. Open the latest release on your Android phone and download the `.apk`.
2. Open it. Android asks you once to allow installing apps from your browser or file manager: allow it.
3. To update, install the new APK over the one you have. Every build is signed with the same key, so your data stays.

## How releases get here

The workflow [`mirror-release.yml`](.github/workflows/mirror-release.yml) copies each stable release of the app's
private repo here, with its title, notes and assets. It doesn't copy the code. Prereleases (the dev builds) stay
private.

- **By itself**: every 30 minutes it looks for releases published after the newest one here and copies them, oldest
  first. When this repo has none yet, it copies only the latest.
- **By hand**: *Actions → Mirror Pallino releases → Run workflow*, with a tag (`v0.16.0`) or empty for the latest. A
  release that is already here gets its notes and assets again. From a terminal:

  ```sh
  gh workflow run mirror-release.yml -R pallino-me/pallino-android-releases            # the latest
  gh workflow run mirror-release.yml -R pallino-me/pallino-android-releases -f tag=v0.16.0
  ```

### Setup

The app's repo is private, so the workflow reads it with a token, the secret `PALLINO_SOURCE_TOKEN`:

1. [Make a fine-grained token](https://github.com/settings/personal-access-tokens/new). Set *Resource owner* to
   `GabrieleGroppo`, set *Repository access* to *Only select repositories* → `pallino`, and set *Permissions* →
   *Contents* to **Read-only**. It needs nothing more.
2. Save it here: `gh secret set PALLINO_SOURCE_TOKEN -R pallino-me/pallino-android-releases`. Paste the token when
   asked.

When the token expires, the runs fail with an error about it. Make a new one and save it the same way.

### Optional: copy a release right away

To copy each release as soon as it's published, without waiting up to 30 minutes, the app's workflow can call this
one. Add a step after *Publish the release* in the app's `build-apk.yml`. It needs a second fine-grained token, set to
*Resource owner* `pallino-me`, repository `pallino-android-releases` and *Contents* **Read and write**, saved there as
`PALLINO_RELEASES_DISPATCH_TOKEN`:

```yaml
      - name: Publish on pallino-me/pallino-android-releases
        if: env.ON_MAIN == 'true'
        env:
          GH_TOKEN: ${{ secrets.PALLINO_RELEASES_DISPATCH_TOKEN }}
        run: |
          gh api repos/pallino-me/pallino-android-releases/dispatches \
            -f event_type=pallino-release -f "client_payload[tag]=v$VERSION"
```

GitHub turns off scheduled workflows in a public repo after 60 days with no commits, and sends an email first.
*Actions → Mirror Pallino releases → Enable workflow* turns it back on.
