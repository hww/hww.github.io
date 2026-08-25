# ENGINEERING MANIFEST: FILE AND FOLDER ORGANIZATION STANDARD

## 1. HARD DISK NAMING

Every physical disk or array in the system must have a unique, meaningful name (Volume Label).

1. System drive label: **WIN_SYS** (Drive C:).
2. Primary workspace drive label: **MAIN_DATA** (Drive D:).
3. Backup and archive drives use the format **NAME_NC**, where **NAME** is the archive category, **N** is the storage sequence number, and **C** is the drive letter in the mirrored array.

Example of two drives in one mirror:

* `ARCHIVE_001A`
* `ARCHIVE_001B`

Example of expanding the archive with a second mirror:

* `ARCHIVE_002A`
* `ARCHIVE_002B`

---

## 2. FOLDER AND FILE NAMING CULTURE

1. **Mandatory extensions**: Do not create files without extensions. Text files must have `.txt`, configuration files `.ini`/`.conf`, and so on.
2. **Character case**: Use only lowercase Latin letters in user file and folder names.
   * *Exception*: Physical disk labels and folders created by the operating system by default.
3. **No special characters**: Do not use special characters in names: `.`, `'`, `(`, `)`, `/`, `&`, `$`, and others.
4. **No spaces**: Never use spaces. Replace them with underscores `_`.
   * *Correct*: `/new_year_photo`
5. **No garbage names**: It is strictly forbidden to use meaningless names that convey no information. Examples of forbidden names: `/1`, `/2`, `/new_folder_1`, `/x5`, `/various`.
6. **Authorship principle**: Use the person's name or company name before the file name.
   * *Example*: `/ivan_photos`, `/rocketamusements_cad_models`
   * When using Last Name + First Name, always maintain strict order: `lastname_firstname` or `lastname_firstname_initial` (e.g.: `/pudov_ivan_photos`, `/pudov_ivan_v_photos`).
7. **Conciseness without redundancy**: Avoid overly long names, but do not use extreme abbreviations that lose meaning.
   * *Correct*: `/rocketamusements_logotype`
   * *Incorrect (too long)*: `/rocketamusements_company_logotype`
   * *Incorrect (over-abbreviated)*: `/rockamus_logo`
8. **THE THREE-LEVEL RULE**: Physical folder nesting depth at the user level must not exceed three levels.
   * `/1` (Level 1) ➔ `/1/2` (Level 2) ➔ `/1/2/3` (Level 3 — final)
9. **Dating and version numbering**: When it is necessary to separate similar folders or files by time or revisions:
   * **By date**: Start the folder name with the strict international format `YYYYMMDD` (e.g.: `/20261115_patterns`).
   * **By number**: Use a fixed two-digit format `00`, `01`, `02` (e.g.: `/patterns_00`, `/patterns_01`).
   * **By version**: If files are versions of the same document, add the letter `v` before the number (e.g.: `/patterns_v00`, `/patterns_v01`).

---

## 3. RESERVED NAMES AND SERVICE MARKERS

### File Names

* `read_me`, `README` — A text file in the root of a folder containing important comments, structure descriptions, or instructions for that directory.
* `preview` — An image containing a collection of thumbnails (previews) of all graphic files in that folder.

#### Minimal `README.md` Structure

```txt
# Folder Name

## Author

## Creation / Update Date

## Contents (brief)
```

### Folder Names

* `temp` — Sandbox folder (like downloads). Everything in it may be permanently deleted at any time without warning. Files in `temp` older than 7 days are subject to automatic deletion (configure via Windows Task Scheduler).
* `incoming` — Temporary buffer for files that require review, analysis, and subsequent sorting.
* `archive` — Folder for data that cannot be deleted but is guaranteed not to be used in daily work.
* `home` — Skeleton folder containing reference user profiles for terminal environment backups.
* `shared` — Dedicated shared folder with local network access.
* `family` — Top-level container of highest privacy and importance for personal family data.

### The Underscore System Marker `_`

An underscore at the beginning of a root folder name (e.g., `_base`, `_altium`) radically changes its status. It means: **"Attention! This is system infrastructure or a category, not a user/project name. Inside lies a complex automated system that requires caution."**

---

## 4. DATA DRIVE STRUCTURE (MAIN_DATA / Drive D:)

All data on drive D: is divided into two global categories: User Content and System Infrastructure. The root of the drive maintains an ideal flat list of 6 core folders.

