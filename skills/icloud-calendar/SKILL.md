---
name: icloud-calendar
description: iCloud calendar skill using configurable vdirsyncer and khal settings.
metadata:
  {
    "openclaw":
      {
        "emoji": "📅",
        "os": ["linux"],
        "requires": { "bins": ["vdirsyncer", "khal"] },
        "install":
          [
            {
              "id": "apt",
              "kind": "apt",
              "packages": ["vdirsyncer", "khal"],
              "bins": ["vdirsyncer", "khal"],
              "label": "Install vdirsyncer + khal via apt",
            },
          ],
      },
  }
---

# iCloud Calendar

This skill uses `SKILL_ICLOUD_CAL_*` environment variables for configuration.

- `SKILL_ICLOUD_CAL_VDIRSYNCER_BIN` defaults to `vdirsyncer`.
- `SKILL_ICLOUD_CAL_KHAL_BIN` defaults to `khal`.
- `SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH` is required.
- `SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH` is required.
- `SKILL_ICLOUD_CAL_PROFILE` is required.
- Scope: all calendars exposed through the configured iCloud-backed profile.

## Sync First

Always sync before querying and after mutations:

```bash
${SKILL_ICLOUD_CAL_VDIRSYNCER_BIN:-vdirsyncer} -c "$SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH" discover "$SKILL_ICLOUD_CAL_PROFILE"
${SKILL_ICLOUD_CAL_VDIRSYNCER_BIN:-vdirsyncer} -c "$SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH" metasync "$SKILL_ICLOUD_CAL_PROFILE"
${SKILL_ICLOUD_CAL_VDIRSYNCER_BIN:-vdirsyncer} -c "$SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH" sync "$SKILL_ICLOUD_CAL_PROFILE"
```

## View Events

```bash
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list                        # Today
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list today 7d               # Next 7 days
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list tomorrow               # Tomorrow
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list 2026-01-15 2026-01-20  # Date range
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list -a Work today          # Specific calendar
```

## Search

```bash
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" search "meeting"
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" search "dentist" --format "{start-date} {title}"
```

## Create Events

```bash
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" new 2026-01-15 10:00 11:00 "Meeting title"
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" new 2026-01-15 "All day event"
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" new tomorrow 14:00 15:30 "Call" -a Work
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" new 2026-01-15 10:00 11:00 "With notes" :: Description goes here
```

After creating, sync to push changes:

```bash
${SKILL_ICLOUD_CAL_VDIRSYNCER_BIN:-vdirsyncer} -c "$SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH" sync "$SKILL_ICLOUD_CAL_PROFILE"
```

## Edit Events (interactive)

`khal edit` is interactive and requires a TTY. Use tmux if automating.

```bash
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" edit "search term"
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" edit -a CalendarName "search term"
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" edit --show-past "old event"
```

Menu options:

- `s` edit summary
- `d` edit description
- `t` edit datetime range
- `l` edit location
- `D` delete event
- `n` skip (save changes, next match)
- `q` quit

After editing, sync:

```bash
${SKILL_ICLOUD_CAL_VDIRSYNCER_BIN:-vdirsyncer} -c "$SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH" sync "$SKILL_ICLOUD_CAL_PROFILE"
```

## Delete Events

Use `khal edit`, then press `D` to delete, then sync:

```bash
${SKILL_ICLOUD_CAL_VDIRSYNCER_BIN:-vdirsyncer} -c "$SKILL_ICLOUD_CAL_VDIRSYNCER_CONFIG_PATH" sync "$SKILL_ICLOUD_CAL_PROFILE"
```

## Output Formats

For scripting:

```bash
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list --format "{start-date} {start-time}-{end-time} {title}" today 7d
${SKILL_ICLOUD_CAL_KHAL_BIN:-khal} -c "$SKILL_ICLOUD_CAL_KHAL_CONFIG_PATH" list --format "{uid} | {title} | {calendar}" today
```

Placeholders: `{title}`, `{description}`, `{start}`, `{end}`, `{start-date}`, `{start-time}`, `{end-date}`, `{end-time}`, `{location}`, `{calendar}`, `{uid}`

## Caching

The khal cache path depends on the configured khal profile and local storage settings. Inspect your khal config if you need to clear or relocate the cache.

If data looks stale after syncing, clear the cache path associated with your configured profile and re-run sync.

## Runtime Config Notes

- This skill assumes the configured profile is already defined in both config files.
- The vdirsyncer and khal config contents are user-managed; this repo does not prescribe a private bootstrap layout.
