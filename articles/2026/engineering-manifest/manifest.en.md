# Engineering Manifest: Standard for Organizing Files and Folders

## 1 Hard Drive Naming

Each physical disk or array in the system must have a unique, understandable name (Volume Label).

1. For the system drive, use the label: **WIN_SYS** (Drive C:).
2. For the primary work drive, use the label: **MAIN_DATA** (Drive D:).
3. For backup and archive drives, use the format **NAME_NC**, where **NAME** is the archive category, **N** is the storage sequence number, and **C** is the drive letter within the mirrored array.

Example of two drives in one mirror:

* `ARCHIVE_001A`
* `ARCHIVE_001B`

Example of extending the archive with a second mirror:

* `ARCHIVE_002A`
* `ARCHIVE_002B`

---

## 2 Folder and File Naming Culture

Naming rules for folders and files may differ depending on whether the name is a username or just a file name.

### 2.1 Naming Folders and Files

1. **Mandatory extensions**: Do not create files without extensions. A text file must have `.txt`, a configuration file must have `.ini`/`.conf`, and so on.
2. **Character case**: Use only lowercase Latin letters in user-defined file and folder names.
   * *Exception*: Physical drive labels and folders created by the operating system by default.
3. **No special characters**: Do not use special characters in names: `.`, `'`, `(`, `)`, `/`, `/`, `&`, `$`, and others.
4. **No spaces**: Never use spaces. Replace them with underscores `_`.
   * *Correct*: `/new_year_photo`
5. **No garbage names**: Using meaningless names that convey no information is strictly prohibited. Examples of prohibited names: `/1`, `/2`, `/new_folder_1`, `/x5`, `/various`.
6. **Authorship principle**: Use the person's name or company name before the file name itself.
   * *Example*: `/ivan_photos`, `/rocketamusements_cad_models`
   * When using Last Name + First Name, always maintain strict order: `lastname_firstname` or `lastname_firstname_initial` (e.g.: `/pudov_ivan_photos`, `/pudov_ivan_v_photos`).
7. **Brevity without redundancy**: Avoid overly long names, but also do not use extreme abbreviations that lose meaning.
   * *Correct*: `/rocketamusements_logotype`
   * *Incorrect (too long)*: `/rocketamusements_company_logo`
   * *Incorrect (overly abbreviated)*: `/rockamus_logo`
8. **Three level nesting rule**: The physical nesting depth of folders at the user level must not exceed three levels.
   * `/1` (Level 1) ➔ `/1/2` (Level 2) ➔ `/1/2/3` (Level 3 — final level)
9. **Dating and version numbering**: When it is necessary to separate similar folders or files by time or revisions:
   * **By date**: Start the folder name with the date in strict international format `YYYYMMDD` (e.g.: `/20261115_patterns`).
   * **By number**: Use a fixed two-digit format `00`, `01`, `02` (e.g.: `/patterns_00`, `/patterns_01`).
   * **By version**: If files are versions of the same document, add the letter `v` before the number (e.g.: `/patterns_v00`, `/patterns_v01`).

### 2.2 Naming User or Developer Folders

For author folders at the second level of hierarchy, a strict corporate pattern applies:

> `firstname.lastname.number` (or `firstname.first_letter_of_lastname.number`)

All characters are written strictly in lowercase. If the author's last name is unknown, use their network nickname. The number is added only in case of name or nickname conflicts, to prevent accidental data overwriting.

Example of using the pattern within a group:

```txt
D:\projects\
    ├── john.doe\                 (First Name + Full Last Name)
    ├── john.s\                   (First Name + First Letter of Last Name)
    └── john.s.2\                 (Exact namesake of the previous author, number added)
```

---

## 3 Reserved Names and Service Markers

Some names have very specific purposes and thus serve as content markers.

### 3.1 File Names

* `read_me`, `README` — A text file in the root of a folder containing important comments, structure descriptions, or instructions for that directory.
* `preview` — An image containing a collection of thumbnails (previews) of all graphic files in that folder.

Minimum structure of `README.md`:

```txt
# Folder Name

## Author

## Creation / Update Date

## Contents (briefly)
```

### 3.2 Folder Names

* `temp` — Sandbox folder (similar to Downloads). Anything inside may be permanently deleted at any time without warning. Files in temp with a lifespan exceeding 7 days are subject to automatic deletion (configure via Windows Task Scheduler).
* `incoming` — Temporary buffer for files that require sorting, analysis, and subsequent organization.
* `archive` — Folder for storing data that cannot be deleted but is guaranteed not to be used in daily work.
* `home` — Skeleton folder containing reference user profiles for backing up terminal environments.
* `shared` — Dedicated shared folder with local network access.
* `family` — Top-level container of the highest privacy and importance for personal family data.

