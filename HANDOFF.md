# Handoff: phone (Termux) → PC

Last updated: 2026-11-12 · Plan and PR breakdown: [PLAN.md](PLAN.md)

This file covers **where the work stands and how to pick it up on a PC**. *What* to build and *why* lives in PLAN.md.

## Where things stand

| Item | State |
|---|---|
| Issue [#1](https://github.com/doccstat/pixel-launcher-folder/issues/1) | Owner approved the plan, including Gradle + Material 3. They asked that everything handle multiple work profiles. |
| **PR 1** — [#2](https://github.com/doccstat/pixel-launcher-folder/pull/2) | **Draft.** Branch `ui/preview-fallback`, commit `6251725`, based on upstream `20dc223`. |
| PR 1 CI | **Blocked.** The run shows `action_required` because first-time contributor workflows need the owner's approval. |
| PR 1 device test | **Not done.** |
| PRs 2–5, Phase 2 | Not started. |

### PR 1 — what's left
1. Get a built APK (see [Building](#building-on-the-pc)), or wait for the owner to approve CI.
2. Test it on the Pixel 8a (see [Testing PR 1](#testing-pr-1)).
3. Add before/after screenshots to the PR and tick its checklist.
4. Mark it ready: `gh pr ready 2 -R doccstat/pixel-launcher-folder`.

## Repos and remotes

| Remote | URL | Use |
|---|---|---|
| `origin` | https://github.com/doccstat/pixel-launcher-folder | Upstream. Fetch from here and target PRs here. |
| `fork` | https://github.com/ChaliceChore/pixel-launcher-folder | Your fork. Push branches here. |

**Fresh setup on the PC:**
```sh
git clone https://github.com/doccstat/pixel-launcher-folder
cd pixel-launcher-folder
git remote add fork https://github.com/ChaliceChore/pixel-launcher-folder.git
git fetch fork
git switch -c ui/preview-fallback --track fork/ui/preview-fallback
```

**Git identity.** Check it before every commit; never use your real email:
```sh
git config user.name   # ChaliceChore
git config user.email  # 22472867+ChaliceChore@users.noreply.github.com
```

**Each new PR branch** starts from up-to-date upstream:
```sh
git fetch origin && git switch -c ui/<short-name> origin/main
```

## Building on the PC

Phase 1 has no Gradle. `build.py` calls the SDK tools directly. CI pins these versions, so install the same ones:

- **JDK 21** (CI uses Temurin 21). Set `JAVA_HOME`.
- **Android SDK** with `platforms;android-37.0` and `build-tools;36.0.0`. Set `ANDROID_HOME`. Install them with `sdkmanager 'platforms;android-37.0' 'build-tools;36.0.0'`.
- **Python 3.**

```sh
python3 -m unittest discover -s tests/build_tests -v   # build/signing tests (all should pass on a PC)
python3 build.py                                        # → dist/PixelLauncherFolders.apk
```

Notes:
- **Why the phone couldn't build:** in Termux, `javac` compiled fine but the SDK's `d8` isn't runnable on aarch64, so the APK step failed. A PC doesn't have this problem.
- **About the 7 failing tests:** they fail in Termux only because the release-workflow tests need `/usr/bin/env`. They fail on clean `main` too, so they aren't regressions. On a PC all 22 should pass.
- **About the signing key:** `build.py` creates a debug signing key at `~/.android/pixel-launcher-folders.keystore` if none exists. Keep that file and reuse it, so later test builds install as updates over each other.

## Installing a test build — read first

⚠️ Your own build, or the CI APK, is **signed with a different key** than the owner's release you have installed. Android refuses to install it as an update, so:

1. Open the module's Settings → **Export folders** to back up your folder setup.
2. Uninstall the release module, then install the test APK:
   ```sh
   adb install dist/PixelLauncherFolders.apk
   ```
3. In LSPosed, re-enable the module, scope it to Pixel Launcher only, and force-stop or restart Pixel Launcher.
4. Choose the same backup folder and use **Import folders**.

When testing is done, reverse the same steps to go back to the release build.

## Test device

- **Phone:** Pixel 8a (not foldable). Rooted, with LSPosed and the module installed. The owner tests foldable layouts on their Pixel Fold.
- **Existing extra profile:** "Pixel Launcher Folders" (user 10, serial 16), created with App Clone. It's a **clone** profile, so the module ignores it. Don't use it for work-profile tests.
- **Work profiles for testing:** create them with the owner's [gist](https://gist.github.com/Akianonymus/1fb3c040080f79e7a015c7948e874499) root method, needed from PR 2 onward:
  ```sh
  adb shell su -c 'setprop fw.sys.max_profiles 10'     # after the gist's resetprop/restart steps
  adb shell su -c 'pm create-user --profileOf 0 --managed TestWork1'
  adb shell pm list users                               # note the new user id
  adb shell su -c 'pm remove-user <id>'                 # clean up afterwards
  ```

## Testing PR 1

| Case | Expected result |
|---|---|
| Folder with installed apps | Four mini icons, the same as before |
| Folder whose apps are all uninstalled ("Hide unavailable apps" on) | Folder glyph in the circle, in both the drawer and the settings list |
| Empty folder | Folder glyph |
| Opening the settings app | No glyph flashes before icons load |
| Logs | `adb logcat -s PixelLauncherFolders` shows a `W` line for each icon or profile load that fails |

Check light and dark mode, and take before/after screenshots of the drawer and the settings list for the PR.

## Next up

After PR 1 is ready: **PR 2, profile names for multiple work profiles** (see PLAN.md). Start by finding which Android 15+ API returns another profile's name without privileged permissions. Then create one, and later two, gist work profiles on the Pixel 8a to test.

## Where these notes live

The notes live on the **`contrib-notes`** branch of the fork, https://github.com/ChaliceChore/pixel-launcher-folder/tree/contrib-notes. It's an orphan branch: it shares no history with the module code, so it never ends up in an upstream PR. Never merge it into a PR branch, and never open a PR from it.

```sh
# get the notes on the PC, beside your working clone
git clone -b contrib-notes --single-branch https://github.com/ChaliceChore/pixel-launcher-folder pixel-launcher-folder-notes
# after editing PLAN.md / HANDOFF.md
git -C pixel-launcher-folder-notes commit -am "notes: update" && git -C pixel-launcher-folder-notes push
```

| File | Purpose |
|---|---|
| [PLAN.md](PLAN.md) | Approved plan: decisions, ground rules, every PR's scope |
| HANDOFF.md | This file: current state and how to continue |
| [issue1-comment-draft.md](issue1-comment-draft.md), [issue1-reply2-draft.md](issue1-reply2-draft.md) | Comments already posted on issue #1; kept for reference |

The module code for PR 1 is on the fork's `ui/preview-fallback` branch, not here.
