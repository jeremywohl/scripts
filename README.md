Scripts
--

| Script | Description |
| --- | --- |
| apple-sign-expiry | Mac: show Apple provisioning-profile expirations for each app id |
| brightmatch | Mac: watch for brightness changes on any monitor and mirror them to the rest |
| fable | How many Fable tokens are left this week? |
| ghts | Touchstore-based gh (GitHub's cmdline) |

### apple-sign-expiry

Lists the provisioning profiles Xcode has left on this Mac, with days
until each expires. A device install stops launching when its 1-year profile expires, so this is
roughly the deadline for re-installing each app. Exits 1 if anything is expired or within
45 days (`WARN_DAYS`).

    apple-sign-expiry
    apple-sign-expiry --notify CMD [ARG...]

`--notify` is for a daily launch agent. It runs `CMD ARG...` with the message appended as
the final argument, one line per app, whenever an app crosses a threshold: 45, 30, 21, 14,
10, 7, 5, 3, 2, 1, 0 days out, then weekly until 60 days past expiry. There is no default
command. For example:

    apple-sign-expiry --notify terminal-notifier -title "Apple signing expiry" -message

Files in `~/.config/apple-sign-expiry/`:

- `state` — thresholds already notified, so a missed day fires on the next run and reruns
  don't repeat. Not written if the command fails.
- `ignores` — app ids to never flag or notify, one per line, with or without the team
  prefix. Shell globs and `#` comments are fine. Ignored apps still show in the table.

`apple-sign-expiry.plist` is a sample launch agent: set `SCRIPTS`, copy it to
`~/Library/LaunchAgents/local.apple-sign-expiry.plist` and `launchctl bootstrap gui/$(id -u)`
that path. It runs once at load, then daily at 09:05.

### brightmatch

Watches every monitor whose brightness macOS can control (built-in, Apple displays, LG
UltraFine) and, when one changes, sets the others to match.


    brightmatch                  run the watcher
    brightmatch -v               log each change and sync
    brightmatch --list           show displays and current brightness, then exit
    brightmatch --interval SECS  poll interval (default 0.5)
    brightmatch --epsilon FRAC   change threshold on a 0-1 scale (default 0.01)

### fable

Shows Claude Code usage as colored bars, per account: current session, week (all models)
and week (Fable), each with its reset time. It runs `claude -p /usage` for each account
and reformats the result. Exits 1 if any account's usage couldn't be read.

    fable              all accounts
    fable work         only the named accounts

Accounts are listed in `~/.config/fable/accounts`, one per line: a label, then that
account's `CLAUDE_CONFIG_DIR`. A label alone (or a path of `-` or `~/.claude`) is the
default account, which runs with `CLAUDE_CONFIG_DIR` unset. A commented boilerplate file
is written on first run.

    personal
    work      ~/.claude-work

### ghts

Runs `gh` with `GH_TOKEN` taken from touchstore (entry `my-github-pat`), so the token
isn't kept in the filesystem. Arguments pass straight through:

    ghts pr list
