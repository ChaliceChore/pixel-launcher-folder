# Pixel Launcher Folders — UI/UX contribution plan

Contributor: ChaliceChore · Upstream: [doccstat/pixel-launcher-folder](https://github.com/doccstat/pixel-launcher-folder) · Base: `main` @ `1063b67` (v0.3.2)
Discussion: [issue #1](https://github.com/doccstat/pixel-launcher-folder/issues/1)
Current progress and PC setup: [HANDOFF.md](HANDOFF.md)

## Status (2026-11-12)

- **2026-10-31:** The owner agreed to accept PRs.
- **2026-11-09:** We [proposed](https://github.com/doccstat/pixel-launcher-folder/issues/1#issuecomment-6749231914) moving the settings app to Gradle + Material 3.
- **2026-11-12:** The owner [replied](https://github.com/doccstat/pixel-launcher-folder/issues/1#issuecomment-6814280154): "Generally looks good to me… feel free to work on whatever you like!" They asked us to:
  1. keep the work scalable to **multiple work profiles** ([gist](https://gist.github.com/Akianonymus/1fb3c040080f79e7a015c7948e874499));
  2. look into the **empty preview circle** (the "Google" folder with 40 apps).
- **2026-11-12:** We [replied](https://github.com/doccstat/pixel-launcher-folder/issues/1#issuecomment-6818594152), committing to multi-profile-safe UI and to PR 1.
- **2026-11-12:** PR 1 opened as a draft: [#2](https://github.com/doccstat/pixel-launcher-folder/pull/2). See HANDOFF.md for what's left.

## Decisions (from plan review)

- The owner's reply counts as a **yes to Gradle + Material 3**. All settings-app UI work (toolbar, settings page, folder list, editor, picker, reorder) moves to Phase 2 and is built in Material 3. Phase 1 keeps only the work that Material 3 won't replace.
- **Folder deletion:** the ⋮ menu is replaced by a **bin icon**, and the **confirmation dialog stays**. There is no undo bar.
- **"Move earlier/later" goes away.** Drag-and-drop replaces it.
- **Test device:** the contributor's Pixel 8a, which is enough for every PR. The extra-device idea is dropped. The Pixel 8a isn't foldable, so we ask the owner to sanity-check fold/unfold layouts on their Pixel Fold.
- Icon-size and grid settings for the popup go on the **Roadmap**.

## Ground rules (apply to every PR)

- **One concern per PR.** Each one is small enough to review in a single sitting, and none depends on another unless noted.
- **Never assume a single work profile.** There can be any number of managed profiles, so identify them by serial and label them by name. Never use `"work"` alone as a key or a label.
- **The launcher side stays framework-only, always.** `FolderRow`, `LauncherHook`, `LauncherDrag` and the `Ui.java` helpers they use run inside Pixel Launcher's process, where AndroidX/Material can't load safely. Material 3 is for the settings app only.
- **Match the house style** in framework code. Use the `Ui.java` helpers and don't reformat lines you aren't changing.
- **No version bumps or release edits.** versionCode/versionName and releases belong to the author.
- **Testing.** CI builds a development APK on every PR. Install it on the Pixel 8a, enable the module, restart Pixel Launcher, and check light and dark mode. Attach before/after screenshots to every PR.
- **Git.** Work in a fork, one branch per PR (`ui/<short-name>`) cut from up-to-date `upstream/main`. Author and committer: `ChaliceChore <22472867+ChaliceChore@users.noreply.github.com>`.

## Test setup (Pixel 8a)

- **Profiles on the device:** Owner (user 0), and "Pixel Launcher Folders" (user 10, serial 16) created with the owner's [App Clone](https://github.com/doccstat/appclone).
- **Clone profiles get no special handling.** That profile is a clone profile, so the module ignores it, and we keep it that way. No code changes target clone profiles.
- **Work profiles follow the owner's gist.** For testing, we create managed work profiles on the Pixel 8a with the [gist](https://gist.github.com/Akianonymus/1fb3c040080f79e7a015c7948e874499)'s root method: raise `fw.sys.max_profiles`, then run `pm create-user --profileOf 0 --managed <name>`. One profile tests the single-work-profile case, two test the multi-profile case. Each test profile is removed afterwards with `pm remove-user <id>`.

---

## Phase 1 — Framework-only PRs (no Gradle)

These touch launcher-side code or shared logic that Material 3 won't replace, so they go first and stay small.

| # | PR | Area | Size | Risk |
|---|----|------|------|------|
| 1 | Preview placeholder + icon-failure logging | settings + launcher | S | Low |
| 2 | Profile names for multiple work profiles | shared + launcher | S | Low |
| 3 | Dynamic-color tokens for hard-coded surfaces | Ui.java | S | Low |
| 4 | Folder popup open/close animation | launcher | M | Med |
| 5 | Accessibility pass on drawer folder cell | launcher | S | Low |

### PR 1 — Preview placeholder + icon-failure logging

**Problem:** The repo's own screenshot `docs/screenshots/drawer-folders.png` shows the "Google" folder (40 apps) with an empty preview circle. It hasn't been reproduced on our device. Icon failures are swallowed silently by `catch (RuntimeException ignored)`, so there's nothing to diagnose with, and an empty preview has no placeholder.

**Change**
- In `MainActivity.folderPreview()` and `FolderRow.render()`: when none of the first four icons load, draw a folder glyph inside the circle instead of leaving it blank. Reuse `Ui.FolderGlyph` by making it package-visible.
- Replace the silent catches around `icon()` calls with `Log.w("PixelLauncherFolders", …)`, including the app key.
- The PR description says this is a mitigation plus diagnostics, not a confirmed root-cause fix.

**Files:** `MainActivity.java`, `FolderRow.java`, `Ui.java`.
**Test:** a folder with installed apps, a folder whose apps are all uninstalled, and an empty folder. Check that logcat shows the warnings.

### PR 2 — Profile names for multiple work profiles

**Problem:** Every managed profile is labelled "Work" (`profileName()`, the hard-coded strings in `FolderRow.open()`, and `Profiles.toString()`). With two work profiles the folder list shows two identical "WORK" sections, the picker shows "Work · profile 28" vs "Work · profile 31", and the popup subtitle can't tell them apart.

**Change**
- Add `Profiles.label(Context)`. It returns the profile's own name (e.g. "Island") when Android exposes it to the app. Otherwise it returns "Work" if there is only one work profile, or "Work 2", "Work 3", … ordered by serial. Personal stays "Personal".
- Use it everywhere a profile is named: folder-list section titles, the editor's profile picker (dropping the raw serial), the Preview buttons, and the folder popup subtitle.
- First check which API returns another profile's name without privileged permissions on Android 15+. If none does, use only the numbered fallback.
- This also gives Phase 2 a single label source.

**Files:** `Profiles.java`, `MainActivity.java`, `FolderRow.java`.
**Test:** no work profile; one gist-created work profile (labels unchanged); two gist-created work profiles (see Test setup). Unit-test the numbering logic in `tests/`.

### PR 3 — Dynamic-color tokens for hard-coded surfaces

**Problem:** `Ui.surfaceMuted()` and `Ui.dividerColor()` use fixed RGB values, so the launcher popup and the drawer cells ignore the wallpaper's Material You palette.

**Change:** map them to `system_neutral2_*` tokens, keeping the current RGB values as fallbacks. Leave `pickerActive`/`pickerElsewhere` alone: they are deliberately fixed.

**Files:** `Ui.java` only.
**Test:** two or three wallpapers, in light and dark mode. Check the popup and the drawer cell. Contrast must not regress.

### PR 4 — Folder popup open/close animation

**Problem:** The launcher popup appears and disappears instantly.

**Change:** scale and fade from the tapped cell on open (about 200 ms, emphasized easing), and reverse on close. Respect the system animation scale. **Do not** animate during a home-drag handoff, because the panel is set to `alpha=0` there on purpose.

**Files:** `FolderRow.open()`, `dismissFolderPopup()`.
**Test (Pixel 8a):** open, close, tap outside, launch an app, long-press drag to home, split-screen drag, and animations turned off in Developer options. Ask the owner to check the taskbar drawer on the unfolded Pixel Fold.

### PR 5 — Accessibility pass on the drawer folder cell

**Problem:** Long-pressing a folder cell silently opens Settings, and TalkBack users don't learn that this is possible. The content description says "1 apps".

**Change:** add an `AccessibilityAction` labelled "Edit folders" and an `ACTION_CLICK` label of "Open folder". Fix the singular/plural wording. Include the PR 2 profile label when there is more than one profile.

**Files:** `FolderRow.render()`.
**Test:** TalkBack on the Pixel 8a.

---

## Phase 2 — Gradle + Material 3 settings app

This covers the settings app (`MainActivity`) only. Each PR builds on the previous one, so they're merged in order.

| # | PR | Size | Risk |
|---|----|------|------|
| 2a | Gradle build (no UI changes) | M | Med |
| 2b | Material 3 theme + app shell + How-to section | M | Low |
| 2c | Settings page as section "islands" | M | Low |
| 2d | Folder list: bin-icon delete + drag-to-reorder | M | Med |
| 2e | Folder editor dialog redesign | M | Low |
| 2f | App picker filters + selected-first | M | Low |
| 2g | Remove `build.py` (owner's call) | S | Low |

### 2a — Gradle build

**Why it's needed, and why it's a separate PR from 2b:** Material 3 is an AndroidX library, and the current `build.py` can't pull in or package libraries, so Gradle is required. Merging 2a into 2b is possible, but it's kept separate for three reasons:
- **Easier review.** A build-only PR can be checked against one question: is the APK equivalent to the current one? Mixing it with a UI redesign means the author reviews CI, signing and release changes and new screens at the same time.
- **Signing risk.** If the Gradle signing setup is wrong, users can't update over their existing install. That risk should be caught on its own, before any UI work depends on it.
- **No dead time.** Once 2a merges, 2b–2f can each build on it without carrying build changes along.

If the owner prefers fewer PRs, 2a and 2b can still be combined, with the Gradle work as the first commit.

- Add a Gradle build that produces the same APK as `build.py`: same package, same signing key, same release flow, and the Xposed metadata and `assets/xposed_init` intact.
- Update `.github/workflows/*` and `tests/build_tests` to match. Keep `build.py` working until 2g.
- No UI changes. Its success measure is an APK equivalent to the current one, which makes it easy to review.

### 2b — Material 3 theme, app shell and How-to section

- Use `Theme.Material3.DynamicColors.DayNight` with `DynamicColors`.
- **Top app bar:** on the folders home page, the title plus a gear icon (`contentDescription="Settings"`). On the Settings page, a back arrow plus the "Settings" title. This replaces the "Settings"/"Folders" text toggle. Back gesture behavior is unchanged.
- **Move the "Launcher setup" steps out of Settings** into a "How to set up" section on the home page. Make it a dismissible card or an expandable row so it doesn't crowd the folder list once setup is done.

### 2c — Settings page as section "islands"

- Each section (App drawer, Backup & restore, Preview) becomes its own rounded card ("island") with a section header above it.
- Each row shows a **title in a larger font and a subtitle in a smaller font in the same row**, with the switch or chevron at the end (Material list items).
- **Backup row:** drop the raw path such as "Internal storage / Download/Pixel Launcher Folders", which users can't make sense of. Show "Connected" with the folder's display name only (queried via `DocumentsContract`, e.g. "Pixel Launcher Folders"), or no location at all.
- Make the main action stand out: "Choose backup folder" while none is set, otherwise Export. Import stays secondary.
- Preview becomes one row per profile, using the PR 2 labels.

### 2d — Folder list: bin-icon delete + drag-to-reorder

- Replace the ⋮ overflow menu with a **bin icon**. Tapping it opens the existing **confirmation dialog**. "Edit folder" and "Move earlier/later" go away.
- Tapping a row still opens the editor.
- **Drag-to-reorder** with a drag handle (`RecyclerView` + `ItemTouchHelper`), only within a profile group, saving on drop. Drops into another profile's group are rejected.
- TalkBack users get "Move up/Move down" through `AccessibilityAction`s instead of the old menu items.

### 2e — Folder editor dialog redesign

**Problems with today's dialog:**
- Plain fields with section labels in capitals ("FOLDER NAME", "PROFILE").
- The profile spinner shows a raw serial.
- The helper text "A saved folder stays in its original profile." sits below the spinner.
- "Choose apps · 22" is an outlined button that doesn't show which apps are in the folder.
- Save stays enabled when the name is empty and only reports an error after it's tapped.

**New layout:** a Material 3 modal bottom sheet. It sits within thumb reach, and the keyboard pushes it up instead of covering the fields.

```
┌──────────────────────────────────────┐
│               ───                    │  drag handle
│  Edit folder                         │  title (headline small)
│                                      │
│  ┌ Folder name ──────────────────┐   │  outlined text field,
│  │ Utility                    ✕  │   │  clear button,
│  └───────────────────────── 7/48 ┘   │  character counter
│                                      │
│  ┌ Profile ──────────────────────┐   │  exposed dropdown,
│  │ 🔒 Personal                 ▾ │   │  locked when editing
│  └───────────────────────────────┘   │
│                                      │
│  Apps                       22 apps  │  row: title + count
│  (●)(●)(●)(●)(●)(●) +16          ›   │  icon strip, tap → picker
│                                      │
│              [ Cancel ]  [  Save  ]  │  text + filled button
└──────────────────────────────────────┘
```

**Details**
- **Name field:** outlined text field with a 48-character counter and a clear button. The keyboard opens automatically for a new folder. The keyboard's Done key saves when the name is valid.
- **Profile field:** an exposed dropdown using the PR 2 labels, with no serials.
  - Hidden entirely when there is only one profile, because there's nothing to choose.
  - When editing a saved folder, it's locked with a lock icon. Tapping it shows the toast "A saved folder stays in its original profile." This replaces the helper text. A locked field gets no clicks, so the tap is caught on its container.
  - Changing the profile on a new folder clears the app selection, as today, and shows a short note that it did so.
- **Apps row:** shows the first six app icons plus "+N", with the count on the right. The whole row opens the picker (2f). An empty folder shows "No apps yet · Choose apps".
- **Buttons:** Save is a filled button, disabled until the name has text. Cancel is a text button. If anything changed, Cancel or a swipe-down asks "Discard changes?" first.
- **Accessibility:** every field has a proper label, and the icon strip is read aloud as "22 apps, including Android Faker, App Manager, …".

### 2f — App picker filters + selected-first

- Add filter chips under the search field: **All · Selected · Unassigned**, combined with search.
- When the picker opens, list the selected apps first, then the rest A–Z. Don't re-sort while the user is toggling.
- "Unassigned" means not in any folder *of this profile*, which keeps it correct with multiple work profiles.
- Keep the blue/gray/empty membership indicators: they're deliberate and documented in the README.

### 2g — Remove `build.py`

Do this only once the owner is happy with Gradle; they may prefer to keep both. Otherwise remove `build.py` and its tests, and update the README build notes.

---

## Roadmap (future, not scheduled)

- Icon-size and grid settings for the folder popup.
- Custom folder colors or icons.
