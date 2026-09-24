# 👻 GhostTrace by ROXX

> **A single-file, read-only forensic scanner for Windows — no installation, no writes, no strings attached.**

[![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011%20%7C%20Server-informational?style=flat-square)](https://www.microsoft.com/windows)
[![Language](https://img.shields.io/badge/language-PowerShell%205.1%2B-blue?style=flat-square)](https://learn.microsoft.com/powershell/)
[![Status](https://img.shields.io/badge/status-stable-brightgreen?style=flat-square)]()
[![Version](https://img.shields.io/badge/version-v14.55-orange?style=flat-square)]()
[![Read-Only](https://img.shields.io/badge/read--only-yes-success?style=flat-square)]()

---

## 📖 Table of Contents

- [Why I Built GhostTrace](#-why-i-built-ghosttrace)
- [The Core Philosophy](#-the-core-philosophy)
- [Quick Start](#-quick-start)
- [What is GhostTrace?](#-what-is-ghosttrace)
- [Feature Overview — The Complete Arsenal](#-feature-overview--the-complete-arsenal)
  - [Search & Filter Parameters](#1-search--filter-parameters)
  - [Core Scan Modes](#2-core-scan-modes)
  - [The Exclusion Engine](#3-the-exclusion-engine)
  - [Results Table & UI Features](#4-results-table--ui-features)
  - [Export Pipeline](#5-export-pipeline)
- [Real-World Scenarios](#-real-world-scenarios)
- [Architecture & Design Decisions](#-architecture--design-decisions)
- [Edge Cases and Robustness](#-edge-cases-and-robustness)
- [What GhostTrace Deliberately Does NOT Do](#-what-ghosttrace-deliberately-does-not-do)
- [Version History](#-version-history)
- [FAQ](#-faq)
- [License](#-license)

---

## 💡 Why I Built GhostTrace

I work with machines that run **massive creative suites** — Adobe Creative Cloud, Avid Media Composer, FL Studio, and similar heavy software bundles. Over the years, I noticed a pattern that almost every IT professional eventually notices:

> **Software uninstallers lie.**

When you uninstall Adobe Premiere Pro, the uninstaller removes the executable. What it *doesn't* remove — what it silently leaves behind — is often measured in **gigabytes**:

- Cached render files under `AppData\Roaming\Adobe\Common\Media Cache Files`
- Orphaned peak files, `.pek`, `.cfa` scratch directories
- Plugin remnants scattered across three or four different installation roots
- Registry keys in `HKLM\SOFTWARE\Adobe` and `HKCU\Software\Adobe` that survive every "official" uninstall
- User preference files, sync databases, crash logs, update installers

And the "solution" the modern internet offers is even worse: **blind junk cleaners.** Tools like CCleaner, BleachBit, and various "PC optimizers" that promise to sweep it all away — except they regularly delete something important, break the OS in subtle ways, or wipe a user's legitimate project files because the heuristic was too aggressive.

I wanted something different. I wanted a tool that:

1. **Shows me exactly what's there**, not what a heuristic *thinks* I should clean.
2. **Never touches anything** unless *I* explicitly decide to.
3. **Runs from a USB stick**, without installing anything, without leaving traces on the target machine.
4. **Is auditable** — one file, readable line-by-line, no compiled binary to trust.
5. **Doesn't need** .NET SDK, Node.js, Electron, Python, or any dependency chain that turns a 200 KB tool into a 200 MB installation.

That's GhostTrace.

---

## 🎯 The Core Philosophy

Every design decision in GhostTrace flows from four principles. When in doubt about a feature, these four win.

### 1. 🔌 Single-File, Zero-Footprint Portability

GhostTrace is **one `.ps1` file**. Copy it to a USB stick, a network share, an admin workstation. Double-click it. It runs. There is:

- **No installation.** No MSI, no setup wizard, no registry writes on the host.
- **No external dependencies.** No `.dll` files to ship alongside. No runtime to install. Windows 10/11 and Windows Server have everything needed out of the box (PowerShell 5.1 ships with Windows).
- **No permanent artifacts.** GhostTrace does not write to `Program Files`, does not register itself, does not create shortcuts. Close it, and the only trace it leaves is whatever files *you* export.

**Why this matters: incident response.** In a forensic or IR context, *installing software alters the machine*. Every install writes to the registry, touches drive sectors, changes timestamps. GhostTrace can be run entirely from RAM — from a mounted USB stick — without modifying the target system in any way. This is why it's a `.ps1` and not a compiled `.exe`.

### 2. 🔒 Read-Only by Design

GhostTrace **does not move, write, delete, or modify a single file or registry key on the target machine.** Every operation is a read:

- `DirectoryInfo.EnumerateFileSystemInfos()` — read
- `Get-ChildItem` on registry hives — read
- `FileInfo.Length` — read

There is no "Clean" button. There is no "Auto-Fix." There is no "Recommended Actions" wizard.

> **The philosophy: GhostTrace finds. You decide.** A forensic tool that deletes things isn't a forensic tool — it's a cleaner.

### 3. 🔍 Transparency Over Convenience

Every scan mode explains *what it does* and *why*. Every result row shows the *reason* the file matched. Every summary line is explicit. There are no hidden heuristics, no "smart" auto-categorization, no opaque scoring systems.

If GhostTrace reports 15 GB of Adobe residue, every single byte of that number is traceable back to a specific file on disk, visible in the results grid, exportable to Excel.

### 4. 🎛️ Non-Destructive by Default, Powerful on Request

GhostTrace offers *depth of investigation*, not *depth of automation*. You choose:

- FAST vs. DEEP
- Whether to include the registry
- Whether to filter by date
- Whether to use Smart Exclusions
- Whether to add Custom Exclusions

Nothing runs automatically. Nothing "just happens." The user is always in control.

---

## 🚀 Quick Start

**Prerequisites:** Windows 10, Windows 11, or Windows Server 2016+. PowerShell 5.1 (default on all modern Windows). No additional software required.

### Running GhostTrace

**Method 1 — Right-click run**

1. Save `GhostTrace.ps1` anywhere (Desktop, USB stick, Downloads folder).
2. Right-click → **Run with PowerShell**.
3. UAC prompt appears → click **Yes**. GhostTrace auto-elevates.

**Method 2 — PowerShell console**

```powershell
powershell -ExecutionPolicy Bypass -File "C:\Path\To\GhostTrace.ps1"
```

**Method 3 — From a USB stick (forensic use)**

1. Copy `GhostTrace.ps1` to a USB drive.
2. Plug into the target machine.
3. Right-click → **Run with PowerShell**.
4. GhostTrace runs entirely in RAM. The USB is only read; nothing is installed, nothing is written to `Program Files`, nothing touches the OS installation.

### First Scan

1. Type a keyword (e.g. `Adobe`) in the first keyword box.
2. Select a drive from the dropdown (`C:\`, `D:\`, or `ALL DRIVES`).
3. Leave **FAST Scan** selected.
4. Click **START SCAN**.

The results appear in real time. When done, the summary is written to the Activity Log and a completion popup shows the totals.

To find out *how much space* those results actually occupy, click **CALC SIZES** after the scan — GhostTrace will stat only the matched files (not the whole drive) and show the total.

---

## 🔎 What is GhostTrace?

**GhostTrace is a standalone, lightweight, read-only forensic scanner that helps users hunt down software residue, hidden remnant files, and storage hogs.**

Unlike blind "junk cleaners" that automatically delete files (and potentially break the system), GhostTrace prioritizes **deep investigation** — allowing users to locate traces, review their size and location, and decide manually what is safe to clean.

### 🩹 The Problem it Solves

When massive software suites (Adobe, Avid, DaVinci Resolve, FL Studio, and similar heavy packages) are uninstalled, they often leave behind:

- **Gigabytes of hidden cache files** in `AppData\Local`, `AppData\Roaming`, and `ProgramData`
- **Orphaned registry keys** under `HKLM\SOFTWARE`, `HKCU\Software`, and the Uninstall hives
- **Scattered user data** — preferences, project templates, plugin caches, log directories
- **Update stubs and installer remnants** that consume disk space indefinitely

Standard uninstallers miss most of this. Blind "junk cleaners" are the alternative — and they routinely break the OS, remove legitimate application files, or delete user data because the heuristic was too aggressive.

GhostTrace exists to close that gap. It lets users **precisely track software residue**, **examine its actual storage footprint**, and **perform a controlled cleanup at their own discretion.**

### 🛡️ System Security Posture

- **GhostTrace is a read-only engine.** It does not move, write, delete, or modify any files on the target drive. Every operation is a filesystem read.
- **The tool automatically elevates privileges upon launch** (requesting Administrator rights via Windows UAC) to bypass "Access Denied" errors when securely scanning system directories and registry hives. Without elevation, half the interesting locations (system caches, other users' AppData, `HKLM`) would be inaccessible.
- **No network access.** GhostTrace never phones home, never downloads anything, never checks for updates online. It runs entirely offline.

---

## 🧰 Feature Overview — The Complete Arsenal

### 1. Search & Filter Parameters

The engine operates on highly customizable filters to narrow down the exact footprint of unwanted software.

#### 🔤 Multi-Keyword Lookup

Users can enter **up to 5 concurrent search terms** (e.g. `Adobe`, `Premiere`, `Creative Cloud`). The engine uses **OR logic** — if any keyword matches, the file is flagged.

The result row shows *which* keywords matched, e.g. `Adobe, Premiere` for a file that matched both.

#### 🎯 Exact-Word Matching Boundaries

GhostTrace does **not** perform a naive substring search. It uses **Unicode-aware Regular Expressions** to enforce word boundaries:

| Search term | ✅ Matches | ❌ Does NOT match |
|-------------|-----------|------------------|
| `Avid` | `Avid_Studio`, `Avid.dll`, `Avid (folder)` | `David`, `AvidLiving` (with no boundary) |
| `1080` | `1080p`, `HD1080` | `bd2025fc` (digits bounded by letters) |
| `2025` | `2025 (folder)`, `Project-2025` | `20250000`, `x2025y` |

**How it works:** for each keyword, GhostTrace builds a dynamic regex with lookbehind/lookahead assertions:

- If the keyword starts with a letter → `(?<!\p{L})` (not preceded by a letter)
- If the keyword starts with a digit → `(?<!\p{N})` (not preceded by a digit)
- If the keyword ends with a letter → `(?!\p{L})` (not followed by a letter)
- If the keyword ends with a digit → `(?!\p{N})` (not followed by a digit)

> **Why it matters:** this is the difference between a search that returns 400 useful results and a search that returns 40,000 false positives. The boundary logic is the same philosophy used in forensic grep and code search tools.

#### 💾 Drive & Scope Selection

Target options:

- **A specific fixed drive** (`C:\`, `D:\`, `E:\`)
- **ALL DRIVES** — scan every fixed drive simultaneously
- **A specific folder** — click *"or specific folder"* to open a browse dialog and target an arbitrary subdirectory

#### 🧹 Input Sanitization

GhostTrace validates keyword inputs before every scan. If you accidentally paste `C:\Users\Adobe` as a keyword, the invalid characters (`\`, `:`) are stripped and a warning is shown — the scan proceeds with the remaining valid keywords. This prevents the search engine from being confused by path-like inputs.

---

### 2. Core Scan Modes

GhostTrace offers multiple scanning methodologies depending on the required depth of the investigation.

#### ⚡ FAST Scan (Paths + Last Modified Dates)

**How:** Scans file paths and metadata *without* requesting physical file sizes.

**Why:** Disk I/O — asking the hard drive to read a file's physical byte size — is the slowest part of any scan. On a spinning disk, each `FileInfo.Length` call may trigger a seek. On an SSD, it's fast but still not free. Skipping this makes FAST scan **nearly instantaneous** (typically 30–60 seconds for an 800 GB drive on modern hardware).

**Scenario:** You need to quickly see where every file named `Adobe` and/or `Photoshop` resides on your `C:\` drive. Use FAST scan to get the layout in under a minute.

**Performance numbers (reference):**

| Storage | Time |
|---------|------|
| ~800 GB SSD | 25–30 seconds |
| ~800 GB HDD | 40 sec – 1 min |

*(These represent metadata enumeration, not content reads — GhostTrace doesn't read file contents, only paths and timestamps.)*

#### 🔬 DEEP Scan (+ File Sizes)

**How:** Forces the OS to calculate the exact `FileInfo.Length` (physical byte size) of every matched file.

**Why:** To determine exactly how much storage space is being wasted by leftover files.

**Scenario:** You need to see not just *where* Adobe files are, but *how much space they take up* — because you're deciding whether to delete them.

**Trade-off:** DEEP scan is slower than FAST scan because it performs an additional stat call per matched file. On SSDs the difference is small; on HDDs with millions of tiny files, it can be significant.

> **💡 Alternative — CALC SIZES button:** If you ran a FAST scan and later decide you want to see the file sizes, you don't need to rescan. Click **CALC SIZES** after the scan finishes and GhostTrace will stat *only the already-matched results* — no second disk traversal.

#### 🗂️ Registry Scan

**How:** Safely performs a read-only scan of Windows Registry hives to locate uninstallation traces and ghost configurations. It checks four locations:

```
HKLM:\Software
HKLM:\SOFTWARE\WOW6432Node          (32-bit apps on 64-bit Windows)
HKCU:\Software
HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall
```

**Why:** Many applications fail to reinstall or misbehave because ghost configuration keys and parameters remain in the Registry. When you install Adobe again after a failed uninstall, the installer sometimes sees the leftover keys and refuses to proceed, or silently inherits broken settings.

**Depth:** The registry scan is deliberately **shallow** — it only inspects top-level children of each hive. It does *not* recurse into every subkey. This is intentional:

- A full recursive registry scan across `HKLM\Software` can take 10–30 minutes and produce thousands of near-meaningless hits.
- The top-level children (e.g. `HKLM\Software\Adobe`) are almost always where the interesting stuff is.
- Deep registry residue cleanup is dangerous without expert judgment.

> If you need deeper registry investigation, dedicated tools (RegScanner, Autoruns) are the right choice. GhostTrace's registry scan is a **quick sweep**, not a complete audit.

#### 📅 Narrow Down by Date

**How:** GhostTrace first searches for your specified keyword(s), then filters the matches to only show files and folders whose **Last Modified Date** falls within the selected date range. Works with both FAST and DEEP scan modes.

**Why:** Helps locate files related to a known application, project, or keyword by focusing only on the period when they were installed, created, or modified.

**Scenario:** You know you installed Adobe Photoshop sometime last month, but your computer already contains thousands of Adobe-related files accumulated over several years. By searching for `Adobe` and enabling *Narrow Down by Date* for last month, GhostTrace will only return Adobe files that were created or modified during that timeframe — making it much easier to identify the files associated with that specific installation or update.

#### 🕐 Timeframe Space Analyzer (TSA)

**How:** Ignores Keywords entirely, automatically forces DEEP Scan, and finds **every single file on the drive** modified *or* created within a specific date range that is larger than a specified MB limit (user-defined).

**Why:** For times when you don't know *the name of the software* causing bloat, but you only know *when* the bloat happened.

**Scenario:** Your `C:\` drive suddenly gains 5 GB over a weekend, but Windows doesn't tell you why. Simply run Timeframe Space Analyzer in GhostTrace for the affected date range (Saturday–Sunday) and set a preferred minimum size threshold (e.g. 20 MB). GhostTrace will rapidly surface all large files greater than 20 MB that were created or modified during that timeframe — making it easy to identify updates, downloads, logs, backups, installers, or other storage-heavy changes responsible for the increase.

**The result rows show a "Reason" instead of "Keyword":**

| Reason | Color |
|--------|-------|
| `Created in Timeframe` | 🟢 Green |
| `Modified in Timeframe` | 🟠 Orange |
| `Created & Modified in Timeframe` | 🔵 Blue |

**🧮 The Min Size Filter (and why it exists):**

Timeframe Space Analyzer is designed to investigate significant storage changes within a specific date range. On a typical system, **millions of files** may be created, modified, or touched by routine OS activity — many of them only a few kilobytes in size.

The Minimum Size Filter helps eliminate this noise by only displaying files above a user-defined threshold (e.g. 50 MB). This allows the analysis to focus on the files most likely responsible for substantial disk space increases — large downloads, software updates, backups, cache files, installers, virtual machine images, media files, log growth.

Without this filter, the results could be overwhelmed by thousands of insignificant file changes, making it much harder to identify the files that actually contributed to a large increase in storage usage.

**☁️ Remote/Placeholder File Handling (v14.54/v14.55):**

Windows supports cloud-backed files (OneDrive Files On-Demand, for example) that appear in the filesystem but aren't physically present on disk. These files have attribute bits set (`RECALL_ON_DATA_ACCESS` = `0x00400000`, `RECALL_ON_OPEN` = `0x00040000`) indicating the OS may need to fetch them from a remote source.

TSA detects these files and:

- **Still lists them** as timeframe matches (they exist, they matched the date range).
- **Flags their size as `Remote/Placeholder (Size N/A)`** in the results grid — because `FileInfo.Length` on such files reports their *logical* size, not their physical local disk usage. Windows does not expose a reliable physical-size value for these files.
- **Excludes them from size totals** so the reported "Total Data Found" isn't inflated by phantom bytes that aren't actually on disk.

---

### 3. The Exclusion Engine

To prevent the engine from freezing on bottomless system folders, GhostTrace utilizes a robust exclusion system.

#### 🛡️ Built-in Smart Exclusions *(especially for C: drive scans)*

**How:** Automatically skips predefined high-volume Windows system directories and other resource-intensive folders that are commonly unrelated to software trace investigations. Individual exclusions can be reviewed, enabled, or disabled through the configuration panel (click the `[N out of M directories excluded]` link).

**Default exclusions:**

| Directory | Reason |
|-----------|--------|
| `\Windows\WinSxS` | Component store — often 100k+ files |
| `\Windows\System32` | Core OS binaries |
| `\Windows\SysWOW64` | 32-bit OS binaries |
| `\ProgramData\Microsoft` | OS-managed program data |
| `\node_modules` | Node.js dependency trees (dev machines) |
| `\Cache` | Generic cache folders |
| `\Temp` | Temp directories |
| `$RECYCLE.BIN` | Recycle bin |
| `System Volume Information` | Shadow copy metadata |

**Why:** Significantly reduces scan time by avoiding locations that often contain hundreds of thousands or even millions of files, while minimizing irrelevant results.

**Scenario:** You are performing a scan across your entire `C:\` drive for traces of an application you recently uninstalled. Instead of spending time scanning massive Windows OS directories, GhostTrace skips them automatically and focuses on locations where software traces are most commonly found.

> ⚠️ **Smart Exclusions are skipped folders, not deleted folders.** Nothing is ever touched — it's simply not enumerated.

#### 📂 Custom Exclusions

**How:** Allows users to specify additional folders or paths that should be ignored during scanning. Click **+ Add Folder** to open a multi-select tree picker. The picker:

- Shows all fixed drives with lazy-loaded folder trees (folders are loaded only when you expand them, so the tree opens instantly).
- Lets you check as many folders as you want across different drives.
- Adds all checked folders to the Custom Exclusions text box as a comma-separated list.

**Why useful:** This is particularly useful for excluding:

- Large data repositories you know are safe
- Backup locations
- Media libraries (movie, music, photo folders)
- Development project directories
- Any folder where you *know* no meaningful trace would be found

**Under the hood:** Custom exclusions are matched against the **full path** (not just the folder name). If you exclude `C:\Users\Me\Videos`, then only `C:\Users\Me\Videos` and everything under it are skipped — a different folder named `Videos` elsewhere is not affected.

#### 🚨 Exclusion Transparency (v14.54)

If the **scan root itself** is excluded (e.g. you scan `C:\Windows` while Smart Exclusions are on), GhostTrace now explicitly warns you:

- **Activity Log:** `[!] SCAN TARGET EXCLUDED: 'C:\Windows' matches an active exclusion rule and was skipped entirely.`
- **Summary "Coverage Warning" line:** a distinct line appears in the summary explaining that the target was never actually examined.

This is a crucial correctness property — **"scanned and found nothing" must never be confused with "never scanned at all."**

#### 🔒 Inaccessible Root Transparency (v14.54)

Similarly, if the scan root cannot be enumerated (permission denied, drive disconnected, etc.), GhostTrace reports:

- **Activity Log message** on the exception
- **Summary "Coverage Warning" line**

Both cases (excluded root, inaccessible root) surface in the summary so you can never be misled by an empty-looking result set.

---

### 4. Results Table & UI Features

The main interface is a split container: the top half is the results grid, the bottom half is the log area.

#### 📊 The Results Grid

| Column | Description |
|--------|-------------|
| **Keyword** | Which search term(s) matched this file (e.g. `Adobe, Photoshop`) |
| **Type** | `File`, `Folder`, or `Registry` |
| **SizeMB** | File size in MB (only populated in DEEP mode or after CALC SIZES) |
| **Date** | Last modified date in `yyyy-MM-dd HH:mm:ss` format |
| **Path** | The full filesystem path (or registry key) |

**Built for performance:**

- **50,000-row UI cap** — the DataGridView freezes at 50,000 visible rows to protect RAM and GDI handles. The background scanner *continues* to store every result in memory (`$global:masterRecord`). This means you can have a scan with 500,000 matches and the UI stays responsive, with all 500,000 available for export.
- **Time-boxed drain (v14.54)** — the UI processes results for up to ~120 ms per 200 ms timer tick. Older versions used a fixed 150-row cap, which meant the UI could fall behind on large scans. Time-boxing keeps the UI catching up smoothly.
- **Batched UI updates** — results are added to the DataTable inside `BeginLoadData()` / `EndLoadData()` blocks, which suppresses per-row repaint and recalc.

#### 🔍 Live Filter

A real-time filter box at the top of the results grid. Type anything — a partial path, a keyword, a date fragment — and the grid instantly filters to matching rows without rescanning.

#### 📄 Files-Only Mode

A checkbox that hides all `Folder` type rows, leaving only `File` results. Useful when you only care about actual storage hogs.

#### 🖱️ Right-Click Context Menu

On any result row:

| Option | Behavior |
|--------|----------|
| **Open File Location** | Opens Windows Explorer with the file selected (`explorer /select,`) |
| **Open Root Match Folder** | Walks from scan root down and opens the first folder that itself matches a keyword |
| **Copy Full Path** | Puts the path on the clipboard |
| **Calculate File Sizes (Current Results)** | Same as CALC SIZES button (FAST scans only) |

**🎯 The Root Match Folder logic (v14.53+):**

Given:

- Scan root: `D:\`
- Result: `D:\Movies\Marvel\Bluray\Spiderman-1080.mp4`
- Keywords: `Marvel`, `Bluray`, `1080`

GhostTrace walks components top-down:

```
Movies       → no match
Marvel       → ✅ match      ← STOP HERE
Bluray       → (never reached)
```

**Root Match Folder = `D:\Movies\Marvel`**

If the keyword only matches in the *filename* (e.g. `Spiderman-1080.mp4` matched on `1080` but no folder did), the "Open Root Match Folder" option is **hidden** — because there is no folder match.

#### 📋 Activity Log & Error Log Tabs

Two tabs at the bottom of the splitter:

- **Activity Log** — live scan progress: startup messages, "Searching: C:\", speed reports, summary, warnings.
- **Error Log** — per-file access errors, metadata errors, and fatal scanner errors. Capped at 1,000 retained error records (with a total count beyond that) to prevent OOM on corrupted drives.

**Detach button:** either tab can be popped out into a separate floating window via the **Detach Log** button. The floating window preserves the tab's color scheme (black background, colored text) and can be closed to reattach.

#### 📈 Progress Bar & Stats Panel

- **Elapsed time** (HH:MM:SS)
- **Scanned count** with live files-per-second rate
- **Marquee progress bar** during scan; switches to full when complete

#### ⏯️ Pause / Resume / Stop

- **PAUSE** — halts the scanner at the next file boundary. The stopwatch pauses too, so elapsed time is accurate. Resume continues from the same point.
- **STOP** — aborts the scan cooperatively (not a hard kill). If there are still results waiting in the queue, a dialog asks whether to wait for them to be drained or to discard them.

---

### 5. Export Pipeline

Click **EXPORT XLSX** after a scan to save the full unfiltered dataset (`$global:masterRecord`, not the truncated grid).

#### 📗 Excel Export (Primary Path)

GhostTrace uses **direct memory injection** into Excel via COM automation. It does *not* write cell-by-cell (which is slow beyond ~5,000 rows). Instead:

1. Builds a 2D `[System.Object[,]]` array in memory containing all rows (headers + data).
2. Assigns the entire array to a single `Range.Value2` in one operation.
3. Applies freeze-panes on row 1, autofilter, and column auto-fit.
4. Appends the summary block (with scan metadata) and the error log block.
5. Places a footer at the bottom.

**Filename suggestion** is auto-generated:

| Scan type | Filename pattern |
|-----------|------------------|
| Keyword scan | `YYYYMMDD-HHMM - <keyword1> & <keyword2>.xlsx` |
| TSA scan | `YYYYMMDD-HHMM - Timeframe Analysis of Target <drive>.xlsx` |

Dots in keywords are replaced with underscores so `.mp4` doesn't create a file called `...mp4.xlsx` (which Windows would misinterpret).

#### 📄 CSV Fallback

If Excel isn't installed on the machine (or COM automation crashes), GhostTrace detects the failure and offers to export as CSV instead. The CSV export:

- Writes the results table with `Export-Csv -Encoding UTF8 -UseCulture`
- Appends the summary block
- Appends the error log block
- Writes the footer

**UTF-8 encoding (v14.54):** Preserves Unicode filenames and paths. In older versions, non-ASCII characters (Telugu, Hindi, Japanese, accented Latin) silently turned into `?` because Windows PowerShell 5.1 defaulted to ANSI.

**CSV field escaping (v14.55):** Every field in the manually-written error log block is quoted and its internal quotes are doubled via the `ConvertTo-SafeCsvField` helper.

#### 📝 Summary Contents

Every export contains a summary block with:

- Scan Status (`COMPLETED` / `FAILED` / `ABORTED BY USER`)
- Scan Target (drive or folder)
- **Coverage Warning** (if the root was excluded or inaccessible)
- Scan Type (FAST / DEEP / REGISTRY / combination / TSA)
- Scan Duration
- Custom Exclusions (as configured)
- Smart Exclusions status
- Total Data Size (or N/A in FAST mode)
- Total Items Scanned
- Total Matches Found
- Errors Encountered
- **Per-keyword match counts** — e.g. `[Adobe] = 1,234 matches.`, `[Photoshop] = 456 matches.`
- **Overlap note** — if multiple keywords matched the same files
- **TSA breakdown** — Created / Modified / Both sizes for TSA scans
- **Remote/Placeholder note** — if any cloud-backed files were found
- **Top 5 Largest Files**

---

## 🎬 Real-World Scenarios

### Scenario 1 — *"I Uninstalled Adobe But It Still Won't Reinstall"*

**Goal:** Find the orphaned configuration that's blocking a reinstall.

1. Launch GhostTrace.
2. Enter keyword: `Adobe`
3. Enable **Registry Keys** and **DEEP Scan**.
4. Target: `C:\`
5. Run.

The results will show both filesystem residue and registry keys. Export to Excel, look at the `Registry` type rows, and examine which `HKLM\Software\Adobe` subkeys remain. You can now decide what to remove manually.

### Scenario 2 — *"My C: Drive Gained 20 GB Overnight and I Don't Know Why"*

**Goal:** Find the culprit without knowing the software name.

1. Launch GhostTrace.
2. Enable **Timeframe Space Analyzer**.
3. Set the date range to *yesterday → today*.
4. Set Min Size to `50 MB`.
5. Target: `C:\`
6. Run.

The results show every file >50 MB that was created or modified in that window. Sort by SizeMB descending. The top results will almost always be the culprit.

### Scenario 3 — *"I Want to Move All My Video Projects Off This Drive Before Formatting"*

**Goal:** Inventory everything video-related.

1. Launch GhostTrace.
2. Enter keywords: `mp4`, `mkv`, `mov`.
3. Enable **DEEP Scan**.
4. Target: the drive.
5. Run.
6. Click **CALC SIZES** to get exact totals.
7. Click **EXPORT XLSX** to save the full list with paths and sizes.

### Scenario 4 — *"Is This USB Drive Safe to Plug Into My Machine?"*

**Goal:** Inventory the drive without executing anything on it.

1. Copy `GhostTrace.ps1` to your local machine.
2. Plug the USB in.
3. Launch GhostTrace, target the USB drive letter.
4. Use TSA mode with a very wide date range (e.g. `2000-01-01` → today) and Min Size = `1 MB`.

This gives you every file >1 MB on the USB without needing a keyword.

---

## 🏗️ Architecture & Design Decisions

GhostTrace is a PowerShell script, but that doesn't mean it's simple. Here's how it actually works.

### 🔄 The Producer / Consumer Model

GhostTrace separates UI rendering from filesystem crawling using a classic producer/consumer pattern:

```
┌────────────────────────────────────────────────────────────────┐
│                      BACKGROUND RUNSPACE                        │
│                                                                 │
│  [powershell]::Create().AddScript({...})                        │
│                                                                 │
│  Scan-Engine                                                    │
│    ├── DirectoryInfo.EnumerateFileSystemInfos()                 │
│    ├── regex matching                                           │
│    ├── metadata extraction                                      │
│    └── Enqueue([PSCustomObject]@{...})                          │
│                                                                 │
└──────────────────┬──────────────────────────────────────────────┘
                   │
       ┌───────────┼───────────┐
       │           │           │
       ▼           ▼           ▼
  ┌─────────┐ ┌─────────┐ ┌────────────┐
  │ Queue   │ │LogQueue │ │ErrorQueue  │
  │(results)│ │ (logs)  │ │ (errors)   │
  └────┬────┘ └────┬────┘ └─────┬──────┘
       │           │            │
       └───────────┼────────────┘
                   │
                   ▼
       ┌─────────────────────────┐
       │    UI THREAD (200ms)    │
       │                         │
       │  Timer.Tick             │
       │   ├── drain queues      │
       │   ├── update grid       │
       │   ├── update logs       │
       │   └── update stats      │
       └─────────────────────────┘
```

**Why this matters:**

- **The UI never blocks.** The scanner is off-thread; the UI just drains queues.
- **The queues are lock-free.** `ConcurrentQueue` handles all cross-thread synchronization.
- **No control access from the worker.** The worker only touches queues and shared state.

### 🧵 The Shared Synchronized Hashtable

```powershell
$syncHash = [hashtable]::Synchronized(@{
    Queue            = New-Object System.Collections.Concurrent.ConcurrentQueue[object]
    LogQueue         = New-Object System.Collections.Concurrent.ConcurrentQueue[string]
    ErrorQueue       = New-Object System.Collections.Concurrent.ConcurrentQueue[object]
    Running          = $false
    Paused           = $false
    BackgroundDone   = $false
    ScannedItems     = 0
    ScanStatus       = "Idle"
    ErrorCounter     = $global:errorCounter
    RootExcluded     = $false
    RootInaccessible = $false
})
```

Every piece of state that both the UI thread and the worker need to see lives here. The hashtable is *synchronized* — PowerShell's wrapper for thread-safe property reads and writes.

### 🔐 The ErrorCounter Class

An earlier version of GhostTrace used a plain hashtable for error counting. It worked, but it wasn't atomic across threads. The current version uses a small C# class:

```csharp
public class ErrorCounter {
    private readonly object _lock = new object();
    public int Total = 0;
    public int AccessDenied = 0;
    // ... other fields ...

    public void IncrementByType(string errorType) {
        lock (_lock) {
            // ... increment the appropriate field ...
            Total = Total + 1;
        }
    }

    public void Reset() {
        lock (_lock) {
            // ... zero all fields ...
        }
    }
}
```

**Why a lock-based class instead of `Interlocked.Increment`?**

- `Interlocked` works on a *single* field. But `IncrementByType` needs to update *two* fields atomically.
- `lock (_lock)` guarantees consistency between the type-specific counter and the total.
- `Reset()` uses the same lock so the GUI thread's reset doesn't race with the worker's increments.

### 📸 The Scan Configuration Snapshot (v14.54)

At scan start, GhostTrace takes a **point-in-time snapshot** of all relevant settings:

```powershell
$global:scanConfig = [PSCustomObject]@{
    DriveText        = $driveBox.Text
    Deep             = $deepCheck.Checked
    Fast             = $fastCheck.Checked
    SpaceAnalyzer    = $spaceAnalyzerCheck.Checked
    Registry         = $regCheck.Checked
    DateFilter       = $dateCheck.Checked
    StartDate        = $dStart
    EndDate          = $dEnd
    SmartExclusions  = $smartCheck.Checked
    CustomExclusions = $exTextBox.Text
    MinSize          = $minSizeBox.Value
    Keywords         = @($global:activeKeywords)
}
```

Every subsequent read (summary, export, grid coloring, CALC SIZES gating) consults `$global:scanConfig`, not the live controls.

### 🎭 The Effective-Exception Helper (v14.54)

When a .NET method call throws inside PowerShell, the exception is wrapped:

```
$_.Exception                →  MethodInvocationException (wrapper)
$_.Exception.InnerException →  the ACTUAL exception
```

The fix: a helper that peels off the wrapper:

```powershell
function Get-EffectiveException($errorRecord) {
    if ($null -eq $errorRecord) { return $null }
    try {
        if ($errorRecord.Exception) {
            return $errorRecord.Exception.GetBaseException()
        }
    } catch {}
    return $errorRecord.Exception
}
```

### 🔁 The Iterative Scanner (No Recursion)

The scanner uses an explicit `Stack<string>` instead of recursive function calls:

```powershell
$stack = [System.Collections.Generic.Stack[string]]::new()
$stack.Push($rootPath)
while ($stack.Count -gt 0) {
    $currentPath = $stack.Pop()
    # ... enumerate $currentPath ...
    if ($isDir -and -not isReparsePoint) {
        $stack.Push($entryPath)
    }
}
```

**Why not recursion?**

- No call-stack depth limit.
- Cancellation is trivial: `if (!$sync.Running) { return }`.
- Traversal state is explicit and inspectable.
- Reparse-point (symlink) skipping is straightforward.

### 🛣️ Long-Path Support

Windows historically has a 260-character path limit. GhostTrace uses the `\\?\` prefix to access long paths:

```powershell
if ($currentPath.StartsWith('\\')) {
    $longPath = '\\?\UNC\' + $currentPath.Substring(2)   # network paths
} else {
    $longPath = '\\?\' + $currentPath                    # local paths
}
```

### 🚫 Reparse-Point Protection

Windows junctions (`mklink /J`) and symlinks create cycles that can trap naive scanners in infinite loops. GhostTrace checks for `FileAttributes.ReparsePoint` and skips.

### 🛡️ Privilege Elevation

GhostTrace requires Administrator privileges to access other users' `AppData`, `HKLM` hives, and system directories. If launched as a standard user, it re-launches itself with `-Verb RunAs`.

### 🚧 The 1,000-Error Cap

A corrupted drive can produce hundreds of thousands of "Access Denied" errors. GhostTrace caps the retained error records at 1,000. Beyond that, errors are still *counted* but not stored as objects.

### 📉 The 50,000-Row UI Cap

The DataGridView becomes slow as it approaches ~100,000 rows. GhostTrace caps the visible grid at 50,000 rows. **But** the background engine continues to store *every* result in `$global:masterRecord`.

---

## 🧪 Edge Cases and Robustness

GhostTrace handles many edge cases that naive scanners miss.

| Edge Case | Handling |
|-----------|----------|
| **Cloud-backed files** | `RECALL_ON_DATA_ACCESS` / `RECALL_ON_OPEN` attribute bits detected → flagged & excluded from size totals |
| **Directory symlinks / junctions** | Skipped via `ReparsePoint` check |
| **Long paths** | Supported via `\\?\` prefix |
| **Access denied errors** | Caught per-file, classified, counted, logged — never crash |
| **File-not-found race conditions** | Caught and logged |
| **Registry hives that don't exist** | Caught and logged |
| **Aborted scans** | Marked `ABORTED BY USER (Partial Results)` |
| **Failed scans** | Marked `FAILED (Fatal Error - Partial Results)` |
| **Excluded scan root** | `Coverage Warning` line in summary |
| **Inaccessible scan root** | `Coverage Warning` line in summary |
| **CALC SIZES read failures** | Marked `Size Unknown (Read Failed)`, distinct from 0 MB |

---

## 🚫 What GhostTrace Deliberately Does NOT Do

Understanding what a tool *won't* do is often as important as what it will.

### ❌ It Does Not Delete Anything
No "Clean" button. No "Auto-Remove." No "Recommended Actions." Every deletion is your decision.

### ❌ It Does Not Modify the Registry
The registry scan is read-only. It uses `Get-ChildItem` on registry paths, nothing else.

### ❌ It Does Not Send Telemetry
No network calls. No update checks. No analytics. No phone-home behavior of any kind.

### ❌ It Does Not Install Anything
No MSI, no EXE bootstrap, no embedded payload. It runs as a `.ps1` file using Windows' built-in PowerShell.

### ❌ It Does Not Read File Contents
GhostTrace enumerates file paths, metadata (size, dates, attributes). It does **not** open files, does **not** read their content, does **not** hash them.

### ❌ It Does Not Do "Smart" Categorization
No ML model. No heuristic "this looks like a cache, safe to delete." Judgment is yours.

### ❌ It Does Not Support Windows 7 / 8 / 8.1
Requires Windows 10+.

### ❌ It Does Not Have a Compiled `.exe` Release
This is intentional. The whole point is that GhostTrace is a plaintext script you can audit before running.

---

## 📜 Version History

### v14.55 *(Current)*

- **Thread-safe `ErrorCounter.Reset()`** — added to the class, used by both reset sites.
- **Robust CSV field escaping** — `ConvertTo-SafeCsvField` quotes and escapes every error-log field.
- **"Remote/Placeholder" relabeling** — TSA's cloud-detection label no longer asserts a specific provider.
- **CALC SIZES failure distinction** — failed size reads shown as `Size Unknown (Read Failed)`.
- **Custom-exclusion tooltip correction**.

### v14.54

- **Corrected `ErrorRecord` → exception classification** via `Get-EffectiveException`.
- **Scan-configuration snapshot** (`$global:scanConfig`) captured at scan start.
- **CALC SIZES visibility fix** — only shown after FAST, non-TSA scan; reentrancy guard.
- **Accurate keyword-match counting** — exact split-based comparison.
- **Faster UI result draining** — time-boxed (~120 ms per 200 ms tick).
- **Excluded/inaccessible scan-root transparency**.
- **CSV Unicode/UTF-8 fix**.
- **TSA cloud/placeholder file size handling**.

### v14.53

- **Fixed cross-runspace error counter** — shared `ErrorCounter` object via `$syncHash`.
- **Fatal-scan status** — `ScanStatus` state machine.
- **TSA keyword leak fix**.

### v14.52

- **Fixed Excel export** and filename sanitization.
- **Font disposal** on form close.
- **`Add-Type` guard** for `ErrorCounter`.

### v14.34 *(Launch Version)*

- First publicly released stable version.

---

## ❓ FAQ

<details>
<summary><b>Do I need to install PowerShell?</b></summary>

No. Windows 10/11 and Windows Server 2016+ ship with PowerShell 5.1 as part of the OS.
</details>

<details>
<summary><b>Can I run GhostTrace without Admin rights?</b></summary>

Technically yes, but you'll miss half the results — other users' `AppData` folders, `HKLM` registry keys, and system directories will return "Access Denied" for every file. GhostTrace automatically requests elevation on launch.
</details>

<details>
<summary><b>Why doesn't GhostTrace have a delete button?</b></summary>

Because that's not what a forensic scanner does. If you want to delete things, use Windows Explorer, `regedit`, or a tool designed for deletion. GhostTrace finds; you decide.
</details>

<details>
<summary><b>Why is the UI capped at 50,000 rows?</b></summary>

To protect against GDI handle exhaustion and UI thread lockups. If a scan returns 500,000 results, the visible grid stops at 50,000 but the background scan continues to store everything. The full dataset is always available via export.
</details>

<details>
<summary><b>My scan completed in 5 seconds. Did it actually work?</b></summary>

Probably yes — FAST scan on an SSD with Smart Exclusions enabled can complete an 800 GB drive in 30 seconds or less. Check the Activity Log for `Scanned : N files`.
</details>

<details>
<summary><b>Can GhostTrace scan network drives?</b></summary>

Partially. If you browse for a specific folder that happens to be a network share, GhostTrace will enumerate it. The "ALL DRIVES" option only covers fixed drives.
</details>

<details>
<summary><b>Why does the summary sometimes say <code>N/A (Fast Mode)</code> for Total Data Size?</b></summary>

Because FAST scan skips file-size lookups. Run DEEP scan, or click **CALC SIZES** after a FAST scan, to get real sizes.
</details>

<details>
<summary><b>What does "Remote/Placeholder (Size N/A)" mean?</b></summary>

It means the file is cloud-backed (e.g. OneDrive Files On-Demand) and isn't physically present on disk. Windows reports its *logical* size, not its *physical* local size, so GhostTrace flags it and excludes it from size totals.
</details>

<details>
<summary><b>Can I add my own keywords to the Smart Exclusions list?</b></summary>

Not in the current UI — the Smart Exclusions list is a fixed built-in set. Use the Custom Exclusions feature instead.
</details>

<details>
<summary><b>Why does the script say "This script must be run as a standalone .ps1 file"?</b></summary>

Because GhostTrace checks `$PSCommandPath` at startup and refuses to run if it can't determine its own file path. This is a safety guard.
</details>

<details>
<summary><b>Does GhostTrace work on ARM Windows?</b></summary>

Yes. The script is architecture-agnostic. `System.IO` and WinForms run fine on ARM64.
</details>

---

## 📄 License

See the accompanying `LICENSE` file (or the repository's About section) for the exact terms.

**Summary:** GhostTrace is provided as-is for personal and professional use. No warranty. No liability for damages arising from use. You are responsible for verifying any action you take based on GhostTrace's output.

---

## 🙏 Acknowledgments

- Built with **PowerShell 5.1** and **Windows Forms**.
- Uses the .NET `System.IO` namespace for filesystem enumeration.
- Uses `Microsoft.Win32.Registry` via PowerShell's `HKLM:` / `HKCU:` drive providers.
- Excel export uses the Excel COM automation interface (falls back to CSV if Excel isn't installed).

---

## 🤝 Contributing

This is primarily a personal project, but **issues and pull requests are welcome** if you have:

- ✅ **Bug reports** with reproduction steps (target drive, keyword, scan mode, observed vs. expected).
- ✅ **Feature suggestions** that align with the project's read-only, single-file philosophy.
- ✅ **Documentation improvements** — corrections, clarifications, examples.

Please **do not** submit:

- ❌ Suggestions to add automated deletion.
- ❌ Requests to compile to `.exe` (it's intentionally a script).
- ❌ Suggestions to add a "cleanup wizard."

---

## 👋 Final Word

GhostTrace exists because I was tired of the two options the software world gave me:

1. **Trust an uninstaller** that silently leaves 8 GB behind.
2. **Trust a junk cleaner** that might delete my registry or my project files.

There's a third option: **look for yourself.**

GhostTrace is that third option. It's a script. It's a lens. It doesn't clean up after you — but it makes sure you *know* what you're cleaning up *before* you do.

**Use it well.**

— **ROXX**

---

<div align="center">

**⭐ If GhostTrace saved your disk space, consider starring the repo ⭐**

</div>
