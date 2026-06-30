# water-reminder

A tiny macOS hydration reminder. A `launchd` agent pops a **"Drink water"** notification
(with the *Glass* sound) at the top of every hour — no app to keep open, no background process
to babysit. macOS runs it for you.

## What's in here

| File                            | Purpose                                                               |
| ------------------------------- | --------------------------------------------------------------------- |
| `com.user.water-reminder.plist` | The `launchd` agent. Runs `osascript` to show an hourly notification. |

The agent runs:

```sh
osascript -e 'display notification "Drink water" with title "Hydration" sound name "Glass"'
```

on a `StartCalendarInterval` of `Minute = 0` — i.e. once an hour, on the hour.

## Requirements

- macOS (uses `launchd` and `osascript`)
- Notifications allowed for the script runner. The first time it fires, macOS may ask you to
  allow notifications — approve it, or enable it later under
  **System Settings → Notifications → Script Editor**.

## Install

```sh
# 1. Copy the agent into your per-user LaunchAgents directory
cp com.user.water-reminder.plist ~/Library/LaunchAgents/

# 2. Load it into your GUI session
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.water-reminder.plist
```

> On older macOS the equivalent is `launchctl load ~/Library/LaunchAgents/com.user.water-reminder.plist`.

### Verify it loaded

```sh
launchctl print gui/$(id -u)/com.user.water-reminder
# or, for a quick check:
launchctl list | grep water-reminder
```

### Test it right now

The agent only fires on the hour (`RunAtLoad` is `false`), so trigger it manually to confirm
notifications work:

```sh
launchctl kickstart -k gui/$(id -u)/com.user.water-reminder
```

You should see the **Hydration · Drink water** notification appear.

## Change the schedule

Edit `~/Library/LaunchAgents/com.user.water-reminder.plist`, then reload (see below). A few common
`StartCalendarInterval` patterns:

```xml
<!-- Every hour on the half hour -->
<key>StartCalendarInterval</key>
<dict>
    <key>Minute</key><integer>30</integer>
</dict>
```

```xml
<!-- Only at specific times (repeat the dict for each time) -->
<key>StartCalendarInterval</key>
<array>
    <dict><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Hour</key><integer>13</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Hour</key><integer>17</integer><key>Minute</key><integer>0</integer></dict>
</array>
```

For a fixed interval instead of clock times, swap `StartCalendarInterval` for
`StartInterval` (seconds): `<key>StartInterval</key><integer>3600</integer>`.

### Reload after editing

```sh
launchctl bootout gui/$(id -u)/com.user.water-reminder
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.water-reminder.plist
```

## Uninstall

```sh
# 1. Unload the agent
launchctl bootout gui/$(id -u)/com.user.water-reminder

# 2. Remove the file
rm ~/Library/LaunchAgents/com.user.water-reminder.plist
```

> On older macOS: `launchctl unload ~/Library/LaunchAgents/com.user.water-reminder.plist` then `rm`.

To confirm it's gone:

```sh
launchctl list | grep water-reminder   # should print nothing
```

## Troubleshooting

- **No notification appears.** Check it's loaded (`launchctl list | grep water-reminder`) and that
  notifications are enabled for **Script Editor** in System Settings → Notifications.
- **`Bootstrap failed: 5: Input/output error`.** The agent is already loaded — `bootout` first,
  then `bootstrap` again.
- **Nothing happens on the hour, but `kickstart` works.** That's expected if your Mac was asleep
  at the scheduled minute; `launchd` runs the job at the next wake.