### 3.3 System Underscore Marker

An underscore at the beginning of a root folder name (e.g., `_base`, `_altium`) radically changes its status. It means: **"Attention! This is system infrastructure or a category, not a user/project name. Inside is a complex automated system that requires caution."**

---

## 4. Data Drive Structure

All data on drive D: is divided into two global categories: User Content and System Infrastructure. The root of the drive maintains an ideal flat list of 6 main folders.

```text
D:│
  ├── ⚠️ _base/             (INFRASTRUCTURE: system black box for software)
  │
  ├── 📁 family/            (CONTENT: highest family and legal value)
  ├── 📁 archive/           (CONTENT: static long-term backups and archives)
  ├── 📁 shared/            (CONTENT: publicly accessible network files, media, and installers)
  ├── 📁 home/              (CONTENT: skeleton dotfile configurations for WSL Linux)
  └── 📁 projects/          (CONTENT: live workspace for Windows-native CAD)
```

### 4.1 Program-Required Data

This folder is the system foundation. It conceals under its protection the finicky clutter of programs, development environment configurations, and cross-cutting libraries. The user does not perform daily work inside this folder.

Inside `_base`, data is divided into three isolated containers:

#### 4.1.1 PCB CAD Libraries

Cross-cutting component databases. Projects from across the computer reference these files directly.

* `_base/eda_libraries/mentor/` — Central libraries for Mentor Xpedition / PADS (`.lmc` configuration files, symbols, and pads).
* `_base/eda_libraries/cadence/` — Component databases for Cadence Allegro / OrCAD.
* `_base/eda_libraries/kicad/` — Global symbol and footprint libraries for KiCad (`.pretty` folders).
* `_base/eda_libraries/footprint_expert/` — PCB Footprint Expert utility databases for generating footprints per IPC standards.

#### 4.1.2 Simulation Libraries

Isolated mathematical and physical models for simulators. They may be used in parallel across different CAD tools.

* `_base/sim_libraries/cadence/` — Physical models for Cadence P-Spice circuit simulation.
* `_base/sim_libraries/mentor/` — Models for Mentor Graphics simulation (IBIS models for signal analysis in HyperLynx).

#### 4.1.3 Software Service Environments

Each software ecosystem has exactly one subfolder here for storing local interface settings, themes, automation scripts, and hidden logs:

* `_base/environments/mentor/` — The Mentor Graphics WDIR working directory is redirected here.
* `_base/environments/cces/` — Metadata (`.metadata` folder) for Analog Devices CrossCore Embedded Studio is directed here. Live firmware code is not stored here.

#### 4.1.4 Use of Symbolic Links in the Root Folder

Creating symlinks in the root of D:/ is prohibited. All references to `_base` must be configured within program settings, not at the file system level.

---

### 4.2 Valuable Data, Shared Access, and WSL Backup

#### 4.2.1 Category: Valuable Data

* `D:/family/` — Personal family documents.
  * `family/business_data/` — Accounting and business documents.
  * `family/documents/` — Scans of passports, diplomas, certificates.
  * `family/job_search/` — Professional portfolio, resumes, cover letters.
* `D:/photos/` — At the top level to ensure folder nesting with photo sessions inside `raw_files` does not exceed 3 levels.
  * `photos/catalogs/` — Lightroom and Capture One databases.
  * `photos/copyright_images/` — Watermark graphics and copyright templates.
  * `photos/raw_files/` — Original shots in RAW and JPEG formats.
  * `photos/exported/` — Finished photos exported for viewing.
* `D:/archive/cloud_mirror/` — Local snapshots and backups of files from cloud storage.

#### 4.2.2 Category: Shared Access

The zone with the lowest content value, open over the local network to other users.

* `shared/incoming/` — Inbound sandbox buffer for unsorted files.
* `shared/installers/` — Distributions and clean program installers.
* `shared/video/` — Movies, TV series, and training video courses.
* `shared/music/` — Audio files and music collections.
* `shared/wallpapers/` — Desktop wallpapers.
* `shared/books/` — Books and documentation.

#### 4.2.3 Category: WSL Linux Backup

A static "skeleton" donor folder. Live work in the WSL terminal is conducted strictly inside the native ext4 virtual disk (`/wsl$/...`). The `D:/home/valery/` folder stores backup copies of your configuration files (dotfiles: `.bashrc`, `.tmux.conf`, git configurations, ssh keys) for instant deployment of a new Linux environment in case of failure or OS reinstallation.

