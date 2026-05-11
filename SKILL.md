---
name: workspace-organizer
description: "Date-structured workspace ($WORKSPACE_ROOT/YYYY/MM/DD) — auto-archive agent outputs, received files, and web-collected data by date. Structured actions + naming conventions."
version: 1.0.0
author: xlykyz
tags: [hermes, skill, workspace, file-management]
---

# Workspace Organizer

A daily file-flow organizer for Hermes Agent. All files that pass through your agent — documents received via messaging platforms, analysis reports, web-collected data, logs, screenshots, deliverables — are archived into `$WORKSPACE_ROOT/YYYY/MM/DD/`.

## Root Directory Configuration

Default: `$HOME/workspace/`
Override by setting the `WORKSPACE_ROOT` environment variable (e.g., `/mnt/data/workspace`).

```bash
# Safe default: fall back to /tmp if HOME is unset
WORKSPACE_ROOT="${WORKSPACE_ROOT:-${HOME:-/tmp}/workspace}"
```

## Directory Structure

```
$WORKSPACE_ROOT/
└── YYYY/
    └── MM/
        └── DD/
            ├── market_review_20260506.md
            ├── feishu_Alice_meeting_notes.pdf
            ├── deliverable_daily_brief_v1.docx
            └── tushare_sector_flow.csv
```

## Structured Actions

All scripts include `set -euo pipefail` to prevent silent pipeline failures, undefined variables, and uncaught exit errors.

### ensure_workspace_dir

Ensure today's working directory exists and return its full path. Run this before writing any file to the workspace.

```bash
#!/bin/bash
set -euo pipefail
BASE="${WORKSPACE_ROOT:-${HOME:-/tmp}/workspace}"
YEAR=$(date +%Y)
MONTH=$(date +%m)
DAY=$(date +%d)
WORK_DIR="$BASE/$YEAR/$MONTH/$DAY"
mkdir -p "$WORK_DIR"
echo "$WORK_DIR"
```

> `mkdir -p` is a no-op when the directory already exists — zero overhead.

### get_workspace_dir

Return today's path without creating the directory.

```bash
#!/bin/bash
set -euo pipefail
BASE="${WORKSPACE_ROOT:-${HOME:-/tmp}/workspace}"
YEAR=$(date +%Y)
MONTH=$(date +%m)
DAY=$(date +%d)
echo "$BASE/$YEAR/$MONTH/$DAY"
```

### archive_file

Move a file into today's working directory. Appends a timestamp suffix to avoid overwriting duplicates.

```bash
#!/bin/bash
set -euo pipefail
SRC="$1"
BASE="${WORKSPACE_ROOT:-${HOME:-/tmp}/workspace}"
YEAR=$(date +%Y)
MONTH=$(date +%m)
DAY=$(date +%d)
DEST_DIR="$BASE/$YEAR/$MONTH/$DAY"
mkdir -p "$DEST_DIR"
FILENAME=$(basename "$SRC")
DEST="$DEST_DIR/$FILENAME"

# Append timestamp suffix if destination already exists
if [ -f "$DEST" ]; then
  EXT="${FILENAME##*.}"
  STEM="${FILENAME%.*}"
  TS=$(date +%H%M%S)
  DEST="$DEST_DIR/${STEM}_${TS}.${EXT}"
fi

if [ -f "$SRC" ]; then
  mv "$SRC" "$DEST"
  echo "Archived: $DEST"
else
  echo "Source file not found: $SRC"
  exit 1
fi
```

> Use this after downloading files via `wget`/`curl` in the terminal, or for any file on a temporary path.

### archive_file_to_date

Move a file into a specific date's directory (for backfilling historical files).

```bash
#!/bin/bash
set -euo pipefail
SRC="$1"
DATE_PATH="$2"  # Format: YYYY/MM/DD
BASE="${WORKSPACE_ROOT:-${HOME:-/tmp}/workspace}"
DEST_DIR="$BASE/$DATE_PATH"
mkdir -p "$DEST_DIR"
FILENAME=$(basename "$SRC")
DEST="$DEST_DIR/$FILENAME"

# Append timestamp suffix if destination already exists
if [ -f "$DEST" ]; then
  EXT="${FILENAME##*.}"
  STEM="${FILENAME%.*}"
  TS=$(date +%H%M%S)
  DEST="$DEST_DIR/${STEM}_${TS}.${EXT}"
fi

if [ -f "$SRC" ]; then
  mv "$SRC" "$DEST"
  echo "Archived to $DATE_PATH: $DEST"
else
  echo "Source file not found: $SRC"
  exit 1
fi
```

> Usage example: `archive_file_to_date /tmp/old_report.pdf 2026/05/06`

## File Naming Convention

| Type | Format | Example |
|------|--------|---------|
| Analysis / Review | `topic_description.md` | `market_review_20260506.md` |
| Platform-attached files | `{platform}_sender_description.ext` | `feishu_Alice_meeting_notes.pdf` |
| Web-collected data | `source_topic.ext` | `tushare_sector_flow.csv` |
| Agent deliverables | `deliverable_description.ext` | `deliverable_daily_brief_v1.docx` |
| Transaction logs | `trade_log_YYYYMMDD.md` | `trade_log_20260506.md` |
| Screenshots / Images | `topic_description.png` | `sector_heatmap.png` |
| Skill files | `SKILL_name_version.md` | `SKILL_workspace-organizer_v1.md` |

> **Customizing the platform prefix**: Replace `{platform}` in the naming convention with your actual source (e.g., `slack_`, `teams_`, `discord_`, `dingtalk_`, `wechat_`). This is a convention, not enforced by code.

## Workflow Rules

### Writing Files
1. Run `ensure_workspace_dir` to get today's path
2. Write the file following the naming convention
3. For backdating files, use `archive_file_to_date`

### Receiving Files (Platform Attachments)
1. Immediately run `archive_file` or manually copy to today's workspace
2. Name as `{platform}_sender_description.ext`

### Sending Deliverables to User
1. Save to workspace first
2. Send from the agent interface:
   - **Single file**: send directly
   - **Multiple files**: `zip` and send as archive
   - **Unless the user explicitly requests individual files**, honor their preference

### Terminal Downloads
1. After download, run `archive_file <path>` to move into today's directory
2. Or specify the target path directly during download

## Search Methods

- **By date**: `ls "$WORKSPACE_ROOT/2026/05/"`
- **Full-text search**: `grep -r "keyword" "$WORKSPACE_ROOT/"`
- **By filename**: `find "$WORKSPACE_ROOT" -name "*keyword*"`
- **By type**: `find "$WORKSPACE_ROOT" -name "*.csv"`
- **Recent files**: `find "$WORKSPACE_ROOT" -type f -printf '%T@ %p\n' | sort -rn | head`

## Design Principles

1. **No cron, zero overhead**. Directories are created only when files exist — nothing happens otherwise.
2. **Flat by date**, not by project. Multi-day projects reference relevant dates in file content.
3. **No subdirectories within a day** — every file at the same level. Simple enough to stick with.
4. **Append-only**. No automatic cleanup. Manual pruning or a future retention policy is expected.
5. **Safety first**: all scripts use `set -euo pipefail`, and duplicate files get automatic timestamp suffixes to prevent overwrites.
