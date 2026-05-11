# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

Infrastructure-as-config for encrypted desktop backups: [restic](https://restic.net/) → Backblaze B2 (S3-compatible API), orchestrated with [Task](https://taskfile.dev/), automated via a systemd user timer.

There is no application code — the project consists of `Taskfile.yml`, systemd unit files, and `restic-excludes.txt`.

## Prerequisites

```bash
sudo apt install restic
sudo snap install task --classic
```

## Setup

```bash
cp .env.example .env   # fill in B2 credentials, repo URL, and restic password
task init              # initialize the restic repository on B2 (run once)
```

## Key commands

| Command | What it does |
|---|---|
| `task backup` | Full backup + prune old snapshots |
| `task backup:run` | Backup only (no prune) |
| `task prune` | Apply retention policy and delete old snapshots |
| `task check` | Verify repository integrity |
| `task snapshots` | List snapshots |
| `task restore [SNAPSHOT_ID=abc123] [TARGET=/tmp/r]` | Restore a snapshot |
| `task systemd:install` | Copy unit files to `~/.config/systemd/user/` |
| `task systemd:enable` | Enable and start the daily timer (02:00) |
| `task systemd:status` | Show timer state + last 50 journal lines |

## Architecture

- **`Taskfile.yml`** — single source of truth. Loads `.env` via `dotenv`, exposes `BACKUP_SOURCES` (space-separated paths) and `EXCLUDES_FILE` as vars. All restic calls live here.
- **`restic-excludes.txt`** — patterns passed to `restic backup --exclude-file`. Edit to add/remove exclusions.
- **`systemd/restic-backup.service`** — oneshot service; hardcodes `EnvironmentFile=%h/dev/backup/.env` and `ExecStart` pointing to `%h/dev/backup`. **Update these paths** if the repo is cloned elsewhere before running `task systemd:install`.
- **`systemd/restic-backup.timer`** — fires daily at 02:00, with `Persistent=true` (catches missed runs on next boot) and up to 10 min random delay.

## Retention policy

Defined in `Taskfile.yml` under the `prune` task: 7 daily, 4 weekly, 6 monthly, 1 yearly. All snapshots are tagged `workstation`.

## Credentials and configuration

`.env` is git-ignored. Required variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (B2 application key), `RESTIC_REPOSITORY` (`s3:https://s3.REGION.backblazeb2.com/BUCKET`), `RESTIC_PASSWORD` (restic encryption passphrase), and `BACKUP_SOURCES` (space-separated absolute paths to back up, e.g. `/home/user/Documents /home/user/Images`).
