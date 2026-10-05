# HW0 — Getting Started: Environment, Workspace & Homework Submission

**Course:** 83-612 – Digital VLSI Design, Bar-Ilan University
**Year:** 2026–27

This README is everything you need before starting HW1: how to work on the course Linux server, how every homework repository is laid out, and how to submit your homework with Git. There is nothing to submit for HW0, but **read all of it** — the rules in [Section 4](#4-submitting-homework-with-git) apply to every assignment.

> Based on course material by Prof. Adam Teman (EnICS Labs).

---

## 1. Using the Linux environment

You are expected to become comfortable with a Linux terminal during the course. If you need a refresher, see the EnICS Labs Linux tutorial at <https://enicslabs.com/tutorials/> or the [YouTube playlist](https://www.youtube.com/playlist?list=PLZU5hLL_713x1w0kaMFXA7k2SGQQZOO-G).

### 1.1 Opening a terminal

Right-click on the desktop and choose **Open Terminal**. The terminal opens in your home folder (`/data/home/<username>/` or similar).

### 1.2 Your workspace and the `tsmc65` alias

The course uses the **TSMC 65 nm LP** technology. All work must be done under `/data/project/` — never in your home directory. The `tsmc65` alias creates your work area, sets up the licenses and environment variables for the technology, and `cd`s into it:

```text
%> tsmc65
Creating user workarea for project tsmc65
Project tsmc65 setup is complete. Entering your workspace
vlsi1:<user>:/data/project/tsmc65/users/<user>/ws>
```

**Run `tsmc65` in every new terminal before doing anything else.** Afterwards, useful environment variables exist, e.g. `$kit_dir` points to the root of the PDK.

### 1.3 What is on the server

| What | Where |
|------|-------|
| EDA tools (and their manuals) | `/opt/` — e.g. Cadence tools under `/opt/cadence/` |
| TSMC 65LP design kit (PDK) | `/data/tsmc/65LP/` |
| TSMC 65 nm digital libraries (standard cells, memories, I/Os) | `/data/tsmc/65LP/dig_libs/` |

### 1.4 Editors

Use a real code editor: **vi/gvim**, **emacs**, or **VS Code** (run `code`). To get Windows-style shortcuts (Ctrl-C/Ctrl-V/Ctrl-S…) in gvim, add this line to `~/.gvimrc`:

```vim
source $VIMRUNTIME/mswin.vim
```

### 1.5 Viewing files

- PDFs: `acroread file.pdf &`
- Browse a directory graphically: `thunar .`

---

## 2. The project workspace (directory structure)

Every homework repository from HW2 onwards follows the same layout. EDA tools take design and technology information as inputs and export files and reports, and many exports become inputs of the next tool — so we keep a strict structure:

```text
<project root>/
├── workspace/    # run ALL tools from here
├── sourcecode/   # (called src/ in HW2–HW3)
│   ├── rtl/      #   RTL files
│   └── tb/       #   testbenches
├── inputs/       # design definitions, SDC, MMMC, floorplan/CTS specs
├── libraries/    # library definition scripts (libraries.*.tcl, *.f)
├── scripts/      # tool scripts (simulation, genus.tcl, innovus.tcl, ...)
├── apps/         # software applications (HW4+)
├── reports/      # reports dumped by the tools — and your submission reports
└── export/       # exported netlists, SDF, databases, ...
```

- **`workspace/`** — tools create a lot of clutter (logs, temp files, scratch dirs). Running them from a dedicated directory keeps the clutter away from your "golden" files; every script refers to everything else through `../`. Consequences:
  - `rm -rf workspace/*` cleans up without risk to anything important.
  - You can keep one run's logs and start another in a parallel directory (`workspace2/`) without changing any script.
  - No need to jump between per-tool directories.
- **`sourcecode/`** (`src/` in HW2–3) — RTL in `rtl/`, testbenches in `tb/`.
- **`inputs/`** — inputs shared between tools: design definitions, SDC files, etc.
- **`libraries/`** — library definition scripts.
- **`scripts/`** — scripts for running the tools (all tools in one place).
- **`apps/`** — application (software) files.
- **`reports/`** — reports from every stage of the flow.
- **`export/`** — exported netlists, saved states, etc.

### 2.1 Design-specific input files — `<TOPLEVEL>.<ext>`

Design-specific inputs are named after the design's top-level module. This forces you to actually edit them for each new design instead of blindly copying them from a previous project.

| File | Purpose |
|------|---------|
| `<TOPLEVEL>.defines` | Basic design definitions, mostly in the `$design()` array. `$design(TOPLEVEL)` must be set **before** sourcing it. |
| `<TOPLEVEL>.sdc` | Timing constraints, common to all stages of the flow. Stage-specific settings go inside `if` statements, so a change is never forgotten in one tool. |
| `<TOPLEVEL>.mmmc` | Multi-mode multi-corner definitions for the Cadence tools. *(Not used during synthesis in this course.)* |
| `<TOPLEVEL>.io` | I/O pad placement for the top level. |
| `<TOPLEVEL>.floorplan.defines` | Floorplan-specific definitions. |
| `<TOPLEVEL>.ccopt` | Clock-tree synthesis specification for CCOpt. |
| `<TOPLEVEL>.cpf` | Common Power Format. *(Not used in this course.)* |

### 2.2 Library definition files — `libraries.<library>.tcl`

Shared by every project that uses the technology/library — one per PDK and one per library:

| File | Contents |
|------|----------|
| `libraries.TSMC65LP.tcl` | TSMC 65 nm LP technology (no cells) |
| `libraries.ARM65LP_SC.tcl` | ARM standard-cell libraries for TSMC 65LP |
| `libraries.ARM65LP_SRAM.tcl` | ARM SRAM compiler outputs for TSMC 65LP |
| `libraries.TSMC65_IO.tcl` | TSMC 65 nm I/O libraries |

### 2.3 Tcl arrays used by the course scripts

Virtually all EDA tools use Tcl as their shell. The provided scripts keep their variables in a few Tcl arrays: fewer accidental name collisions, more readable code, and easy passing to procedures (just declare the array `global`).

| Array | Holds |
|-------|-------|
| `$design()` | Everything about this design (mostly set in `<TOPLEVEL>.defines`), including design-specific paths |
| `$paths()` | Important paths in the Linux environment — usually all you change when moving to a new computing environment |
| `$tech_files()` | Technology files: PDK files, `.lib`/`.lef` of standard cells, memories, ... |
| `$tech()` | Technology-specific values, e.g. the clock-buffer list or minimum well-tap spacing |

---

## 3. Homework rules (all assignments)

- **Grading is by an in-lab exam.** Every homework has an in-lab exam, and **your grade for the homework is determined by the exam** — not by when you submitted.
- **Submit before the lab session starts.** Your homework must be pushed to your branch **before the lab session in which you take its exam begins**. A push made after the session has started does not count, and you will not be allowed to take the exam. **No submission before the session — no exam.**
- **Exam dates** are listed in the **course syllabus**, pinned in the `#important-resources` channel of the **DVD2026-27 Discord server**.
- **Late exams:** for every week you take the exam after its scheduled date, **5 points are deducted** from that homework's grade:

  | Exam taken | Deduction |
  |------------|-----------|
  | On the scheduled date | — |
  | Up to 1 week late | −5 points |
  | Up to 2 weeks late | −10 points |
  | Up to 3 weeks late | −15 points |
  | More than 3 weeks late | The exam can no longer be taken |

  Extensions only under the special circumstances described in the syllabus.
- **Interfaces are fixed.** Module names, port names and output-file formats given in an assignment are checked automatically — if they differ, the tests cannot run and you lose the points.
- **Name and ID** must appear wherever the assignment asks for them (e.g. report headers). Missing or wrong details cost points.
- The README of each homework repository is the complete specification of that assignment.

---

## 4. Submitting homework with Git

Every assignment lives in its own repository under <https://github.com/DVD2027>. The `main` branch only contains a README; the assignment files are at tag **`v1.0`**. You submit by pushing **a branch named exactly your 9-digit ID number**.

```bash
# 1. Enter your workspace and a course directory
tsmc65
mkdir -p DVD && cd DVD

# 2. Clone the assignment (replace hwN)
git clone git@github.com:DVD2027/hwN.git
cd hwN

# 3. Start from tag v1.0 on a new branch named after your ID
git checkout v1.0 -b <ID_NUMBER>

# ... do the work ...

# 4. Stage, commit and push to YOUR branch
git add <files and directories listed in the assignment>
git commit -m "HWN solution"
git push origin <ID_NUMBER>
```

**Rules**

- The branch name must be **exactly** your ID number (e.g. `206123456`) — no prefixes, suffixes or names.
- **Never push to `main`** or to any branch other than your own.
- Commit only what the assignment asks for. Do not commit workspace clutter (`xcelium.d/`, `waves.shm/`, logs) — the provided `.gitignore` files already exclude most of it.
- You can push as many times as you like, but **your final push must happen before the lab session of your exam starts** — the exam is based on the work in your branch at that time.
