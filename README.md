# gt-restic

A simple CLI to manage media backups (movies, audiobooks, games, etc.) using [restic](https://restic.net/) on any [rclone](https://rclone.org/)-supported cloud storage.

## Requirements

- [restic](https://restic.net/) — install via your package manager:
  ```bash
  # Debian/Ubuntu
  sudo apt install restic

  # Arch
  sudo pacman -S restic

  # macOS
  brew install restic
  ```

- [rclone](https://rclone.org/install/) — install via your package manager or:
  ```bash
  sudo apt install rclone
  # or
  brew install rclone
  ```

## Set up rclone

Before using gt-restic, configure a remote with rclone. This works with any rclone-supported cloud (Jottacloud, S3, Backblaze B2, Google Drive, Dropbox, etc.):

```bash
rclone config
```

Follow the interactive prompts to add your cloud provider. When done, verify it works:

```bash
rclone listremotes
# e.g. output: mycloudremote:
```

Note the remote name — you'll need it in the config file.

## Install

```bash
git clone https://github.com/gthoo/gt-restic.git
cd gt-restic

cp gt-restic ~/.local/bin/gt-restic
chmod +x ~/.local/bin/gt-restic

mkdir -p ~/.config/gt-restic
cp config.example ~/.config/gt-restic/config
cp password.example ~/.config/gt-restic/password
chmod 600 ~/.config/gt-restic/password
```

Edit the config with your rclone remote name and desired backup path:
```bash
nano ~/.config/gt-restic/config
# e.g. RESTIC_REPOSITORY="rclone:mycloudremote:media-backups"
```

Edit the password file with a strong password:
```bash
nano ~/.config/gt-restic/password
```

> **Warning:** This password encrypts your backup. If you lose it, your backup is unrecoverable. Store it in a password manager.

## Initialize the repository

Run this once before first use:
```bash
gt-restic --init
```

## Usage

### Backup a file
```bash
gt-restic --send "Attack.of.the.Space.Potatoes.mkv" --type movie
gt-restic --send "The.Invisible.Sandwich.m4b" --type audiobook
gt-restic --send "super-blob-runner-3000.zip" --type game
gt-restic --send "DJ.Cactus.Greatest.Hits.flac" --type music
gt-restic --send "The.Adventures.of.Captain.Socks.S01" --type show
```

Multiple types are supported:
```bash
gt-restic --send "Atlantis.iso" --type game --type win95win98
```

### Search the backup
```bash
gt-restic --search potato
gt-restic --search sandwich
```

### Restore a file
```bash
# Restores to ./restored by default
gt-restic --restore potato

# Restore to a specific directory
gt-restic --restore potato --target ~/Movies
```

### List snapshots
```bash
gt-restic --list
gt-restic --list --type movie
gt-restic --list --type movie --type show
```

### Remove snapshots

Auto-remove older duplicates of the same path, keeping only the latest:
```bash
gt-restic --forget
```

Remove specific snapshots by ID:
```bash
gt-restic --forget 26151022 84ce716e
```

> **Note:** Forgetting a snapshot only removes the record. The actual data is not freed until you run `restic prune`.

## Supported types

| Type        | Example                              |
|-------------|--------------------------------------|
| `movie`     | Attack.of.the.Space.Potatoes.mkv     |
| `show`      | The.Adventures.of.Captain.Socks.S01  |
| `audiobook` | The.Invisible.Sandwich.m4b           |
| `music`     | DJ.Cactus.Greatest.Hits.flac         |
| `game`      | super-blob-runner-3000.zip           |
| `other`     | anything else                        |

## Notes

- **Do not mount restic to watch movies directly.** Every read requires fetching from the remote — use `--restore` to get the file locally first, then watch it.
- Config files live in `~/.config/gt-restic/` and are never committed to this repo.
