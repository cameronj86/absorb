# Absorb (queue fork) — notes for the next session

Fork of absorb `main`, rebased onto v1.12.1+303 (2026-11-11). Adds an AntennaPod-style queue, an Inbox, and a list-based Now Playing tab with its own player. Built by GitHub Actions (`.github/workflows/personal-apk.yml`, push to `queue`); nothing here is ever compiled in a cloud session, so every patch needs a real CI run before anything is claimed to work.

## Carrying the source between sessions
The cloud workspace does not persist. Source of truth is the fork's `queue` branch; failing that, upstream zip plus the patches applied in order (`absorb-queue.patch`, `-v2`, `-v3`). Do not reconstruct from memory.

## Rebasing onto upstream
Rebase the fork commits onto the new upstream, don't merge. Last done at 1.12.1+303; what that rebase had to reconcile:
- Upstream added `playNextInAbsorbing(key, {item})` and a next/last picker (`absorb_placement.dart`). Upstream's version is kept, with a strict-queue branch; the fork's own "Play next" pill in the episode sheet was dropped as redundant.
- Upstream added `rememberStartedInAbsorbing`, which enqueues a book started from its sheet. Under strict queue it now only warms the cache and leaves enqueueing to adoption.
- Upstream took the pref key `nowPlayingLayout` (notification text layout). The fork's player layout is `forkPlayerLayout`.
- Check `_updateAbsorbingCache` for any new raw `_absorbingIdsAdd` call after each rebase: everything that auto-includes must go through its local `queueAdd`.

## Where things live
New logic is in new files wherever possible; upstream files only get small hooks, each commented "fork addition" or "Fork addition".

- `lib/widgets/ap_episode_tile.dart` — the one episode row shape (feed, Inbox, queue), cover fallback, time formatters.
- `lib/widgets/queue_list_view.dart` — Now Playing tab as a drag-to-reorder list.
- `lib/widgets/now_playing_sheet.dart` — full player (layouts: dial, tape, cover) and `MiniPlayerBar`.
- `lib/providers/_lp_absorbing.dart` — strict queue, adoption, dismissals, `reorderAbsorbingVisible`, `playNextInAbsorbing`.
- `lib/services/player_settings.dart` — all fork prefs: `strictQueue`, `strictQueueAdoptSeconds`, `inboxWindowDays`, `feedSwipeRight/Left`, `queueListLayout`, `forkPlayerLayout`.
- Hooks: `absorbing_screen.dart` (returns `QueueListView` early in `build`), `app_shell.dart` (mini bar above the nav bar), `podcast_episode_feed.dart` (Inbox filter, swipes, list row), `settings_screen.dart` ("Queue & Inbox" section), `audio_player_service.dart` (chapterless skip), `login_screen.dart` + `backup_sync_screen.dart` (trust-all-certs applied on restore; Advanced opens on a TLS error).

## Decisions worth not re-litigating
- The card carousel is kept, behind Settings → Queue & Inbox → List layout (off). Casting also falls back to it; the new player has no cast transport.
- Strict queue: nothing enters the queue uninvited; a finished item leaves it and the item that slid into its slot plays next.
- Skip-next on an item with no chapters finishes it and advances (AntennaPod's behaviour). Skip-previous restarts it.
- Inbox is a filter plus a timestamped dismissed map, not a screen. Dismissals older than the Inbox window are pruned.
- No colours of the fork's own: everything reads the app theme.
- New strings are hardcoded English (no ARB keys, no `gen-l10n`).

## Still waiting on a real-world answer
- Whether ABS sends `size` on feed episodes (the row shows it when present, hides it otherwise).
- Whether a queued episode whose file the server deleted at 21 days fails cleanly or stalls the queue.
- Whether the player sheet closing two seconds after the player empties is the right debounce between queue items.
