---
name: caldav-calendar
description: Personal agent CalDAV skill using dedicated vdirsyncer/khal configs.
metadata:
  {
    "clawdbot":
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

# CalDAV Calendar (Personal)

Use this skill only in the personal agent workspace.

- `vdirsyncer` config: `/home/node/.config/vdirsyncer/personal.conf`
- `khal` config: `/home/node/.config/khal/personal.conf`
- Scope: all calendars available in the personal iCloud account.

## Sync First

Always sync before querying and after mutations:

```bash
vdirsyncer -c /home/node/.config/vdirsyncer/personal.conf discover personal
vdirsyncer -c /home/node/.config/vdirsyncer/personal.conf metasync personal
vdirsyncer -c /home/node/.config/vdirsyncer/personal.conf sync personal
```

## View Events

```bash
khal -c /home/node/.config/khal/personal.conf list                        # Today
khal -c /home/node/.config/khal/personal.conf list today 7d               # Next 7 days
khal -c /home/node/.config/khal/personal.conf list tomorrow               # Tomorrow
khal -c /home/node/.config/khal/personal.conf list 2026-01-15 2026-01-20  # Date range
khal -c /home/node/.config/khal/personal.conf list -a Work today          # Specific calendar
```

## Search

```bash
khal -c /home/node/.config/khal/personal.conf search "meeting"
khal -c /home/node/.config/khal/personal.conf search "dentist" --format "{start-date} {title}"
```

## Create Events

```bash
khal -c /home/node/.config/khal/personal.conf new 2026-01-15 10:00 11:00 "Meeting title"
khal -c /home/node/.config/khal/personal.conf new 2026-01-15 "All day event"
khal -c /home/node/.config/khal/personal.conf new tomorrow 14:00 15:30 "Call" -a Work
khal -c /home/node/.config/khal/personal.conf new 2026-01-15 10:00 11:00 "With notes" :: Description goes here
```

After creating, sync to push changes:

```bash
vdirsyncer -c /home/node/.config/vdirsyncer/personal.conf sync personal
```

## Edit Events (interactive)

`khal edit` is interactive and requires a TTY. Use tmux if automating.

```bash
khal -c /home/node/.config/khal/personal.conf edit "search term"
khal -c /home/node/.config/khal/personal.conf edit -a CalendarName "search term"
khal -c /home/node/.config/khal/personal.conf edit --show-past "old event"
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
vdirsyncer -c /home/node/.config/vdirsyncer/personal.conf sync personal
```

## Delete Events

Use `khal edit`, then press `D` to delete, then sync:

```bash
vdirsyncer -c /home/node/.config/vdirsyncer/personal.conf sync personal
```

## Output Formats

For scripting:

```bash
khal -c /home/node/.config/khal/personal.conf list --format "{start-date} {start-time}-{end-time} {title}" today 7d
khal -c /home/node/.config/khal/personal.conf list --format "{uid} | {title} | {calendar}" today
```

Placeholders: `{title}`, `{description}`, `{start}`, `{end}`, `{start-date}`, `{start-time}`, `{end-date}`, `{end-time}`, `{location}`, `{calendar}`, `{uid}`

## Caching

Personal khal cache path:

```bash
/home/node/.local/share/khal/personal/khal.db
```

If data looks stale after syncing:

```bash
rm /home/node/.local/share/khal/personal/khal.db
```

## Runtime Config Notes

Config files are bootstrap-managed and already present in:

- `/home/node/.config/vdirsyncer/personal.conf`
- `/home/node/.config/khal/personal.conf`

The vdirsyncer config uses:

- `collections = ["from b"]` (sync all remote calendars)
- `username.fetch` and `password.fetch` from environment
- local store at `/home/node/.local/share/vdirsyncer/personal`
