# Patch Notes — OMEN (RHYT)

**April 2026 — Vollständiger Audit + Visual Rework**

---

## Alpha 1.3 — Fail-Screen, Live-PP, Sterne-Cache (April 2026)

**Keine Gameplay-Logik geändert** — Rendering, HUD und Dateizugriffe.

| Bereich | Änderung |
|--------|-----------|
| **Fail-Screen** | `render_osu_result_layout` wird **pro Phase** in einen Offscreen-Cache gerendert (nicht jedes Frame zwischen Panel- und Button-Phase). |
| **Gameplay HUD** | **Live-PP:** Sterne (`resolve_mania_stars_for_pp`) **pro Map/Mods** cachen; `get_judgment_counts` nur **einmal** pro PP-Berechnung; statische Labels **„score“ / „pp“** cachen. |
| **SR / meta.json** | **Modul-Cache** für gleiche `meta.json` (Pfad + mtime + Mod-Signatur) — weniger wiederholtes Lesen (u. a. Song-Select). |

**Release:** Menü-Version **Alpha 1.3** (`config.GAME_VERSION`). **`omen.exe` SHA-256:** Hex `a78258576015105eaeae23be4a864dd82f495cdf16ff93ed4b6c136d20274551` · Base64 `p4JYV2AVEF6uriO+SoZN2C9JXN8W/5PtS2wTbSAnRVE=`

---

## Alpha 1.2 — Performance & consistency (April 2026)

**Keine Gameplay-Logik geändert** — nur Rendering und Cache-Verhalten.

| Bereich | Änderung |
|--------|-----------|
| **Mania** | `hit_y` und Scroll-Faktor (ppm) **einmal pro Frame pro Lane** statt pro Note; weniger redundante `hit_y_for_lane`-Aufrufe und Skin-Manager-Reads. |
| **DEPTH** | Notenfarben pro Lane **`note_color_rgb_for_lane` einmal pro Frame** für alle vier Lanes cachen statt pro Note. |
| **Particles** | Optional **`pygame.gfxdraw.filled_circle`**, Fallback auf `pygame.draw.circle`. |
| **Blur** | **`clear_blur_cache()`** ruft auch **`invalidate_blur_iterations_cache()`** auf — keine veralteten Blur-Iterationen aus dem TTL-Cache nach komplettem Cache-Clear. |

---

## Update — April 2026 (Portable / Freunde-Build)

**Start:** Im Release-Ordner **`omen.exe`** (Kopie von `RHYT.exe`) — kompletter Ordner inkl. `_internal`, Skins, leerer `assets\maps`, neutrales Profil.

### Butze / Omen — Profil & Stats

- Profil-Sync lädt zuverlässiger PP, Level, Spielzeit, ACC: `merge_game_player_profile` und `merge_public_omen_player_profile` laufen bei eingeloggtem Zustand immer (nicht nur wenn `butze_stats_authoritative` fehlt), damit lokale Werte (z. B. gewichtetes PP aus `scores.json`) Serverwerte nicht mehr „überdecken“.
- Zusätzliche API-Felder für PP: u. a. `rankedPp`, `totalRankedPp`, `globalPp`, `performance_points`; `_extract_remote_profile_fields_from_omen_payload` / `extract_omen_profile_fields_for_display` in `butze_client.py` — gleiche Metrik-Auflösung wie beim Settings-Merge (camelCase, verschachtelte Objekte).
- Ranking: `_parse_omen_profile_for_display` kombiniert Supplementary-Parser + Butze-Extraktion.
- Lokale PP-Berechnung: `_should_recompute_pp_from_local_scores()` — bei aktivem Butze-Login werden `total_pp` / `avg_accuracy` nicht mehr unkontrolliert aus `scores.json` überschrieben.

### Menü & Song Select