---

## 5 Rules for Organizing the Projects Folder

`D:/projects/` is the primary, live workspace for Windows-native development (Altium, KiCad, Unity, Creo).

Two strict data architecture laws apply inside it:

1. **AUTHORSHIP PRINCIPLE (Level 1)**: The first directory identifies the author, client, or source of the code (`hww` — you, `ti` — Texas Instruments, `gary` — colleague).
2. **TECHNICAL MARKER (Level 2)**: Folders at the second level must begin with an underscore `_` and define only the development environment/software. This creates a single entry point (Workspace) for programs.
3. **VERSION CONTROL STATUS (Level 3)**: The final project is located strictly at the 3rd nesting level. The dynamic synchronization status of the project with the cloud is determined strictly by the prefix in the folder name:
   * **`.git`** — A serious project under Git management with a GitHub repository.
   * **no extension** — A local draft, sandbox, or quick test without version control.

### 5.1 Reference Visual Diagram of the Projects Folder

```txt
D:/projects/
    │
    ├── ti/                           (Level 1: Author — Texas Instruments)
    │   └── _altium/                  (Level 2: TI PCB CAD)
    │       └── ref_design/           (Level 3: END. Downloaded reference schematic)
    │
    └── hww/                          (Level 1: Author — You / personal projects)
        │
        ├──_altium/                   (Level 2: PCB CAD — Altium Designer)
        │   ├── power_supply.git/     (Level 3: END. Your production board on GitHub)
        │   └── test_amp/             (Level 3: END. Quick amplifier schematic sketch)
        │
        ├── _cces/                    (Level 2: Unified Workspace for CrossCore environment)
        │   ├── dsp_filter.git/       (Level 3: END. Production processor firmware on GitHub)
        │   └── blinky_test/          (Level 3: END. Quick local LED blink test)
        └── jtag_adaptor/             (Level 3: Multi-system project)
```

The `jtag_adaptor` folder is an example of a multi-system project that contains a spectrum of technologies. For example, a PCB with embedded software.

```txt
jtag_adaptor/                                (Level 3: Multi-system project)
jtag_adaptor/_cces/                          (Level 4: Analog Devices projects)
jtag_adaptor/_cces/jtag_adaptor_firmware     (Level 5: Analog Devices project)
jtag_adaptor/_mentor/                        (Level 4: Mentor projects)
jtag_adaptor/_mentor/jtag_adaptor_pcb        (Level 4: PCB project)
```

### 5.2 Project Migration Rule

If a local draft (`blinky_test`) grows into a full-fledged task and a repository is created for it online, the project folder is initialized in Git, and the prefix in its name changes to `blinky_test.git`. The physical location of the project within the software folder `_cces` does not change, ensuring absolute path stability and preventing broken links in the development environment.

#### Projects for Third-Party Companies (Commercial Development)

If a project is created by you on commission, under contract, or as part of a collaboration with a third-party company, a folder with the official name of that company (in lowercase Latin letters) is created at the first level of the hierarchy.

Even though you are the author of the code, the project is physically placed inside the client company's folder, as they are the owner of the final product or development context.

Example of organizing commercial development:

```txt
D:\projects\
    └── yandex\                    (Level 1: Client company / Context)
        └── _cces\                 (Level 2: Software category — CrossCore environment)
            ├── telecom_board.git\ (Level 3: END. Production project repository)
            └── test_spi\          (Level 3: END. Local draft for bus testing)
```

### 5.4 Grouping Third-Party Projects (Collective Authorship)

If a global project or open platform has several independent authors whom you do not know personally (e.g., GitHub repositories from different developers for a single device), the first level of the hierarchy is given to the Group.

The name of such a folder must begin with an underscore and end with the suffix `.group`. This signals that the folder is a technical container, not a specific person's name. Author names are then moved to the second level.

Example of organizing a group for the Aleste retro computer:

```txt
D:\projects\
    └── _aleste.group\            (Level 1: Artificial collective author)
        ├── gary\                 (Level 2: Specific developer Gary)
        │   └── loc_power_sch\    (Level 3: END. Power supply schematic from Gary)
        └── ron\                  (Level 2: Specific developer Ron)
            └── git_main_board\   (Level 3: END. Motherboard repository from Ron)
```

### 5.5 Archiving Projects and Data

Closed projects that should not be deleted are marked in one of two ways:

1. Moved to the `_archive` folder
2. Suffix `.archive` or `.deprecated` is added