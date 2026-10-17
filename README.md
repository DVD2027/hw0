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

These rules follow the **course syllabus**, which is pinned in the `#important-resources` channel of the **DVD2026-27 Discord server**. If anything here conflicts with the syllabus, the syllabus wins.

### 3.1 Weight and grading

Homework is **38%** of the course grade, and **submitting every assignment is mandatory**. To get a homework grade at all, you must submit all of them. The assignments are not equally weighted:

| HW1 | HW2 | HW3 | HW4 | HW5 | HW6 |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 10% | 15% | 25% | 5% | 20% | 25% |

Each assignment is graded in two parts:

| Part | Weight | What is checked |
|------|:------:|-----------------|
| **Automatic grader** | 20% | Runs on your submission. It checks that you met the deadline and basic sanity checks: required files are present, interfaces and output formats are correct, and the results make sense. |
| **In-lab evaluation** | 80% | In the **lab after the deadline**, the staff ask you questions about the assignment — both theory and your actual results, which you show and explain. |

**A homework grade is given only after the in-lab evaluation.**

### 3.2 Deadlines

- **Deadline:** **Sunday night at 23:59** of the submission week. Due dates are in the syllabus timeline. **They are final and will not be moved.**
- **Submitting is the entry ticket to the evaluation lab.** You will be evaluated only if your homework was pushed to your branch **before that lab session starts**. **No submission — no evaluation.**
- **Late submission or missed evaluation:** if you missed the deadline, or were not present at the evaluation lab, submit and be evaluated in a following lab session. **Every week of delay costs 5 points off that assignment's grade** (out of 100):

  | Evaluated in | Deduction from the assignment's grade |
  |--------------|---------------------------------------|
  | The lab after the deadline | — |
  | 1 week later | −5 points |
  | 2 weeks later | −10 points |
  | 3 weeks later | −15 points |
  | More than 3 weeks later | Evaluation is no longer possible |

### 3.3 Extensions and excuses

- **There are no extensions** other than for approved reasons: Miluim, your own wedding, death of a family member, birth, illness, and so on. The syllabus lists each case and its predetermined extension.
- **Upload every excuse to the course excuses Google Form** (linked in the syllabus). The form is not answered: if you meet the criteria, the extension applies automatically. Do not ask the staff about it.
- An excuse known in advance must be reported **before** it happens.

### 3.4 Working in pairs

- Assignments may be done **in pairs** — **highly recommended**. Groups of three or more are not approved.
- Each pair designates **one representative**, who signs up through the course registration form with a GitHub account and gets access to the **DVD2027** organization.
- Each pair submits **one branch**, named with the **representative's ID** (Section 4). Homework is run on the cloud account of one of the partners.
- **Write both partners' names and IDs** wherever an assignment asks for a name and ID, e.g. report and output-file headers.
- Wherever an assignment derives parameters from "your ID", use the **representative's ID** — the branch name.
- **Both partners are evaluated in the lab**, and each must be able to explain the whole assignment.
- Don't pair with someone who is expected to miss a lot of time with an excuse, such as Miluim. Miluimnikim should pair with each other or submit alone — no extensions are given for ignoring this.

### 3.5 Assignment requirements

- **The README of each homework repository is the complete specification** of that assignment.
- **Interfaces are fixed.** The automatic grader depends on the exact module names, port names, file names and output formats. If yours differ, its checks fail.
- **Name and ID** must appear wherever the assignment asks for them.

### 3.6 Questions and communication

- All discussion happens on the **course Discord server**. Each homework has its own channel (e.g. `#hw1`), and there are channels like `#technical-problems`.
- Ask and answer questions in the right channel — helping your classmates is encouraged.
- **Use the course chatbot first.** It was trained on the course material and on previous questions, and it can answer most of them. **The staff answer on Discord during lab hours only.**
- For personal matters, open a **ticket** on the Discord server.
- **Personal emails or WhatsApp messages to the staff are not answered.**

---

## 4. Submitting homework with Git

Every assignment lives in its own repository under <https://github.com/DVD2027>. The `main` branch only contains a README; the assignment files are at tag **`v1.0`**. Each pair submits by pushing **one branch named exactly the representative's 9-digit ID number** — the ID the representative registered with in the course sign-up form. In the commands below, `<ID_NUMBER>` is that ID.

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

- **Branch naming — mandatory, no exceptions:** the branch name must be **exactly** the representative's ID number.
  - ✅ Correct: `206123456`
  - ❌ Incorrect: `hw1-206123456`, `israel-branch`, `ID_206123456`, `206123456-hw`, or anything else with extra text or symbols
  - Consistent branch names are what the automatic grader and the staff rely on. **A wrong branch name may cost points and, in severe cases, disqualify the homework.**
- **One branch per pair**, in every assignment repository.
- **Never push to `main`** or to any branch other than your own.
- Commit only what the assignment asks for. Do not commit workspace clutter (`xcelium.d/`, `waves.shm/`, logs) — the provided `.gitignore` files already exclude most of it.
- You can push as many times as you like. The automatic grader checks your branch **at the deadline (Sunday 23:59)**, and the in-lab evaluation is based on your branch **at the start of the evaluation lab**.