- **Hauptmenü — Profilkarte:** Spielzeit, Level rechts; Hintergrund-Sync nach Login; `_reload_profile_stats()` inkl. `total_playtime_ms`.
- **Song Select:** Beim Betreten blockierender `sync_profile_from_butze_blocking()` vor dem Laden der Settings (aktuelle Butze-Daten).
- Profil-Topbar (Avatar, Name, Lv/pp/Zeit/Acc, Level-Balken) entfernt; Suchfeld nutzt den Platz; Klick „Profil → Ranking“ entfernt. Ungenutzte Hilfen (`_format_topbar_playtime_ms`, `_truncate_str_to_font_width`, `_render_user_panel`, Topbar-Fonts) bereinigt.

### Performance & Import

- **osu-Import:** Jede `.osu` wird nur noch einmal voll gelesen (`parse_osu_mania_notes` nutzt bereits geladene Zeilen). Songlänge für `total_beats` einmal aus der Audio-Datei, nicht pro Difficulty. Zip-Extraktion streamt mit `copyfileobj`; geschriebene Import-Maps nutzen kompaktes JSON.
- **Song Select / Map-Liste:** Kein vollständiges `load_map` mehr für jeden Eintrag, sobald `meta.json` `bpm` und `total_beats` enthält (neue Imports); ältere Sets ohne diese Felder fallen auf das bisherige Laden zurück. Hilfsfelder (`list_total_beats_hint`, `list_mapper`) und `_ensure_map_data` laden die Map bei Bedarf nach. Collection-Filter lädt Collections nur einmal statt pro Song.

---

## Kritisch · Crashes & Gameplay-Breaking

### DT-Mod Abstürze behoben

- `pygame.mixer.get_init()` gibt `None` zurück, wenn der Mixer nicht initialisiert ist — führte zu **TypeError** beim Unpacking. Guard eingebaut. (`dt_audio_cache.py`)
- Leeres PCM (0 Samples) erzeugte **ValueError** in `np.linspace`. Wird jetzt abgefangen.
- Korrupter Cache wurde wiederverwendet statt neu generiert — Validierung mit Fallback auf Neugenerierung.
- Division durch null wenn `factor == 0` — Parameter-Guard.
- BPM `None`/`0` in der DT-Statistikanzeige crashte bereits im Song-Select. Fix: sicherer BPM-Fallback. (`song_select.py`)

### DT-Fallback bei fehlgeschlagenem Audio

- Wenn `get_or_create_dt_audio_path` fehlschlägt, wird DT automatisch deaktiviert und alle Gameplay-Parameter (Scroll, Hit-Windows, Score-Multiplikatoren) für normales Tempo neu berechnet. Kein Audio-Gameplay-Desync mehr. (`gameplay_new.py`, `mods.py`)

### UnboundLocalError `gp_st` behoben

- `_cached_playfield_ox`, `_cached_playfield_oy`, `_cached_scroll_speed` werden jetzt in `on_enter()` initialisiert statt in einem bedingten Branch. (`gameplay_new.py`)

### Settings werden korrekt gespeichert

- `on_enter()` lädt Settings von der Platte — kein veralteter RAM-Stand.
- Gleichzeitige Writer: Read-Modify-Write statt blindem Überschreiben (Lost Update).
- Stille Fehler bei Permission/IO werden geloggt statt verschluckt. (`config.save_settings`)

---

## Hoch · Visuals & Rendering

### Note-Rendering komplett überarbeitet

- **Immer Kreis-Modus:** Note-Skin-Auswahl (Rechteck/Kreis/Pfeile) aus Settings entfernt. Alle Noten, Rezeptoren und Long Notes werden als programmatische Kreise gezeichnet.
- **Exakt gleiche Größe:** Fallende Noten, Rezeptoren und LN-Köpfe/Enden verwenden alle `r = int(138 × 0.38) = 52` → **104px Durchmesser**.
- **Skin-Image-Problem gelöst:** osu!mania Skin-Images hatten intern viel transparenten Raum, was zu Größen-Mismatch führte. Programmatische Kreise ersetzen jetzt alle Skin-Image-Pfade für Noten, Rezeptoren und LNs.
- **Lane-Breite fest auf 138px** — kein Slider mehr in den Settings, keine Varianz.
- **Border-Konsistenz:** Note-Border und Rezeptor-Border gleich (2px normal, 3px Hard Rock).