```text
D:│
  ├── ⚠️ _base/             (INFRASTRUCTURE: system black box for software)
  │
  ├── 📁 family/            (CONTENT: highest family and legal value)
  ├── 📁 archive/           (CONTENT: static long-term backups and archives)
  ├── 📁 shared/            (CONTENT: network-shared files, media, installers)
  ├── 📁 home/              (CONTENT: skeleton dotfiles for WSL Linux)
  └── 📁 projects/          (CONTENT: active workspace for Windows-native CAD)
```

### Section 4.1: Program-Required Data (`D:/_base/`)

This folder is the system foundation. It conceals within its protection the capricious clutter of programs, development environment configurations, and cross-cutting libraries. Daily user work inside this folder does not occur.

Within `_base`, data is divided into three isolated containers:

#### 1. `_base/eda_libraries/` (EDA/CAD Component Libraries)

Cross-cutting component databases. Projects across the entire computer reference these files directly.

* `_base/eda_libraries/mentor/` — Central libraries for Mentor Xpedition / PADS (`.lmc` config files, symbols, and pads).
* `_base/eda_libraries/cadence/` — Component databases for Cadence Allegro / OrCAD.
* `_base/eda_libraries/kicad/` — Global symbol and footprint libraries for KiCad (`.pretty` folders).
* `_base/eda_libraries/footprint_expert/` — PCB Footprint Expert utility databases for IPC-compliant footprint generation.

#### 2. `_base/sim_libraries/` (Simulation Libraries)

Isolated mathematical and physical models for simulators. May be used in parallel across different CAD tools.

* `_base/sim_libraries/cadence/` — Physical models for Cadence P-Spice circuit simulation.
* `_base/sim_libraries/mentor/` — Models for Mentor Graphics simulation (IBIS models for HyperLynx signal analysis).

#### 3. `_base/environments/` (Software Service Environments)

Each software ecosystem has strictly one subfolder here for storing local interface settings, themes, automation scripts, and hidden logs:

* `_base/environments/mentor/` — Mentor Graphics WDIR workspace directory is redirected here.
* `_base/environments/cces/` — Analog Devices CrossCore Embedded Studio metadata (`.metadata` folder) is directed here. Live firmware code is not stored here.

#### Symbolic Links in Root Folder

Creating symlinks in the root of `D:/` is prohibited. All references to `_base` must be configured within program settings, not at the file system level.

---

### Section 4.2: Valuable Content, Shared Access, and WSL Backup

#### 1. Category: Valuable (`D:/family/`, `D:/photos/`, `D:/archive/`)

* `D:/family/` — Personal family documents.
  * `family/business_data/` — Accounting and business documents.
  * `family/documents/` — Scans of passports, diplomas, certificates.
  * `family/job_search/` — Professional portfolio, resumes, cover letters.
* `D:/photos/` — At the top level to ensure folder nesting for photoshoots within `raw_files` does not exceed 3 levels.
  * `photos/catalogs/` — Lightroom and Capture One databases.
  * `photos/copyright_images/` — Watermark graphics and copyright overlays.
  * `photos/raw_files/` — Original RAW and JPEG captures.
  * `photos/exported/` — Final exported photos for viewing.
* `D:/archive/cloud_mirror/` — Local snapshots and backups of files from cloud storage.

#### 2. Category: Shared Access (`D:/shared/`)

The zone with the lowest content value, open via local network to other users.

* `shared/incoming/` — Inbound buffer/sandbox for unsorted files.
* `shared/installers/` — Distributions and clean program installers.
* `shared/video/` — Movies, series, and training video courses.
* `shared/music/` — Audio files and music collections.
* `shared/wallpapers/` — Desktop wallpapers.
* `shared/books/` — Books and documentation.

#### 3. Category: WSL Linux Backup (`D:/home/`)

A static "skeleton" donor folder. Live work in the WSL terminal is strictly conducted inside the native ext4 virtual drive (`/wsl$/...`). The folder `D:/home/username/` stores backup copies of your configuration files (dotfiles: `.bashrc`, `.tmux.conf`, git configs, ssh keys) for instant deployment of a new Linux environment during a crash or OS reinstall.

---

## 5. PROJECTS FOLDER ORGANIZATION RULES

The `D:/projects/` folder is the primary, active workspace for Windows-native development (Altium, KiCad, Unity, Creo).

