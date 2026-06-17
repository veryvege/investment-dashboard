# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A personal financial portfolio management workspace for a personal Moomoo Malaysia account. It combines a browser-based productivity dashboard with investment tracking data.

## Running the app

Open `dashboard.html` directly in a browser — no build step, no server, no dependencies. The app uses the **File System Access API** to read and write files on disk. When you open it, you grant it access to a directory containing `TASKS.md` (tasks) and optionally the `memory/` directory (Claude Code memory viewer).

## Architecture

### `dashboard.html` (3000+ lines, single-file app)

A self-contained vanilla HTML/CSS/JS application. No framework, no bundler, no npm. All logic lives in a single `<script>` block at the bottom.

**Two main views (tab-toggled):**

- **Tasks** — reads/writes `TASKS.md` using the File System Access API. Parses markdown into sections (`## Section Name`) and tasks (`- [ ] title`). Supports board (kanban) and list layouts, drag-and-drop column/card reordering, subtasks, and notes. Auto-saves on change; polls for external edits.

- **Memory** — read-only viewer for Claude Code's `memory/` directory. Parses YAML frontmatter + markdown body from each `.md` file, groups files by subdirectory, and renders them as cards or flat tables. The `memory/MEMORY.md` index file is shown as the overview tab.

**Key data flow:**
1. User opens a file/directory via File System Access API picker
2. Content is parsed in-memory (no localStorage, no backend)
3. Mutations call `markChanged()` → `autoSave()` which writes back to disk
4. `checkForExternalChanges()` polls the file handle to detect out-of-band edits

**Important functions:**
- `parseTaskMarkdown()` / `toMarkdown()` — bidirectional TASKS.md serialization
- `parseMemoryMarkdown()` — parses Claude memory file frontmatter and body
- `renderMemory()` / `renderMemoryTabs()` / `renderMemoryContent()` — memory UI
- `createColumn()` / `createCard()` — board card/column DOM generation

### `investments/inputs/holdings.csv`

Source-of-truth for portfolio positions: ticker, name, exchange, asset type, quantity, cost basis, currency, purchase date, target allocation. Update this when positions change. Used as input for any portfolio analysis.

### `memory/`

Claude Code's persistent memory for this project. `MEMORY.md` is the index. `portfolio.md` contains the latest portfolio snapshot (holdings, NAV, allocations). See the memory system instructions for how these files are structured.

## Data

- **Base currency:** MYR; US positions stored in USD
- **Platform:** Moomoo Securities Malaysia (margin account)
- **Portfolio snapshot date:** see `memory/portfolio.md` — update after each Moomoo daily statement

## Key conventions

- TASKS.md sections use `## Section Name`; tasks use `- [ ] title` or `- [x] title`; notes are indented continuation lines
- Memory files use YAML frontmatter (`name`, `description`, `metadata.type`) followed by markdown body
- When editing `dashboard.html`, keep all CSS in the `<style>` block and all JS in the single `<script>` block at the bottom — do not split into separate files