### Skin-Import weiterhin aktiv

- osu!mania `.osk` Skins können weiterhin importiert werden.
- Skin-Farben und Konfiguration werden gelesen; nur die Note/Rezeptor-Grafiken nutzen programmatische Kreise statt Skin-Images.

---

## Hoch · Gameplay-Korrektheit

### Vollständiger Bug-Audit (14 Bugs behoben)

| # | Bug | Fix |
|---|-----|-----|
| 1 | Visuelle Noten-Drift während Pause | `get_visual_time_ms()` gibt `get_time_ms()` zurück wenn Mixer pausiert (`audio_clock.py`) |
| 2 | `hash()` nicht-deterministisch in Map-Generator | `hashlib.md5` für reproduzierbare Ergebnisse (`map_generator.py`) |
| 3 | MISS inkrementiert falschen Counter | Nur `_count_miss` wird bei MISS erhöht, nicht `_count_dissonance` (`score_system.py`) |
| 4 | ACC-Formel Diskrepanz | Standard osu!mania-Formel: (320×MAX + 300×300 + …) / (320×total) (`score_system.py`) |
| 5 | DT-Fallback spielt normales Audio mit DT-Timing | DT wird bei Fehler deaktiviert, Parameter neu berechnet (`gameplay_new.py`) |
| 6 | Arrow-Preview falscher Lane-Index | `draw_tap_note_preview` akzeptiert jetzt `lane_idx` Parameter (`mania_lane.py`) |
| 7 | `has_miss()` zählt BREAK als Miss | Nur `_count_miss > 0` (`score_system.py`) |
| 8 | Replay-Delta-Rundung driftet | Absolute Timestamps statt kumulative Deltas (`replay.py`) |
| 9 | Replay-Dateinamen-Parsing bricht bei `_` im Hash | `rsplit("_", 2)` statt `split("_")` (`replay.py`) |
| 10 | LN-Mindestdauer zu hoch | Von 2 Beats auf 0.25 Beats reduziert (`map_generator.py`) |
| 12 | `sv_seg` Parsing kann crashen | Längenprüfung `if len(row) < 3: continue` (`loader.py`) |
| 13 | `combat.reset()` unvollständig | Alle Combat-State-Variablen werden zurückgesetzt (`combat.py`) |
| 14 | osu-Parser Mode: ValueError | `try/except` um `int(mode_str)`, Default Mania (`osu_parser.py`) |

### Notelock-Verhalten verbessert

- LN-Grace nahe dem Tail: automatischer Release-Pfad.
- Notenwahl nach |Δt| priorisiert. (`gameplay_new.py`)

### Live-Scoreboard-Interpolation korrigiert

- Ghost-Scores steigen nicht mehr linear mit der Zeit in Breaks.
- Fortschritt entlang der Chart-Zeitachse. (`live_score_bar.py`)

---

## Mittel · Online & Profil

### Level-Import von Butze

- **Level wird nur noch von butzebot.com importiert** — das Game pusht kein Level/TotalScore mehr an die API.
- `apply_remote_profile_to_settings`: Remote `total_score` ist autoritativ (kein `max(local, remote)` mehr).
- `push_profile_snapshot` und `schedule_profile_push` deaktiviert.
- Lokale Level-Berechnung nach Score-Gewinn entfernt — Butze ist die einzige Quelle.

### Profilbild-Sync

- PFP wird von butzebot.com geladen (async, disk/memory Cache, WebP/SVG-Konvertierung).
- Andere Spieler-PFPs in Global Leaderboards angezeigt.