Within it, two rigid data architecture laws apply:

1. **AUTHORSHIP PRINCIPLE (Level 1)**: The first directory defines the author, customer, or code source (`hww` — you, `ti` — Texas Instruments, `gary` — colleague).
2. **TECHNICAL MARKER (Level 2)**: Folders at the second level must begin with an underscore `_` and define only the development environment/software. This creates a single entry point (Workspace) for programs.
3. **VERSION CONTROL STATUS (Level 3)**: The final project is located strictly at the 3rd nesting level. The dynamic sync status of the project with the cloud is determined strictly by the prefix in the folder name:
   * **`.git`** — A serious project under Git control, with a GitHub repository.
   * **no extension** — A local draft, sandbox, or quick test without version control.

### Reference Visual Scheme of the `projects` Folder

```txt
D:/projects/
    │
    ├── ti/                           (Level 1: Author — Texas Instruments)
    │   └── _altium/                  (Level 2: TI's PCB CAD)
    │       └── ref_design/           (Level 3: END. Downloaded reference schematic)
    │
    └── hww/                          (Level 1: Author — You / personal projects)
        │
        ├──_altium/                   (Level 2: PCB CAD — Altium Designer)
        │   ├── power_supply.git/     (Level 3: END. Your production board on GitHub)
        │   └── test_amp/             (Level 3: END. Quick amplifier schematic draft)
        │
        ├── _cces/                    (Level 2: Unified CrossCore Workspace)
        │   ├── dsp_filter.git/       (Level 3: END. Production firmware on GitHub)
        │   └── blinky_test/          (Level 3: END. Quick local LED blink test)
        └── _system/                  (Level 3: Multi-system project)
```

The special folder `_system` for storing multi-system projects that encompass a spectrum of technologies. For example, a PCB with embedded software.

### Project Migration Rule

If a local draft (`blinky_test`) grows into a full-fledged task and a repository is created for it online, the project folder is initialized with Git, and the prefix in its name changes to `blinky_test.git`. The physical location of the project within the software folder `_cces` does not change, ensuring absolute path stability and preventing broken links in the development environment.

#### Projects for Third-Party Companies (Commercial Development)

If a project is executed by you on commission, under contract, or as part of collaboration with a third-party company, then a folder with the official name of that company (lowercase Latin) is created at the first level of the hierarchy.

Even though you are the code author, the project is physically placed inside the customer company's folder, as they are the owner of the final product or development context.

Example of commercial development organization:

```txt
D:\projects\
    └── yandex\                    (Level 1: Customer company / Context)
        └── _cces\                 (Level 2: Software category — CrossCore environment)
            ├── telecom_board.git\ (Level 3: END. Production project repository)
            └── test_spi\          (Level 3: END. Local draft for SPI testing)
```

#### Grouping External Projects (Collective Authorship)

If a single global project or open platform has multiple independent authors whom you do not know personally (e.g., GitHub repositories downloaded from various developers for a single device), the first level of the hierarchy is given to a Group.

The name of such a folder must begin with an underscore and end with the suffix `-group`. This signals that the folder is a technical container, not a specific person's name. Author names then shift to the second level.

Example of organizing a group for the Aleste retro-computer:

```txt
D:\projects\
    └── _aleste-group\            (Level 1: Artificial collective author)
        ├── gary\                 (Level 2: Specific developer Gary)
        │   └── loc_power_sch\    (Level 3: END. Power supply schematic from Gary)
        └── ron\                  (Level 2: Specific developer Ron)
            └── git_main_board\   (Level 3: END. Motherboard repository from Ron)
```

#### User/Developer Folder Naming

For author folders at the second level of the hierarchy, a strict corporate pattern applies:

> `firstname.lastname.number` (or `firstname.last_initial.number`)

All characters are strictly lowercase. If the author's last name is unknown, their network nickname is used. A number is added only in case of identical names or nicknames to prevent accidental data overwriting.

Example of pattern usage within a group:

```txt
D:\projects\
    ├── john.doe\                 (Full First + Last Name)
    ├── john.s\                   (First Name + Last Initial)
    └── john.s.2\                 (Full duplicate of previous author, number added)
```

#### Project Archiving

Closed projects that are not subject to deletion are marked in one of two ways:

1. Moved to the `_archive` folder
2. Suffix `.archive` or `.deprecated` is added


## Links

+ [GitHub version of this document](https://github.com/hww/engineering-manifest)