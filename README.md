# OMEN — Portable Build

**osu!mania-style Rhythm Game for Windows. Portable EXE with runtime, skins, and online scores. Beatmaps optional (not included).**

---

## What's New (April 2026)

Full changelog: **[PATCH_NOTES_MARCH_2026.md](PATCH_NOTES_MARCH_2026.md)**

### Alpha 1.3.3 — Latest portable build

- Fresh **omen.exe** from current source: **fixes and polish** since 1.3.2. See **PATCH_NOTES** and dev repo.

### Alpha 1.3.2 — Latest portable build

- Fresh **omen.exe** from current source: **fixes and polish** since 1.3.1. See **PATCH_NOTES** and dev repo.

### Alpha 1.3.1 — Faster downloads & download fixes

- **Faster download speed** and **download bug fixes** for in-game / portable downloads. See **PATCH_NOTES** for detail.

### Alpha 1.3.0 — Latest portable build (1.3 line)

- Fresh **omen.exe** from current source: **fixes, polish, and features** since 1.2.3. Details: **PATCH_NOTES** and dev repo history.

### Alpha 1.2.3 — Latest portable build

- Fresh **omen.exe** from current source: **fixes and polish** since 1.2.2. See **PATCH_NOTES** and your dev repo for specifics.

### Alpha 1.2.2 — Chat scroll & last-score card

- **Chat (thread view):** Mouse wheel scrolls messages when the pointer is over the chat panel except the **text input** and **send** area; uses wheel **position** when available; maps horizontal wheel to vertical where needed; **Page Up / Page Down** for message scroll.
- **Input:** `MOUSEWHEEL` uses **`event.x`** fallback when **`event.y`** is 0 (trackpads).
- **Last score card:** Taller bubble body and **refined layout** to fix overflow.

### Alpha 1.2.1 — Receptor borders & long-note (hold) judgement

- **Receptor borders:** Idle/pulse on lane receptors (circle + rectangle skins) and depth-mode hitzone circles use the same **white–blue blend** as the playfield (`flare_dual_colors_for_lane`), not fixed gray or simple scaling. Border colors use **integer tuple math only** (no extra surfaces or per-frame allocations).
- **Long notes (osu!mania-style):** Early release before the tail window is a **combo break**, not a miss (fairer ACC/PP). Holding past the tail without key-up **auto-resolves** with a lenient tail judgement. **Hold ticks** (`add_tick`, half-beat spacing) with `_key_held` tracking. Score: `add_hold_combo_break` / `_count_hold_combo_break` for osu!-like denominators.

### Alpha 1.2 — Performance & consistency (April 2026)

Rendering and cache behavior only; **no gameplay logic changes.**

- **Mania:** `hit_y` and scroll factor (ppm) are computed **once per frame per lane** instead of per note — fewer redundant `hit_y_for_lane` calls and skin-manager reads.
- **DEPTH:** Per-lane note colors (`note_color_rgb_for_lane`) are **cached once per frame** for all four lanes instead of per note.
- **Particles:** Optional `pygame.gfxdraw.filled_circle` with **fallback** to `pygame.draw.circle`.
- **Blur:** `clear_blur_cache()` also calls **`invalidate_blur_iterations_cache()`** so blur iteration counts are not served from a stale TTL cache after a full cache clear.

### Latest (friend-ready portable build)

- **Butze / profile:** Reliable sync of PP, level, playtime, and accuracy — merge runs whenever logged in so local `scores.json` weighted PP no longer overwrites server values. Extra API fields (`rankedPp`, `totalRankedPp`, `globalPp`, `performance_points`) and unified extraction in `butze_client.py`. Ranking combines supplementary parser + Butze data. With Butze login, local PP/avg ACC are not recomputed from `scores.json` when inappropriate.
- **Main menu:** Profile card shows playtime and level (right); background sync after login; `_reload_profile_stats` includes `total_playtime_ms`.
- **Song select:** Blocking `sync_profile_from_butze_blocking()` before settings load on enter. Profile top bar removed (avatar/name/lv/pp/time/acc bar); search field uses the space; “Profil → Ranking” click removed; unused helpers/fonts cleaned up.
- **Performance & import:** Each `.osu` parsed once (reuse loaded lines); song length for `total_beats` once per audio; zip extraction streams; compact JSON for imported maps. Map list avoids full `load_map` per row when `meta.json` has `bpm` + `total_beats` (legacy sets fall back); collection filter loads collections once.

### Earlier highlights (see patch notes for detail)

- **Visual Rework:** Notes, receptors, and long notes as consistent circles (104px diameter).
- **Beatmap Browser:** “Most Played”, “OMEN Top”, genre filters.
- **ACC / gameplay fixes:** osu!mania accuracy formula, DT/replay/combat fixes, performance work (fonts, particles, sorting).
- **Level from Butze:** Level/XP from butzebot.com; profile images and login UX improvements.

---

## Quick Start

> **WICHTIG / IMPORTANT:** Do NOT use GitHub's "Download ZIP" (Code → Download ZIP).
> That downloads the source code, not the game. Instead:
>
> **Option 1:** `git clone https://github.com/Gexanx4real/OMEN.git` then `git lfs pull`
>
> **Option 2:** Download from [Releases](https://github.com/Gexanx4real/OMEN/releases)

1. Download / clone this repo (see above)
2. Double-click **`omen.exe`**
3. Keep the entire folder together (don't move just the `.exe`)

**First time:** The game opens a login screen for butzebot.com (optional).

**DLL Error?** If you see "Failed to load Python DLL", make sure:
- You have the **full folder** with `_internal/` next to `omen.exe`
- Install [Microsoft Visual C++ Redistributable (x64)](https://aka.ms/vs/17/release/vc_redist.x64.exe)

## Beatmaps

**`assets/maps/`** is intentionally empty. Import maps in-game (Beatmap Browser) or copy osu!mania map folders manually.

## Skins

Included skins:
- **ECHOES Default** — built-in circle style
- **IceTea Orbs** — imported osu!mania skin

Note: All note/receptor rendering uses programmatic circles regardless of skin. Skin colors and configuration are still respected.

## Controls

| Key | Action |
|-----|--------|
| A, S, D, Space | Lane 1-4 |
| ESC | Pause / Back |
| F2 | Random map |
| ALT + Scroll | Volume |

## Tech

- **Runtime:** Python 3 + Pygame (bundled in `_internal/`)
- **Git LFS:** `llvmlite.dll` (~102 MB) — run `git lfs pull` after cloning if needed
- **Requirements:** Windows 10+, Microsoft Visual C++ Redistributable (x64) if DLL errors occur
- See **`LIESMICH_RELEASE.txt`** for German instructions