### Login-Screen

- Register-Link klickbar mit Hover-Feedback.
- **Maus-Cursor wird nach Login zurückgesetzt** — kein I-Beam-Symbol mehr nach Passwort-Eingabe.

---

## Mittel · Mod-System

- **NF/HT/HR/DT:** PP-/Stern-Logik an Lazer angeglichen.
- `pp_multiplier()` und `mania_lazer_pp_multiplier()` vereinheitlicht.
- Mod-Panel skaliert, Kategorien mit Trennlinien.

---

## Performance-Audit (16 Bottlenecks behoben)

| # | Bottleneck | Fix |
|---|-----------|-----|
| P1 | `config.load_settings()` im Hot-Path | 1-Sekunden TTL-Cache (`mania_lane.py`) |
| P2 | `_check_missed_obstacles` O(N) | `_miss_check_start_idx` → O(1) (`gameplay_new.py`) |
| P3 | Font-Erstellung in Render-Loops | Modul-Level `_font_cache` (`renderer.py`) |
| P4 | Neue Surface pro transparenter LN | `_scratch_surfaces` Pool (`slider_skins.py`) |
| P5 | `particles.pop(0)` ist O(n) | `collections.deque` + `popleft()` (`particles.py`) |
| P7 | Uncached HR-Glow-Surfaces | Cache in `_hitzone_cache` (`depth.py`) |
| P9 | Unconditional `print()` Spam | `_DEBUG = False` Flag (`osu_parser.py`) |
| P10 | Arrow-Surface-Cache Thrashing | Limit von 256 auf 1024 erhöht (`mania_lane.py`) |
| P11 | `sorted()` pro Frame in Live-Score | `_merge_sort_dirty` Flag (`live_score_bar.py`) |
| P13 | MP-Scoreboard Sort pro Frame | Cached sorted list (`gameplay_new.py`) |
| P14 | Pause-Overlay Surface pro Frame | Cached `_pause_overlay_surf` (`gameplay_new.py`) |
| P15 | O(n²) Effect-Removal in Combat | List comprehensions → O(N) (`combat.py`) |
| P16 | `_trim_particles` fehlt bei Spawn | Zu `spawn_harmony_ring`/`spawn_wave_trail` hinzugefügt (`particles.py`) |

---

## Song Select · UI

### Star-Rating Range in Song-Liste

- In der Song-Karussell-Ansicht wird jetzt der **Min–Max Star-Rating** aller Difficulties angezeigt (z.B. **1.6–2.9★**) statt nur einer einzelnen Difficulty.
- Songs mit nur einer Difficulty zeigen weiterhin den einzelnen Wert.

---

## Beatmap Browser · Erweitert

### Neuer Filter-Tab "Most Played"

- Im Beatmap-Browser gibt es jetzt einen **"Most Played"**-Tab, der die beliebtesten osu!mania 4K Maps nach Spielanzahl sortiert anzeigt (via Nerinyan API `sort=plays_desc`).

### Neuer Filter-Tab "OMEN Top"

- **"OMEN Top"**-Tab zeigt die meistgespielten Maps auf OMEN an.
- Versucht zuerst die Daten von butzebot.com `/stats/most-played` abzurufen.
- Fallback: Lokale Scores werden ausgewertet und die meistgespielten Songs auf osu.direct gesucht.

### Genre-Filter

- Zweite Chip-Zeile mit **Genre-Filtern**: All, Anime, Video Game, Electronic, Rock, Pop, Hip Hop, Metal, Classical.
- Genre wird direkt als API-Parameter an Nerinyan (`g=<id>`) übergeben und als Tag-Suche an osu.direct.
- Filter-Chips haben jetzt dynamische Breite (passt sich an Textlänge an) statt fixer Breite.

### UI-Verbesserungen Beatmap Browser

