# Hermes Skill: Workspace Organizer

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A [Hermes Agent](https://github.com/NousResearch/hermes-agent) skill that automatically organizes files by date into a structured `$WORKSPACE_ROOT/YYYY/MM/DD/` hierarchy. Perfect for keeping agent outputs, received attachments, web-collected data, and working files tidy without any cron jobs or background daemons.

## Features

- **Zero-overhead daily archiving** — directories created on demand, no background processes
- **Four shell utility scripts**: `ensure_workspace_dir`, `get_workspace_dir`, `archive_file`, `archive_file_to_date`
- **Safe by default** — `set -euo pipefail` on all scripts, automatic timestamp dedup for duplicate filenames
- **Configurable root path** via `WORKSPACE_ROOT` environment variable
- **Platform-agnostic naming conventions** — adapt to Slack, Teams, Discord, Feishu, or any messaging platform
- **Append-only design** — never deletes your files

## Installation

### Prerequisites

- Hermes Agent (any version)
- Bash 4.0+

### Quick Install

```bash
# Clone the repository
git clone https://github.com/xlykyz/hermes-skill-workspace-organizer.git
cd hermes-skill-workspace-organizer

# Copy to your Hermes skills directory (adjust path as needed)
cp -r . /path/to/hermes/skills/workspace-organizer

# Or symlink:
ln -s "$PWD" /path/to/hermes/skills/workspace-organizer
```

### Configure Hermes Agent

Add the skill to your Hermes Agent configuration:

```yaml
# In your hermes agent config or user config
skills:
  - workspace-organizer
```

The agent will load instructions from `SKILL.md` automatically.

## Usage

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `WORKSPACE_ROOT` | `$HOME/workspace` | Root directory for the date-structured archive |

### Utility Scripts

```bash
# Get today's directory path (create if missing)
ensure_workspace_dir
# Output: /home/user/workspace/2026/05/11

# Get today's directory path (read-only)
get_workspace_dir
# Output: /home/user/workspace/2026/05/11

# Archive a file into today's directory
archive_file /tmp/downloaded_report.pdf
# Output: Archived: /home/user/workspace/2026/05/11/downloaded_report.pdf

# Archive a file to a specific date
archive_file_to_date /tmp/old_report.pdf 2026/05/06
# Output: Archived to 2026/05/06: /home/user/workspace/2026/05/06/old_report.pdf
```

### Directory Structure

After a few days of usage:

```
$WORKSPACE_ROOT/
├── 2026/
│   ├── 05/
│   │   ├── 09/
│   │   │   ├── market_review_20260509.md
│   │   │   ├── slack_Bob_project_update.pdf
│   │   │   └── deliverable_sprint_report_v2.docx
│   │   ├── 10/
│   │   │   ├── trade_log_20260510.md
│   │   │   └── sector_heatmap.png
│   │   └── 11/
│   │       └── research_notes.md
```

## Customization

### Change the Workspace Root

```bash
export WORKSPACE_ROOT=/mnt/bigdrive/archive
```

Add to your `~/.bashrc` or Hermes Agent environment config to persist.

### Change the Platform Prefix in Naming Convention

The `{platform}_` prefix in file naming (e.g., `feishu_`, `slack_`, `teams_`) is a **convention, not enforced by code**. Simply name your files however you like. To customize:

- **For agent-generated files**: modify the naming instructions in your Hermes Agent config or in `SKILL.md`
- **For manually archived files**: just use whatever naming format you prefer

### Add a Retention Policy (Optional)

The skill is append-only by design. To add cleanup:

```bash
# Example: delete files older than 90 days
find "$WORKSPACE_ROOT" -type f -mtime +90 -delete

# Or set up a cron job (if you want one)
0 3 * * 0 find "$WORKSPACE_ROOT" -type f -mtime +90 -delete
```

## File Naming Convention Reference

| Type | Format | Example |
|------|--------|---------|
| Analysis / Review | `topic_description.md` | `market_review_20260506.md` |
| Platform attachments | `{platform}_sender_description.ext` | `slack_Alice_notes.pdf` |
| Web-collected data | `source_topic.ext` | `tushare_sector_flow.csv` |
| Agent deliverables | `deliverable_description.ext` | `deliverable_daily_brief_v1.docx` |
| Transaction logs | `trade_log_YYYYMMDD.md` | `trade_log_20260506.md` |
| Screenshots / Images | `topic_description.png` | `sector_heatmap.png` |

## Design Principles

1. **No cron, zero overhead** — directories exist only when files do
2. **Flat by date, not by project** — multi-day projects reference dates in content
3. **No subdirectories per day** — simple enough to stick with
4. **Append-only** — no automatic deletion; add a retention policy if needed
5. **Safety first** — `set -euo pipefail` everywhere, timestamp dedup on filename collisions

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

MIT — see [LICENSE](LICENSE).
