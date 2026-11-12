Thanks!

**Multiple work profiles:** Noted. The data layer already keys folders by profile serial, so storage and matching scale fine. The UI is the gap: every managed profile is labelled just "Work", so two work profiles show up as two identical "WORK" sections, and the profile picker only tells them apart by serial ("Work · profile 28"). I'll make sure everything I touch handles any number of work profiles. That includes labelling each profile by its own name (e.g. "Island") where Android exposes it, and falling back to "Work 2", "Work 3" and so on where it doesn't.

**Empty circle:** I noticed it in the repo's own screenshot, `docs/screenshots/drawer-folders.png` (the "Google" folder with 40 apps). I haven't seen it on my device. I'll start with a PR that shows a placeholder glyph whenever a preview has no icons, and that logs icon-load failures instead of silently ignoring them. Those logs should help pin down the cause if it happens again.