- Status-Filter-Chips (Ranked, Loved, etc.) und Genre-Chips haben jetzt **dynamische Breite** — kein Overlap oder Abschneiden mehr.
- Search-Bar Position passt sich dynamisch an die Chip-Zeilen an.
- Sauberes, kompaktes Layout.

---

## Auto-Download bei Login

### Fehlende Maps mit Bestätigung herunterladen

- Nach dem Login werden fehlende Maps gezählt.
- **Popup-Prompt**: „X fehlende Maps erkannt. Jetzt herunterladen?" mit Ja/Nein Buttons.
- Download startet **nur nach Bestätigung** (Taste Y oder Klick auf „Ja").
- Nein (N) oder ESC überspringt den Download.
- Downloads laufen im Hintergrund; Maps erscheinen automatisch in der Song-Auswahl.
- **Fix:** scores.json Struktur (`difficulties` Verschachtelung) wird jetzt korrekt durchlaufen.

---

## Mapper-Anzeige Fix

### Map Creator korrekt angezeigt

- **`.echoes`-Dateien** enthalten jetzt das `mapper`-Feld (aus `Creator:` der `.osu`-Datei).
- **`meta.json`** enthält jetzt das `mapper`-Feld (bevorzugt aus der osu-API `creator`, Fallback `.osu` Creator).
- **Song Select** liest mapper aus meta.json als Fallback wenn die .echoes-Datei keinen hat.

---

## Level-Import Fix

### Erweiterte Butze-Profil-Keys

- Level-Import sucht jetzt in mehr Feldern: `level`, `user_level`, `playerLevel`, `omen_level`, `game_level`.
- Sucht auch in verschachtelten `stats` und `omen` Objekten.
- Debug-Logging zeigt importierte Werte an.

---

## April 2026 — Update 2

### UI
- **Alpha 1.0** Versionsanzeige unten links im Menü.
- **Multiplayer**: Button zeigt "Coming Soon" mit Lock-Overlay, Klick blockiert.
- **Song Select**: Sort-Modus `DATE` (nach Import-Datum, neueste zuerst).
- **Song Select**: Scroll kann jetzt ganz nach unten (Difficulty-Sub-Rows werden berücksichtigt).
- **Map Info**: Zeigt RANKED (grün), LOVED (rosa + "Score wird nicht gespeichert") oder UNRANKED (grau).
- **Preview**: Nur bei Klick auf Song, nicht mehr bei Hover.

### Beatmap Browser
- **Genre-Filter**: Post-Filtering nach Genre-ID für zuverlässigere Ergebnisse.
- **Mapper + Tag Suche**: Creator und Tags werden bei der Suche berücksichtigt.
- **Lazy Import**: Downloads werden erst beim Verlassen des Browsers (ESC) importiert — kein Lag.

### Online / Butze
- **Top Plays**: Werden jetzt von Butze abgefragt (Cloud), Fallback auf lokale Scores.
- **Replay Upload**: Replay-Datei wird nach Score-Submission an Butze hochgeladen.
- **Level Import**: Level wird nie heruntergestuft, nur hochgestuft. Mehr API-Schlüssel werden geprüft.

### Release
- **Neutrale Settings**: `sfx_volume: 0`, leere persönliche Daten.
- **Keine persönlichen Bilder/Daten** im Release-Ordner.

---

## Aufräumen

- Tote Settings entfernt: `auto_dim_background`, `beat_pulse_intensity`, `background_parallax`, `wave_glow`, `language`.
- UIRenderer / SurferRenderer / toter `Renderer.update()`-Pfad entfernt.
- DEBUG für Release-Builds auf False.
- TIFF-Warning-Dialog unterdrückt (`SetErrorMode` in `main.py`).
- Hit-Sound-Lautstärke wird jetzt korrekt bei ALT+Scroll aktualisiert.

---

## Beatmaps

Unter **`assets/maps/`** liegen **keine** Songs — nur eine README. Maps im Spiel importieren oder manuell in den Ordner kopieren.
