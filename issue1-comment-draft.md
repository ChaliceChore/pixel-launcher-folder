Thanks for being open to PRs! Before I start anything big, I'd like to check the direction with you.

**Proposal: move the settings app to Gradle + Material 3**

The scope would be limited to `MainActivity` and the settings screens:

- Rebuild the folder list, folder editor, app picker and settings page with Material 3 components: top app bar, list items, filter chips, switches, and a snackbar with undo for deleting a folder.
- Use `DynamicColors` for wallpaper theming instead of the manual `system_accent*` lookups.
- Shrink the release build with R8 so the APK stays small.

What would **not** change:

- `LauncherHook`, `LauncherDrag`, `FolderRow` and everything else that runs inside Pixel Launcher would stay framework-only views. I know Material/AndroidX can't safely load in the launcher's process.
- Behavior, the storage format and backups stay the same.

What it would cost:

- `build.py` would be replaced by Gradle, and the CI workflows, build tests and signing setup would need updating. I'd do that work and keep the current signing-key and release flow intact.
- The APK would grow from a few tens of KB to roughly 1–3 MB after shrinking.

I understand that a plain-framework build with no Gradle may be a deliberate choice. If so, no problem: I'd keep the current setup and send small, focused UI PRs that use the existing `Ui.java` helpers instead. Some examples:

1. A placeholder in the folder preview when icons fail to load (one of my folders shows "40 apps" with an empty circle).
2. A toolbar with a back arrow and a settings icon instead of the "Settings"/"Folders" text toggle.
3. "Personal"/"Work" in the profile picker instead of "Personal · profile 0".
4. All / Selected / Unassigned filters in the app picker.

Which would you prefer: the Gradle + Material 3 migration, or small PRs within the current setup?
